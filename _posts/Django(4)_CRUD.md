Database
=========

- 클래스로 테이블을 정의
  - 각 속성에 필드 타입을 정의해주어야 한다. 
  - 


- migration
  - 데이터베이스에 변경사항을 반영하는 것 
  - ```python manage.py migrate```
    - 초기화
    - 변경사항 반영
  - ``` python manage.py makemigrations ```
    - 변경사항을 담은 ```migration``` 파일을 만듬
    - 이후에 ```migrate``` 명령어를 통해 이를 데이터베이스에 반영

#
프로젝트를 만들고 최초로  ```runserver```를 하면 다음과 같은 경고 메시지를 출력한다. 이는 초기값을 migrate하지 않아서 출력되는 메시지이다. 
```
You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```


#
#
### 객체 선언
models.py 에 다음과 같이 테이블 클래스를 생성한 뒤 migrate 하면 클래스에 대응되는 테이블이 DB에 생성된다. 

```
class Blog(models.Model): # Model을 상속받음 --> 테이블을만들것이라는 것을 선언
    title = models.CharField(max_length=200)  # 제목.  
    body = models.TextField() # 본문 
    tag = models.CharField(max_length=50)
    date =  models.DateTimeField(auto_now_add=True)  # 작성날짜 # 자동으로 지금 시간을 추가하겠다. 
    


    def __str__(self):
        return self.title # 블로그 

```

> 궁금한 점: 테이블 구조를 바꿀 때는 makemigrations 명령이 선행되어야 했지만, 인자값(eg. auto_now_add = True)를 바꾸는 것은 수정만으로도 변경사항이 적용되었다. 그 이뉴는 무엇인가? 

> 테이블에 속성을 추가할 때는 ```makemigrations```을 해야하지만 속성을 삭제할 때는 이 작업이 필요 없다. (내가 해봄)

#
#
```makemigrations```을 하면 다음과 같이 json형식의 파일이 생성된다. 

```
class Migration(migrations.Migration):

    initial = True

    dependencies = [
    ]

    operations = [
        migrations.CreateModel(
            name='Blog',
            fields=[
                ('id', models.BigAutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('title', models.CharField(max_length=200)),
                ('body', models.TextField()),
                ('date', models.DateTimeField(auto_now_add=True)),
            ],
        ),
    ]

```
> - Json형태로 저장됨. 
> - ID 속성이  primary key로서 자동으로 만들어짐.
> - admin 사이트에서 우리가 만든 테이블 객체를 확인할 수 있음. 

#

### *객체 변경하기: 새로운 속성을 추가하는 경우 

기존에 있던 테이블(클래스)에 새로운 속성을 추가하고 makemigrations을 하면 다음과 같은 경고?에러? 메시지가 발생한다. 

```
It is impossible to add a non-nullable field 'tag' to blog without specifying a default. This is because the database needs something to populate existing rows.
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit and manually define a default value in models.py.
Select an option: 
```
  기존의 테이블에 새로운 속성을 추가하면 기존의 튜플들은 그 속성값을 부여받지 못했던 상태이다. 따라서 "기존의 튜플에 입력되는 기본값을 입력하라"는 메세지이다. 
(populate: 덧붙이다, one-off: 단한번의)


#
makemigrations을 하면 다음과 같은 메시지가 프롬프트에 출력된다. 내가 어떤 작업을 수행했는지에 대해 꽤나 사용자친화적으로 출력해줌을 확인할 수 있다. 

```
Migrations for 'blogapp':
  blogapp/migrations/0002_blog_tag.py
    - Add field tag to blog

  blogapp/migrations/0003_remove_blog_tag.py
    - Remove field tag from blog

```



#
#
#

## 새 글 입력하기 

### html form 이용하여 새 글 입력하기  

- ```'새 글 생성'``` a 태그를 타면 ```new.html```로 넘어가서, 
```input``` 태그나 ```textarea``` 태그에 정의된 폼에 제목과 본문을 작성할 수 있다. 

```html
<form action="{% url 'create' %}" method="POST">
    {% csrf_token %}  <!-- 보안문제 해결 기능 토큰-->
    <div>
        <label for="title">제목</label><br/>
        <input type="text" name="title" id="title">
    </div>
    <div>
        <label for="body">본문</label><br/>
        <textarea name="body" id="body" cols="30" rows="10"></textarea> 
    </div>

    <input type="submit" value="글 생성하기">

</form>
```

#### form tag 
> - 메소드
>>  form data가 서버로 제출될 때 사용되는 HTTP 메소드를 명시한다.
>  - GET: 폼 데이터를 URL에 추가하여 전달
>>   - 브라우저에 캐시되어 저장 
>>    - 보안상 취약점 존재 
> - POST: 폼 데이터를 별도로 첨부하여 전달 
>>     GET 방식보다 보안성이 좋음 
>
>- 액션 
>>  폼 데이터를 서버로 보낼 때 해당 데이터가 도착할 url

#
#

## 입력한 글 DB에 저장하기 
-submit 타입의 input 태그를 입력하면  ```/create``` 로 이동하고, 여기서는 ```create.html```을 render하는 것이 아니라 새 글을 생성해주는 함수가 실행된다. 


```
from django.shortcuts import render, redirect
from .models import Blog
from django.utils import timezone

def create(request): # 블로그 글을 저장해주는 함수 --> 앞선 함수와 성격이 다르다 (무언가를 보여줄_render_필요가 없음)
    if(request.method=='POST'):
        post = Blog()                               # Blog 객체 생성 
        post.title = request.POST['title']          
        post.body = request.POST['body']
        post.tag = 0
        post.date = timezone.now()
        post.save()                                 # 데이터베이스에 저장하는 함수(```insertSQL```을 수행)

    return redirect('home') # 인자에 해당하는 URL로 HttpResponseRedirect 리턴 
```
#
> ### Redirect와 Render의 비교 
> render
> 요청한 페이지를 템플릿과 context를 결합하여 리턴 
> HttpResponse 객체를 리턴
> # 
> redirect 
> 특정한 Url로 요청을 보냄 


### django form을 이용하여 새 글 작성하기 

```py
# forms.py에 정의
from django import forms

class Blogform(forms.Form):
    # 내가 입력받고자 하는 값들 
    title = forms.CharField()
    body = forms.CharField(widget=forms.Textarea)

```


``` py
def formcreate(request):
    if request.method == 'POST':
        # 입력 내용을 DB에 저장
        form = Blogform(request.POST)
        if form.is_valid(): # form이 유효한지 체크 
            post = Blog() # 튜플 생성
            post.title = form.cleaned_data['title']
            post.body = form.cleaned_data['body']
            post.save() 
            return redirect('home')
    else: # GET request
        form = Blogform()
    return render(request, 'form_create.html', {'form': form}) 
```
세번째 인자: form을 'form'이라는 이름으로 form_create.html에 보냄. html에서는 '{{ form }}' 으로써 사용 (as below)

```html
<h1>django form을 이용한 새 글 작성 페이지</h1>
<form action="" method="POST">
    {% csrf_token %}
    <table>
        {{ form.as_table}}
    </table>
    <input type="submit" value="새 글 생성하기">
</form>
```

```{{form.as_table}}```은 넘겨받은 form(tuple)을 table 형식으로 보여준다. 



### Modelform 을 이용하여 새 글 작성하기 

```Modelform```은 ```Form```에 ```Model```을 연결시킨 것이다.

```forms.py```에 다음과 같은 모델폼을 form의 Modelform을 상속받아 정의한다. 

```py
class BlogModelForm(form.Modelform):
  class Meta:
    model = Blog      # models.py로부터 Blog객체를 import했기 때문에 가능 
    fields = ['title', 'body']    # 모든 ATTRBT을 포함하고 싶으면 '__all__'이라고 적음 
```

```views.py```에서는 사용자가 ```request```를 보냈을 때 처리할 함수를 정의한다. 
```py
def modelFormCreate(request):
  if request.METHOD = 'POST':
    form = BlogModelForm(request.POST)
    if form.is_valid():
      form.save()
      return redirect('home')
  else:
    form = BlogModelForm()
  return render(request, 'form_create.html', {'form':form})
```







```

1. 장고의 form으로부터 Modelform을 불러온다. 



#
#
#

1. db로부터 블로그 객체를 가져오는 코드를 views.py로 가져와야 함.
   1. rende
2. render의 세번째 인자로 index.html로 해당 객체를 보낼  수 있음

posts = Blog.objects.all() // 블로그 객체들이 모조리 가져와짐. 

posts = Blog.objects.filter().order_by('-date')

return render(request , 'index.html', post)

<table>
{{ post }}
</table>

- 쿼리셋
  - views.py가 db로부터 전달받은 객체들의 목록 
  - for문으로써 템플릿언어로써 객체 하나하나를 찍어줄것 
```
{% for post in posts %}

  <h3>제목: {{ post.title }}</h3>
  <h4>작성날짜: {{ post.date }}</h4>
  {{ post.body }}

{% endfor %}

---


### 디테일 페이지 렌더링하기 

```detail/1```, ```detail/2```,....하나하나 html파일 만들어서 렌더링할꺼야? 

- db에서 자동으로 생성된 primary key인 id로써 디테일 페이지 하나하나를 렌더딩해주는 프로그램을 짤 수 있다. 

```index.html```에는 다음과 같이 경로 지정

  ```
  {% for post in posts %}
  
  <a href="{% url 'detail' post.id %}"> {{post.id}
  } {{ post.title}} </a>

  {% endfor %}
  
  ```

  'detail'외에 post.id 값이 추가적으로 필요

#

```urls.py```에는 다음과 같이 경로 지정 

  ```
  path('detail/<int:blog_id>', view.detail, name='detail'), 
  ```

  정수형 변수 blog_id(임의로 지정 가능)가 detail함수에 넘겨지게 됨 



```views.py```에는 다음과 같이 ```detail```함수 선언

  ```py

  from django.shortcuts import get_object_or_404
  def detail(request, blog_id): # blog_id라는 추가적인 인자를 받음
    # blog_id번째 블로그 글을 db로부터 가져와서 
    
    blog_detail = get_object_or_404(Blog, pk=blog_id) # 특정 개체 하나를 가져오는 메소드 

    
    # detail.html로 띄워주는 코드 
    return render(request, 'detail.html', {'blog_detail': blog_detail})
  ```

  ```get_obeject_or_404```함수는 blog_id를 pk로 가지는 Blog객체 '하나'를 반환하거나 없으면 404에러를 반환하는 함수 



- 파일 업로드 
- 미디어
  - 사용자에 의한 데이터
  

- settings.py 맨 아래에 설정 필요
```
  MEDIA_ROOT = os.path.join(BASE_DIR, 'media') 
  MEDIA_URL ='/media/'
```  
- root: 미디어파일이 저장되는 경로
- url: 사용자가 올린 미디어(파일)에 접근할 수 있는 url경로


- urls.py에 경로 추가 
  - urlpatterns에 더하라
    - +static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

  or 

    - urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
  
  +)

  fron django.conf import settings
  from django.conf.urls.static import static

  --> 이해x 암기o 


```models.py```안에 블로그 클래스 안에 사진을 추가학 ㅗㅅ피음
  - photo = models.ImageField(blank=True, null=True, upload_to='blog_photo')
    - 미디어가 추가될 때마다 미디어 안의 blog_photo 디렉토리 안에 자동으로 저장됨.
    - 클래스가 변경됐으니 manage.py로 makemigration 및 migrate 이 필요 

- 객체가 바꼈으니 form도 바껴야함. 


