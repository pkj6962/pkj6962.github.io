****


## 템플릿 언어 

템플릿 언어는 HTML 작업을 더 수월하게 해주는 장고 프레임워크 상의 언어이다. 


템플릿은 (지금까지 배운 내용 하에서) ```{% %}```로 감싸진 형식 안에 ```태그``` 와 ```변수```로 이루어진다. 
 

 #### 변수 
 변수는 ```'{{ }}'``` 안에 작성된다.

 #### 태그 
 
 템플릿 태그를 작성함으로써 HTML 내부에서 if 문, for문과 같은 flow control 이 수행되게 할 수 있다. 

- 단독으로 사용 가능한 태그와, ```opening tag```와 ```closing tag```의 쌍으로 이루어진 태그가 있다. 
-  단독: ```{% extends sth %}```
- 개폐: ```{% block sth %}``` and ```{% endblock %}```
 

- 내장 태그 목록(공식 문서)
  https://django-doc-test-kor.readthedocs.io/en/old_master/ref/templates/builtins.html#ref-templates-builtins-tags



### url 

html 내부에서 ```href```와 같이 url을 입력해야 하는 경우 템플릿 언어로써 urls.py 안의 url들을 참조할 수 있다.  

```
<a class="nav-link active" aria-current="page" href="{% url 'home' %}">Home</a>

```

```url``` 태그가 urls.py 에서 ```'home'```을 ```name```으로 가지는 path를 찾아준다.

urls.py의 urlpatterns 리스트에는 ```home```을 ```name```으로 가지는 경로가 있어야 한다. 

```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.home, name='home'), 
    path('about/', views.about, name='about')
]
```

### 상속 

중복되는 html 내용을 반복해서 작성해야 하는 번거로움을 줄여주는 기능이다.

카카오톡 클론코딩을 했을 때의 기억을 되살려 보면, 하단의 ```네비게이션 바```나 최상단의 ```상태바``` 등은 모든 파일에서 중복되었었고, 새로운 파일을 작성할 때마다 이전 파일의 내용을 복붙한 뒤 새로운 부분을 작성해주어야 했다. 장고는 이러한 중복성과 비효율성을 막아주는 기능을 제공한다. 


모든 파일에서 공통적인 "골격" 템플릿과 "블록"을 정의해야 한다. 


### 골격 문서 
골격 문서(```base.html```으로 생성)에는 기본 템플릿 코드를 작성한 뒤, 파일마다 달라지는 부분에는 아래와 같이 블럭을 추가해준다.(```block```태그가 활용된다) 

```
{% block <블럭이름A> %}

// 서로 다른 html에서 서로 다른 코드가 들어가는 부분 

{% endblock %} 
```

이 블럭 위 아래로는 템플릿 코드(공통의 코드)인 것이다. 


### 자식문서

위 템플릿 코드(중복된 코드)를 활용할 자식문서의 html 맨위에는 템플릿 언어로 작성된 다음의 명령문을 작성해준다. ```extends```태그가 활용된다. 

```
{% extends 'base.html' %}
```

그리고 템플릿 코드의 __블럭__ 에 들어가는 블럭을 정의해준다. 


```
{% block content %}


<div class="alert alert-success" role="alert">
    <h4 class="alert-heading">This is Our Homepage</h4>
    <p>Aww yeah, you successfully read this important alert message. This example text is going to run a bit longer
        so that you can see how spacing within an alert works with this kind of content.</p>
    <hr>
    <p class="mb-0">Whenever you need to, be sure to use margin utilities to keep things nice and tidy.</p>
</div>

{% endblock %}
```

템플릿코드 파일에 작성한 블럭의 이름과 이를 활용하는 파일에서의 블럭의 이름이 (당연히) 일치해야 한다. 

#
#
#

#### 부록 

참고로 block 태그를 열고 엔딩 태그로 닫아두지 않으면 요청한 웹페이지는 열리지 않고 아래와 같은 에러메시지가 출력된다. 

```
Unclosed tag on line 5: 'block'. Looking for one of: endblock.
```







