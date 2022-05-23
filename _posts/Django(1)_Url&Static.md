# Hello Django (2)
#####
서버의 토대를 이루는 Static Files들을 관리하고 불러오는 법
------

스태틱 파일은 서버를 배포하기 전에 미리 만들어 놓은 Css, Java Script, Image 파일 등을 통틀어 이르는 말이다. 

이렇게 미리 만들어 놓은 파일들은 이곳 저곳 산개해서 저장하기 보다 일정한 곳에 체계적으로 관리해야 장고가 파일을 효율적으로 탐색할 수 있다. 

```settings.py```에서 ```static file```과 관련한 설정을 할 수 있다. 



### 1. STATIC_URL 
   ```static_url```객체에는 static 파일을 제공하는 url을 저장한다. 이로써 브라우저단에서, 즉 외부에서 스태틱 파일에 접근할 수 있다.
```
STATIC_URL = '/static/' # 스태틱 파일을 제공하는 url 
```
# 
# 
### 2. Staticfiles_dir
   
   ```staticfiles_dirs```객체(두 단어 끝에 모두 ```s```가 붙음에 유의)에는 스태틱 파일이 저장된 곳을 리스트 형태로 저장한다. 
# 

스태틱 파일을 앱 별로 저장할 필요가 있을 때에는 이 객체에 앱별 저장소(디렉토리)를 저장하면 되는 것이다. 

장고는 개발자가 스태틱파일 디렉토리를 명시했을 때 이 리스트 상 원소를 하나씩 탐색하면서 그 디렉토리에 원하는 스태틱 파일이 있는지를 확인할 것이다.



```
   STATICFILES_DIRS = [
    BASE_DIR / 'static',
    os.path.join(BASE_DIR, 'staticapp', 'static'), 
] # 개발할 때 스태틱 파일들이 위치한 경로 
```

첫번째 인자는 ```'(최상위폴더)/static'``` 를,
두번째 인자는 ```'(최상위폴더)/staticapp/static'```을 의미한다. 이 디렉토리들에 스태틱 파일이 저장되어 있다는 뜻이다.
#
> __BASE_DIR__
> ```BASE_DIR```는 프로젝트의 최상위 디렉토리를 말한다.(```startproject``` 명령을 실행한 워킹디렉토리) ```settings.py``` 에 
> _BASE_DIR = Path(__file__).resolve().parent.parent_ 로 선언되어 있다. _(```settings.py```의 부모 폴더의 부모 폴더)_

# 
### 3. Static_Root
   
```static_root```는 작성한 모든 스태틱 파일을 저장해놓는 디렉토리이다. 
```python manage.py collectstatic``` 명령어를 입력하면 지금까지 작성한 모든 스태틱 파일이 ```settings.py```에 선언한 디렉토리에 모두 저장된다.

#
# 

### 스태틱 파일을 불러 오는 법

이렇게 작성하고 저장한 스태틱 파일이 다른 파일에서 불려오질 때(가령 _html_ 파일에서 _css_ 파일을 _import_ 하는 등)는 장고의  __템플릿 언어__ 의 문법을 따라야 한다. 


```html
{% load static %}
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" typ="test/css" href="{% static 'css/style.css' %}">
    <link rel="stylesheet" typ="test/css" href="{% static 'style2.css' %}">
    <title>static Practice</title>
</head>

<body>
    <div>hello static! </div>
    <img src="{% static 'img/likelion.png' %}" />
</body>

</html>
```

가장 먼저 static 파일을 로드하는 파일의 첫줄에는 ```{% load static %}``` 이라는 문장을 작성해주어야 한다. 

이때 중괄호```{}```와 ```%```로 감싼 양식의 언어를 __템플릿 언어__ 라고 한다.


그 다음 헤더에서 ```style.css```를 _import_ 해주고 있다. 이 파일의 경로가 (최상위 폴더 다음으로) ```static/css/style.css```라는 의미를 담당하고 있다. 
이때 ```static```과 그다음 경로가 공백으로 구분되는 문법에 유의하자.(_static_ 이 _reserved word_ 역할을 하는 것은 아니고, 최초에 디렉토리명을 그렇게 선언한 것이다.) 


