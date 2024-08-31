# 장고를 이용한 인스타그램 클론 코딩 (CSS, DB 연결)

### *MVT 구조
모델 : DB  
템플릿 : 유저가 보는 화면  
뷰 : 모델-템플릿 중계  

### *가상환경
```
python -m venv myenv
```
```
source myenv/scripts/activate # 활성화

deactivate # 비활성화
```

### *프로젝트 실행
```
python manage.py runserver
```
http://127.0.0.1:8000  
127.0.0.1 로컬 호스트 : 자신을 의미  
8000 : 포트번호, 외부에서 들어오는 요청을 받아들임
<hr/>

# MVT 구조 이해하기

### 1. 프로젝트 생성
```
django-admin startproject kinsta
```

```
cd kinsta
```

### 2. Django가 사용할 데이터베이스 생성
```
python manage.py migrate
```

### 3. html 파일 넣는 장소
> kinsta/templates/kinsta/index.html 생성  
> kinsta/kinsta/settings.py
```
TEMPLATES = [
    'DIRS': [BASE_DIR / 'templates'], # 추가
] 
```

### 4. Django Rest Framework 설치
```
pip install djangorestframework
```

### 5. url 연결 (rest_framework 이용)
#### 템플릿-뷰 과정
1. templates에 html 생성
2. views.py에 함수 등록
3. urls.py에서 url에 따라 함수를 불러옴
> kinsta/kinsta/views.py 생성
```
from django.shortcuts import render
from rest_framework.views import APIView

class sub(APIView): # 클래스형 뷰
    def get(self, request): # get 요청이 왔을 때
        return render(request, "index.html") # html 파일 위치
```

```
# rest_framework 사용 안한 기본 형태
def index(request):
    return render(request, 'index.html')
```

urls.py -> views.py -> templates의 html 실행
> kinsta/kinsta/urls.py
```
from .views import index, membership

urlpatterns = [
    path('', sub.as_view(), name='index'), # 추가, 127.0.0.1:8000/'' 호출하면 이 함수 실행    
]
```
<details>
<summary> **5.1 앱 만들고 url 연결** </summary>

**1. 앱 생성**  
**2. templates 생성**  
**3. view 함수 등록**  
**4. 앱의 url 설정**  
**5. 프로젝트 url에서 앱 연결**  


1. 앱 생성
```
python manage.py startapp content
```

2. 앱에 대한 html 담을 폴더 생성 (kinsta/templates/kinsta 폴더가 있는 곳)
> kinsta/templates/content 생성

3. views.py에 함수 등록
> kinsta/content/views.py
```
class test(APIView):
    def get(self, request):
        return render(request, 'content/test.html') # templates/content/test.html 불러옴
```

4. urls.py에서 함수를 불러옴
> kinsta/content/urls.py 생성
```
urlpatterns = [
    path('test', test.as_view(), name='test'), # http://localhost:8000/content/test 경로로 접근
]
```

5. kinsta의 urls에서 앱의 url 연결
> kinsta/kinsta/urls.py
```
urlpatterns = [
    path('content/', include('content.urls')) # 추가
]
```
</details>

<details>
<summary>현재까지의 파일 구조</summary>

* kinsta
  * db.sqlite3  
  * manage.py  
  * kinsta/  
    * settings.py  
    * urls.py  
    * views.py
  * content/ (앱)
    * urls.py
    * views.py
  * templates/  
    * kinsta
      * main.html
    * content
      * test.html
</details>

<hr/>


<details>
<summary>메인화면 만들기</summary>

### 부트스트랩 이용
> 웹 페이지의 아이콘, 버튼같은 것들이 디자인 되어있는 패키지

https://getbootstrap.kr/docs/5.3/getting-started/introduction/#%eb%b9%a0%eb%a5%b8-%ec%8b%9c%ec%9e%91

빠른 시작 코드 붙여넣기 -> 부트스트랩 코드 사용 가능

### 구글 머티리얼 아이콘 사용
https://fonts.google.com/icons

### [css-flex 익히기](https://studiomeal.com/archives/197)

### 6. 상단 바 만들기

### 7. 하단 정렬
display: flex = 가로 방향으로 배치  
justify-content: center = 가운데 정렬  
padding-top:55px = 상단 바와의 간격

<details>
<summary>margin과 padding의 차이</summary>

* margin : 외부에서 밀어주는 느낌
* padding : 내부에서 미는 느낌
</details>


### 8. 상단 바 고정, 오른쪽 고정, 상단 바 페이지의 가로 꽉 채우기

### 9. 좌측 피드 만들기

1. 사용자 정보
2. 사진 부분
3. 아이콘
4. 댓글 부분

### 10. 우측 추천 목록 만들기
</details>
<hr/>

# 모델 구조 (DB연결, ORM)
ORM : Object Relational Mapping

객체-DB 매핑, SQL을 사용하지 않고도 DB 작업 수행 가능

### 11. 피드에 대한 필드 파악
#### 피드
|**ID**|**프로필사진**|**작성자이름**|**올린사진**|**글내용**|댓글|좋아요수|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|o|o|o|o|o|-> 참조|-> 참조|

#### 댓글
|**피드_ID**|**댓글작성자**|**댓글내용**|
|:-:|:-:|:-:|
|o|o|o|

#### 좋아요
|**피드_ID**|**좋아요한사람**|**좋아요여부**|
|:-:|:-:|:-:|
|o|o|o|

(좋아요여부 : 취소 기능을 위함)

### 12. 모델 작성
> kinsta/content/models.py
```
class Feed(models.Model):
    profile_image = models.TextField() # 프로필사진
    user_id = models.TextField() # 작성자이름
    image = models.TextField() # 올린사진
    content = models.TextField() # 글내용
    like_count = models.IntegerField() # 좋아요수
```

### 13. migration 작업
모델 변경사항 기록
```
python manage.py makemigrations
```
DB에 적용
```
python manage.py migrate
```

[SQLite Tool](https://sqlitestudio.pl/)

### 14. view에서 DB 불러오고 html(템플릿)에 전달(렌더링)
> kinsta/content/views.py
```
from .models import Feed

class main(APIView):
    def get(self, request):
        feed_list = Feed.objects.all().order_by('-id') # select * from content_feed
        
        # 사전 형식으로 전달 { key(템플릿으로 전달할 이름) : value }
        return render(request, 'kinsta/main.html', context=dict(feeds=feed_list))
```
### 15. main.html에 템플릿 언어 사용해서 피드 데이터 DB에서 가져오기
```
{% for feed in feeds%}            
    <div>{{feed.content}}</div>
    ...
{% endfor%}
```

<hr>

### 16. 게시물 추가 기능 (모달 창 만들기, JS, JQuery)
* 모달 창 띄우기 html
```
<!-- 게시물 추가 기능 -->
<div class="modal_overlay">
    <div class="modal_overlay_top">
        <button id="modal_X" class="modal_close">
            <span class="material-symbols-outlined">close</span>
        </button>
    </div>
    <div style="display: flex; align-items: center; justify-content: center;">
        <div class="modal_window">
            <div class="modal_window_top">
                새 게시물 만들기
            </div>
            <hr>
            <div class="modal_window_bottom">
                <div class="image_upload_section">
                    사진을 여기에 끌어다 놓으세요
                </div>
            </div>
        </div>
    </div>
</div>
```

* JQuery 사용하기 위해 추가
```
<!-- JQuery -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
```

* 게시글 추가 아이콘에 클릭 기능 추가 (id를 넣어줌)
> kinsta/templates/kinsta/main.html
```
<span id="nav_add_box" class="material-symbols-outlined" style="font-size:30px; cursor:pointer;"> add_box </span>
```

* 아이콘 클릭 시 작동 (이벤트 추가)
``` 
<script>
    $('#nav_add_box').click(function () {
        // modal 띄우기
        $('.modal_overlay').css({
            display: 'block',
            top: window.pageYOffset + 'px',
        })
    })
</script>
```

JQuery 형식
```
$(selector).on(event, function)
$('#id').이벤트
$('.class').이벤트
$(선택자).동작함수()
```

* 드래그 앤 드롭으로 이미지 업로드
```
$('.image_upload_section')
    .on("dragover", dragOver)
    .on("dragleave", dragOver)
    .on("drop", uploadFiles);

function dragOver(e) {
    e.stopPropagation();
    e.preventDefault();

    if (e.type == "dragover") {
        $(e.target).css({
            "background-color": "skyblue",
        });
    } else {
        $(e.target).css({
            "background-color": "white",
        });
    }
}

// 사진을 업로드 했을 때
function uploadFiles(e) {
    e.stopPropagation(); // 부모들에게 영향 주는 것을 막음
    e.preventDefault(); // 이벤트에 대한 기본 동작을 막음

    e.dataTransfer = e.originalEvent.dataTransfer;
    var files = e.dataTransfer.files;

    if (files.length > 1) {
        alert('하나만 올려라.');
        return;
    }

    // 사진이 정상적으로 올라가면
    if (files[0].type.match(/image.*/)) {
        $(e.target).css({
        "background-image": "url(" + window.URL.createObjectURL(files[0]) + ")",
        "outline": "none",
        "background-size": "100% 100%"
    });
    } else {
        alert('이미지가 아닙니다.');
        return;
    }
}
```

* 글 작성으로 넘어가기
이미지가 업로드 됐을 때 modal_window_bottom의 html 수정
```
$('.modal_window_bottom').html(`
    <div class='image_upload_section'></div>
    <div class='글 작성 부분'><div>
`)
```

### 17. 서버로 파일 업로드하는 API 만들기 (AJAX), view로 넘기기
* 업로드 사진의 변수 files를 전역 변수로 선언해서 함수에서도 쓸 수 있도록 함

> 공유하기 버튼 클릭 -> ajax로 데이터 보내기
```
// 글 작성 후 공유하기 버튼 클릭했을 때
// 동적으로 생성된 버튼이기 때문에 $(document).on으로 접근
$(document).on('click', '#feed_create_button', function () {
    let file = files[0];
    let img_name = file.name;
    let content = $('#feed_input_content').val();
    let user_id = $('#my_id').text();
    let profile_img = "";

    alert('내용: ' + content + '\n아이디: ' + user_id);
    
    let fd = new FormData();            

    fd.append('file', file);
    fd.append('img_name', img_name)
    fd.append('content', content);
    fd.append('user_id', user_id);
    fd.append('profile_img', profile_img);

    // /content/upload url로 접속하면 실행되는 함수로로 넘기기(view)
    $.ajax({
        url: "/content/upload",
        data: fd,
        method: "POST",

        processData: false,  // 데이터를 쿼리스트링으로 변환하지 않음
        contentType: false,

        success: function(data){
            console.log("성공");
        },
        error: function(request, status, error){
            console.log("에러");
        },
        complete: function(){
            console.log("ajax 완료");
        }
    }
    )
    console.log("클릭--");
});
```

### 18. 장고 서버에 이미지 올리기
일반적으로 이미지 DB를 따로 두어 그곳에 이미지를 저장하고 그 주소를 불러오지만<br>
이 프로젝트에서는 장고 프로젝트(media 폴더)에 이미지를 저장한다.

> kinsta/media 폴더 생성<br>
> kinsta/kinsta/settings.py
```
import os

# 각 media 파일에 대한 URL Prefix
MEDIA_URL = '/media/' # 항상 / 로 끝나도록 설정
# MEDIA_URL = 'http://static.myservice.com/media/' 다른 서버로 media 파일 복사시

# 업로드된 파일을 저장할 디렉토리 경로
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

> kinsta/kinsta/urls.py
```
# media 폴더에 접근하기 위한 코드
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

> kinsta/content/views.py
```
from rest_framework.response import Response
from uuid import uuid4
import os
from kinsta.settings import MEDIA_ROOT, MEDIA_URL

class upload_feed(APIView):
    def post(self, request):
        # file = request.data.get('file')
        
        file = request.FILES['file']
        
        # 파일 이름을 고유한 id로 변경 (프로그램에서 다루기 쉽게)
        uuid_name = uuid4().hex
        # /MEDIA_ROOT/uuid_name로 이미지 저장
        save_path = os.path.join(MEDIA_ROOT, uuid_name)
        
        # 파일 읽기
        with open(save_path, 'wb+') as destination:
            for chunk  in file.chunks():
                destination.write(chunk)
        
        content = request.data.get('content')
        user_id = request.data.get('user_id')
        profile_img = request.data.get('profile_img')
        
        img_location = uuid_name
        
        print(file)
        print(uuid_name)
        print(content)
        print(user_id)
        print(profile_img)
        
        # 컬럼명 = 데이터
        Feed.objects.create(profile_image = profile_img, user_id = user_id,
                            image = img_location,content = content,
                            like_count = 0)

        return Response(status=200)
```

이대로 하면 DB에 저장된 파일이름만으로는 파일 위치를 알 수 없음
> main.html 수정 (MEDIA URL을 앞에 붙인 주소로 접근)
```
{% load static %}
<!-- 사진 올리는 부분 -->
<div class="photo_section">
    <img src="{% get_media_prefix %}{{feed.image}}">
</div>
```
<details>
<summary>서버에 이미지 올리기 과정 정리</summary>

1. 브라우저(main.html)의 피드 정보를 ajax로 view에 정보 전송 (url: "/content/upload")
2. kinsta/content/urls.py 에서 url 연결: path('upload', upload_feed.as_view())
3. settings.py 에서 MEDIA URL 설정
4. kinsta/kinsta/urls.py 에서 media 폴더 접근 연결
5. views.py 에서 이미지 처리(class upload_feed), DB에 objects.create
6. html에서 피드에 올라올 사진 media/image_name 으로 접근할 수 있도록 함
</details>

### 19. 피드 올리기 완료하면 메인 다시 불러오기
```
location.replace('/main')
```
<hr>

### 20. 사용자(user) 모델 만들기
사용자 모델은 장고에 디폴트로 존재 (auth_user)<br>
이것을 상속 받아 커스텀 유저 모델 만들기<br>
유저 앱 만들기 (settings.py 에 앱 등록하는 것 잊지 말기)
```
python manage.py startapp user
```
#### 유저 모델
|**이메일**|**비밀번호**|**이름**|**닉네임**|**프로필사진**|
|:-:|:-:|:-:|:-:|:-:|
|o|o|o|o|o|


> kinsta/user/models.py
```
from django.contrib.auth.models import AbstractBaseUser

class user(AbstractBaseUser):    
    user_email = models.EmailField(unique=True) # 유저 이메일 주소 (회원가입할때 사용하는 아이디)
    # user_password  # 유저 비밀번호
    user_name = models.CharField(max_length=20) # 유저 실제이름
    user_nickname = models.CharField(max_length=20, unique=True) # 유저 닉네임
    profile_img = models.TextField() # 유저 프로필 사진
    
    USERNAME_FIELD = 'user_nickname'
    
    class Meta: # 테이블 이름
        db_table = "user"
```

> settings.py
```
# Apply custom user model
AUTH_USER_MODEL = 'user.user'
```

### 21. 회원가입 html 작성

### 22. 로그인 html 작성

views.py 작성, urls 연결

### 23. ajax를 통해 회원가입 정보 보내기, 받기
> signup.html
```
<script>
    $('.btn-primary').click(function () {

        let email_address = $('#input_email_address').val();
        let name = $('#input_name').val();
        let nickname = $('#input_nickname').val();
        let password = $('#input_password').val();

        // 피드에서는 사진을 올리기 위해 FormData를 사용했지만
        // 회원가입 정보는 String만 있기 때문에 JSON 사용
        $.ajax({
            url: "/user/signup",
            data: {
                user_email: email_address,
                user_name: name,
                user_nickname: nickname,
                user_password: password
            },
            method: "POST",

            success: function (response) {
                if (response.success) {
                    alert("회원가입 성공");
                    console.log("--회원가입--성공");
                    location.replace('/user/login')
                } else {
                    alert("회원가입 실패\n" + response.error);
                    console.log("--회원가입--실패");
                }
            },
            error: function (request, status, error) {
                alert("회원가입 에러");
                console.log("----에러");
            },
            complete: function () {
                console.log("----ajax 완료");
            }
        }
        )
    })
</script>
```

패스워드는 장고에서 제공하는 암호화(make_password) 사용
> kinsta/user/views.py
```
from .models import user
from django.contrib.auth.hashers import make_password
from rest_framework.response import Response
from django.http import JsonResponse


# Create your views here.
class signup(APIView):
    def post(self, request):
        profile_img = "media/default_profile.jpg"
        email = request.data.get("user_email")
        user_name = request.data.get("user_name")
        user_nickname = request.data.get("user_nickname")
        user_password = request.data.get("user_password")

        if user.objects.filter(user_email=email).exists():
            return JsonResponse({"success": False, "error": "이메일이 중복되었습니다."})
        
        if user.objects.filter(user_nickname = user_nickname).exists():
            return JsonResponse({"success": False, "error": "닉네임이 중복되었습니다."})

        user.objects.create(
            profile_img=profile_img,
            user_email=email,
            user_name=user_name,
            user_nickname=user_nickname,
            password=make_password(user_password),
        )
        return JsonResponse({'success' : True})
```

### 24. ajax를 통해 로그인 정보 보내기, 받기
> kinsta/user/views.py
```
class login(APIView):
    def post(self, request):
        user_email = request.data.get("user_email")
        user_password = request.data.get("user_password")
        
        login_user = user.objects.filter(user_email = user_email).first()
        
        if login_user == None:
            print("존재하지 않는 사용자 접근")
            # return JsonResponse({"success" : False, "error" : "회원정보가 잘못되었습니다."})
            return JsonResponse({"success" : False, "error" : "존재하지 않는 회원입니다."})
        
        if login_user.check_password(user_password):
            return JsonResponse({"success" : True})
        else:
            # return JsonResponse({"success" : False, "error" : "회원정보가 잘못되었습니다."})
            return JsonResponse({"success" : False, "error" : "비밀번호가 잘못되었습니다."})

```
<hr/>
https://youtu.be/M8UPyeF5DfM
