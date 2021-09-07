---
title: "[프로젝트] 고기각 (Cloning '정육각')"
date: 2021-07-18T14:34:24+09:00
authors: ["Jungminsayho"]
# tags: [""] 

---
## 프로젝트 개요
**프로젝트 소개**: Django를 이용한 커머스 사이트 '정육각' 클론 프로젝트 <br/>
**개발기간**: 2021.07.05 - 2021.07.16 (총 12일)<br>
**참여인원**: 프론트엔드 3명, 백엔드 3명<br>
**github 주소**: https://github.com/jungminsayho/gogigak

<br>

## 백엔드 구현 기능
(내가 작업한 내용은 **👀 표시** 하였다.)
- bcrypt와 jwt를 이용한 회원가입 / 로그인 기능
- 마이페이지 구현
- 주소 별 신선배송 가능여부 반환 기능 
- **제품 상세페이지 구현 👀**
- **제품 리스트 필터링 기능 (카테고리 별, 판매순/리뷰순/가격순) 👀**
- **제품에 대한 리뷰 작성/읽기/삭제 기능 👀**
- 장바구니에 상품 추가/읽기/삭제/수량변경 기능
- 구매 기능

<br>

------

## 기능 구현에 대한 고민
(내가 작업한 기능 중, 고민이 많이 들어갔던 기능에 한하여 작성하였다.)<br><br>
### 1. 제품 리스트 필터링 기능

```python
# products/views.py

sort          = request.GET.get('sort', '')
category_name = request.GET.get('category', None)
category      = None
```

<br>

먼저, query parameter로 받을 `sort`와 `category`의 value를 모두 받아둔다.<br>
`category = None`으로 미리 변수설정을 해둔 이유는 아래에 나오지만, 카테고리 별로 카테고리 이미지도 반환해줘야 하는데,
제품리스트의 전체를 반환하는 경우(카테고리 필터 없이)의 이미지는 프론트에서 하드코딩으로 넣는 방법을 사용했기 때문에, 이 경우 백엔드에서는 카테고리 이미지를 빈 값으로 반환하기 위함이다.

<br>

```python
# products/views.py

q = Q()

if Category.objects.filter(name=category_name).exists():
    category = Category.objects.get(name=category_name)
    q.add(Q(category=category), q.AND)
```

<br>

카테고리 필터링을 위해 **`Q 객체`**를 사용했고, **`Q 객체`**에 아무 필터조건도 저장되지 않은 경우에는 전체 product를 반환한다.

<br>

#### 🧹 Q를 향한 삽질

처음에 Q에 대해 알아봤을 때는, `조건A or 조건B`와 같은 여러 조건의 합집합을 사용하기 위한 방법이라고만 생각했고, 그래서 내가 구현해야 할 필터링에는 필요하지 않다고 생각했다.<br>
그래서 길고 긴 `if 조건문`들과 `에러 처리`로 줄줄이 내려적었고, 결국 원하는대로 구현은 됐었다.

하지만, Q에 대해 다시 힌트를 얻고 코드를 수정한 결과, 코드가 정말 많이 줄었고 불필요한 에러처리도 줄일 수 있었다.<br> 그리고 지금보다 훨씬 더 복잡한 필터링 로직을 구현해야 할 때 정말 유용한 방법이라는 것을 알았다.

(2주라는 기간 안에서) 이걸 너무 늦게 알았지만, 그래도 비효율적인 방법으로 삽질한 경험을 얻었고 그를 통해 Q의 소중함을 알게 되었다.

<br>

```python
# products/views.py

results = {
    'category_image': category.image if category else None,
    'items': [
        {
            'id'       : product.id,
            'name'     : product.name, 
            'price'    : int(product.price),
            'grams'    : int(product.grams),
            'thumbnail': product.thumbnail,
            'isOrganic': product.is_organic,
            'sales'    : product.sales,
            'reviews'  : product.reviews,
            'options'  : [{'id': option.id, 'name': option.name} for option in product.options.all()],
            'stock'    : product.stock
        } for product in Product.objects.filter(q).order_by(sort_dict.get(sort, 'id'))]
    }
```

<br>

category 필터를 저장한 q로 product를 필터링한 후, `sort`라는 key로 받은 value를 통해 판매순/리뷰순/가격순 필터링도 적용한 결과물을 반환한다.

<br><br>

### 2. 리뷰 읽기/삭제 기능 구현

리뷰 읽기를 구현하려는데, 리뷰를 읽어올 때 본인이 작성한 리뷰에는 `삭제`버튼이 달리도록 구현하고 싶었다.<br>
다른 로그인이 필요한 기능에 모두 달았던 `login_decorator`를 달자니, 비로그인 상태에서는 아예 리뷰를 읽을 수도 없게 되니, 어떻게 해야할까 고민이 많았다.<br><br>

그래서 팀원들과 함께 내놓은 해결책은, 기존의 `login_decorator`를 사용하지 않고, `login_decorator`의 내용을 `ReviewView`의 `get`함수 내에 삽입하되, 토큰을 받아올 때와 아닐 때의 경우를 나누어 로직을 만드는 것이었다.<br>
이 방법으로도 내가 원하는 기능은 구현이 됐지만, 멘토님이 여러가지를 고려하여 2가지의 대안을 제시해 주셨다.<br><br>

**1. 데코레이터 나누기** <br>
(비로그인/로그인 상태에서 모두 사용할 수 있는 데코레이터와 로그인이 필요한 기능에만 사용하는 데코레이터)<br>
**2. 엔드포인트 나누기** <br>
(비로그인 상태와 로그인 상태로 접근하는 엔드포인트를 달리 하기)<br><br>

프로젝트 막바지여서 시간이 부족했기 때문에, **1번**을 선택하여 데코레이터를 하나 더 작성했다.<br>
토큰이 없을 때는 `request.user`를 빈값으로 반환하고, 토큰이 있을 때는 `request.user`에 로그인한 유저를 담아 반환한다.<br><br>


```python
# utils.py

def public_login_required(func):
    def wrapper(self, request, *args, **kwargs):
        try:
            request.user = ''
            token        = request.headers.get("Authorization", None)
            
            if token:
                payload      = jwt.decode(token, SECRET_KEY, algorithms="HS256")
                request.user = User.objects.get(id = payload.get('user_id', None))

            return func(self, request, *args, **kwargs)
        
        except jwt.exceptions.DecodeError:     
            return JsonResponse({'message' : 'INVALID_TOKEN'}, status = 400)
        
        except jwt.ExpiredSignatureError:
            return JsonResponse({"message": "EXPIRED_TOKEN"}, status = 400)

        except User.DoesNotExist:
            return JsonResponse({'message' : 'INVALID_USER'}, status = 401)

    return wrapper
```


<br> `ReviewView`의 `get` 함수에 위에서 작성한 데코레이터를 달아주고,
`results`에 `myReview`라는 key의 value로, 로그인한 유저와 리뷰를 작성한 유저가 일치할 때 `True`를 반환해준다.<br><br>

```python
# products/views.py

@public_login_required
def get(self, request, product_id):
    signed_user = request.user

    results = [{
        'id'            : review.id,
        'user'          : review.user.id,
        'purchaseCount' : Order.objects.filter(user=review.user).count(),
        'title'         : review.title,
        'content'       : review.content,
        'image'         : review.image_url,
        'createdAt'     : review.created_at,
        'myReview'      : True if review.user == signed_user else False
    } for review in Review.objects.select_related('user').filter(product_id=product_id).order_by('-id')]

    return JsonResponse({'results': results}, status=200)
```

<br><br>

### 3. 최적화 과정

<img src="https://images.velog.io/images/jungminnn/post/90de4d7f-7b8b-4c0f-91e5-4e807056e503/image.png" width="100%" height="100%"/>

제품리스트를 읽어올 때, 코드가 미흡한 탓에 호출되는 쿼리가 너무 많았다. (38개,,)
계획한 기능 구현이 모두 끝나고 난 후, 쿼리 최적화를 진행해보고자 `select_related`와 `prefetch_related`에 대해 간단히 알아보았다.
`product`와 `option`은 ManyToMany 관계이기 때문에 `prefetch_related`를 사용했다. <br><br>



```python
# products/views.py

products = Product.objects.filter(q)

results = {
    'category_image': category.image if category else None,
    'items': [
        {
            'id'       : product.id,
            'name'     : product.name, 
            'price'    : int(product.price),
            'grams'    : int(product.grams),
            'thumbnail': product.thumbnail,
            'isOrganic': product.is_organic,
            'sales'    : product.sales,
            'reviews'  : product.reviews,
            'options'  : [{'id': option.id, 'name': option.name} for option in product.options.all()],
            'stock'    : product.stock
        } for product in products.prefetch_related('options').order_by(sort_dict.get(sort, 'id'))]
    }

```


<img src="https://images.velog.io/images/jungminnn/post/c51ce79d-c65e-4a8a-b34b-41855825a184/image.png" width="100%" height="100%"/>

쿼리가 38개에서 5개로 줄었다,, 
너무 신기하고 재밌었다. 추후에 다른 기능들에도 모두 시도해 볼 예정이다!

<br>

---


### 🥩어디선가 누군가에 무슨 일이 생기면 틀림없이 나타난다 고기각 만세🥩
내가 과연 잘할 수 있을까, 시작하기 전엔 걱정이 많았다.
하지만 운이 좋게도 너무 좋은 팀원들을 만났고, 덕분에 정말 즐거운 시간을 보냈다.
1주차도 즐겁게 했고, 1주차 회고 때에도 지금처럼 남은 기간도 즐겁게 하자고 말했고,
프로젝트 마무리 회고 때에는 **정말 즐거운 시간이었다**고 말했다. 진심이었다!

처음이라 모두 어려웠을텐데, 모두의 배려와 노력으로 끝까지 잘 올 수 있었다.
너무 고맙고, 즐거웠다!

<br><br><br><br>

