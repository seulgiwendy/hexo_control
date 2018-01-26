---
title: Django로 삽질한 이야기 (01) - 사이트가 없어졌다! 
date: 2018-01-24 22:12:48
tags:
    - python
    - django
---

### Django로 삽질한 이야기 (01) - 사이트가 없어졌다! 
*글쓴이 주: Java-Spring이 백엔드 개발의 전부인 줄 알았던 봄이네집이 Django 기반 웹서비스 개발 학습을 시작한다.*

*Django의 늪에서 삽질하는 얘기를 앞으로도 종종 블로그에 담을 생각이다.*

---
#### 삽질의 배경

[Hoyoung Jung님의 Django로 웹서비스 만들기 강의](https://www.inflearn.com/course/django-%ED%8C%8C%EC%9D%B4%EC%8D%AC-%EC%9E%A5%EA%B3%A0-%EA%B0%95%EC%A2%8C/) 를 열심히 따라하다보면 Django 기반 웹 프로젝트에 Social Login을 붙일 수 있게 된다. 

[Django-Allauth](https://github.com/pennersr/django-allauth)를 사용하는 것인데 이 패키지는 사용법이 매우 쉽고, 설정이 간결하며 무엇보다도 시중의 많은 소셜 로그인 공급자들의 설정에 맞게 다양한 구현체가 존재한다는 점이 굉장한 이점이다. 

**그러나 이제 이 패키지는 쓸 수 없다.** Django가 2.0으로 판올림되면서 [get_val_from_obj() 메소드가 삭제](https://code.djangoproject.com/ticket/24716)되었는데 아직 Django-allauth 패키지는 이에 대응하지 못하고 있기 때문이다. 삭제된 이 메소드를 여전히 사용하고 있는 Django-allauth는 admin page에서 유저 정보에 접근시 엄청난 에러를 뿜어낸다. 정확히 말하면 User 객체가 갖고 있는 정보를 JSON으로 serialize해주지 못하는 상황에 직면하게 된다. 

그래서 대안으로 소셜 로그인 패키지의 교체를 결정하고, 삽질이 시작되는데..... 

---
#### 삽질의 시작

소셜 로그인 패키지를 [Python-social-auth](https://github.com/python-social-auth/social-app-django)로 교체하였다. 

교체한 후 애플리케이션의 모델 중 기존 allauth가 생성한 user_id를 Foreign key로 갖고 있는 부분을 다 수정하였다. 

그러나 기존 DB 스키마가 갖고 있는 column constraints에 자꾸 걸려서 migrate 명령이 **실행되지 않았고** 결국 아래와 같은 무식한 방법으로 그냥 DB를 **통째로 날려버렸다.**

```Shell
rm db.sqlite3
```

**진짜 단순무식한 방법이지만 효과는 굉장했다!** 이제 migrate 명령이 잘 작동한다. *당연하지. DB를 통째로 날렸는데....*

그리고 admin page가 접속되지 않기 시작하는데......

---
#### 삽질의 원인

admin 페이지는 아래와 같은 메시지를 뿜어내며 접속되지 않고 있었다. 

```
DoesNotExist at /admin/ Site matching query does not exist.
```

문제가 발생한 원인은 간단한 것으로 Django가 DB에 갖고 있던 **사이트 정보** 가 사라진 것이 원인이다. 원래 Django는 DB에 Site라는 객체를 매핑한 후 사이트 정보를 담아둔다. 이 매핑 작업은 django-admin startproject 시에 수행되지만, 나처럼 project 개발 도중에 DB를 날려버리는 **화끈한 개발자**를 위한 대비는 해 두지 않았으므로 나같은 사람들은 간단한 shell 작업을 통해 Site 객체를 다시 DB에 입력시켜줘야 할 것이다. 

---
#### 삽질의 끝

```Shell
(from your project root directory)

python manage.py shell

>>>from django.contrib.sites.models import Site
>>>Site.objects.create(name='your_site_name', domain='your_site_domain')
```

작업을 수행해준 후 project root directory의 settings.py에 방문한다. 아래와 같은 라인이 존재하는지 체크하고 없으면 작성해준다. 

```Python
SITE_ID = 1
#많은 경우에 SITE_ID는 1번이다. 그러나 여러분의 상황에 따라 다를 수도 있다. 이런 설정은 알아서 잘 하자. 
```

이제 admin 페이지는 잘 보일 것이다. 

---
#### 참고 문헌 

[Stack Overflow-Getting Site Matching Query Does Not Exist Error after creating django admin](https://stackoverflow.com/questions/11476210/getting-site-matching-query-does-not-exist-error-after-creating-django-admin)

---

*과연 삽질이 이것뿐이었을까요? Django로 삽질한 이야기의 글감은 넘쳐나고 있습니다.*

*앞으로도 계속 연재될 Django 삽질기, 많이 기대해주세요! 고맙습니다.*



