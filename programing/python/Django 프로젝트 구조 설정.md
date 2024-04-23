### **기본적인 장고 프로젝트 구조**

------

우선 startproject와 startapp을 실행하면 생성되는 기본 구조를 살펴보겠습니다.

```python
$ mkdir all_project
$ cd all_project
$ django-admin startproject my_project
$ cd my_project
$ python manage.py startapp myapp
📦 all_project			# (1) repository_root
└─ my_project			# (2) project_root
   ├─ my_project		# (3) configuration_root
   │  ├─ __init__.py
   │  ├─ asgi.py
   │  ├─ settings.py
   │  ├─ urls.py
   │  └─ wsgi.py
   ├─ myapp/
   │  ├─ __init__.py
   │  ├─ admin.py
   │  ├─ apps.py
   │  ├─ models.py
   │  ├─ tests.py
   │  └─ views.py
   └─ manage.py
```

**(1) repository_root** : **프로젝트의 최상위 루트.** `.gitignore`, `requirements.txt` 등등의 배포에 필요한 파일 들이 위치합니다.
**(2) project_root** : **프로젝트명으로 생성된 디렉터리.** 장고 프로젝트 소스들이 위치합니다. 단순 디렉터리 역할만 하므로 이름을 변경하여도 괜찮습니다.
**(3) configuration_root** : **프로젝트명으로 생성된 디렉터리.** `settings 모듈`과 `URLConf`가 자리잡습니다. 이 곳의 이름으로 참조하고 있는 코드가 존재하므로 이름을 함부로 변경하시면 안됩니다.
만약 이 곳의 이름을 수정하고 싶다면 의존되는 몇몇 파이썬 파일들의 코드를 변경해주어야 합니다. (dependency)

##### 🚀 [(Django) Writing your first Django app, part 1](https://docs.djangoproject.com/en/3.2/intro/tutorial01/)

### **참고 할만한 장고 프로젝트 구조**

------

> **계층 체계가 복잡해지면 디버그, 수정, 확장 등이 매우 어려워진다. 수평 구조가 중첩된 구조보다 좋다.**

공부할 때는 기본 구조만으로도 충분하지만, 실제 프로젝트를 하다보면 기본 구조는 유지보수가 힘듭니다. 다른 구조로 프로젝트를 만들어 봅시다.

```null
$ mkdir all_project
$ cd all_project
$ django-admin startproject my_project .
$ cd my_project
$ django-admin startapp myapp
```

my_proejct 다음에 **점 기호(.)**가 있음에 주의합시다. 점 기호는 현재 디렉터리를 의미합니다. 위 명령은 현재 디렉터리인 all_project(루트 폴더) 를 기준으로 프로젝트를 생성하겠다는 의미입니다.

대부분의 듀토리얼 프로젝트들이 `$ django-admin startproject my_project` 명령어를 사용하지만, 이럴 경우 my_project 디렉터리 밑에 똑같은 이름의 my_project 앱 디렉터리가 생성되어 불필요하게 depth가 깊어진 'all_project/my_porject/my_project' 와 같은 구조가 되어 버립니다.

우리의 최종 목표는 아래와 같은 프로젝트 구조입니다.

```null
📦 all_project/		# (1) repositroy_root	
├─ .gitignore
├─ README.md
├─ requirements.txt
├─ myvenv/
└─ my_project/		# (2) project_root		
   ├─ apps/
   │  └─ myapp/
   │     ├─ models/
   │     │  └─ __init__.py
   │     │  └─ profile.py
   │     │  └─ instagram.py
   │     ├─ admin.py
   │     ├─ models.py
   │     ├─ tests.py
   │     ├─ views/
   │     └─ forms/
   ├─ config/		# (3) configuration_root
   │  ├─ settings/
   │  │  └─ settings.py
   │  ├─ __init__.py
   │  ├─ asgi.py
   │  ├─ urls.py
   │  └─ wsgi.py
   ├─ static/
   │  └─ myapp/
   ├─ media/
   │  └─ myapp/
   ├─ templates/	
   │  └─ myapp/
   └─ manage.py
```

진행해야하는 작업들은 다음과 같습니다.

**1. ( configuration_root 이름 변경 )** `settings` 파일들이 위치한 `my_project` 디렉터리의 이름을 가독성을 위해 `config` 로 변경.

**2.** `config` 디렉터리 밑에 `settings` 디렉터리 생성 후 `settings.py` 파일을 이곳으로 이동.

**3.** `app`, `views.py`, `forms.py` 등등은 프로젝트의 규모에 따라 조절해나가면됩니다. 여기선 파일들이 많아질 것을 대비하여 `apps/` `views/` `forms/` 디렉터리를 생성하여 관리하기로 했습니다.

여기선 확장 가능성을 염두에 두어 `apps/` 폴더를 만들었지만, 앱은 확신이 없다면 웬만하면 확장하지 말고 단일 앱으로 시작하도록 합시다. 종속성이 물리는걸 해결하기 위한 설계는 어렵기 때문입니다.

**4.** `apps/myapp` 디렉터리를 생성한 후 다음 명령어를 실행하여 앱을 생성합니다.

```null
$ python manage.py startapp myapp ./apps/myapp
```

**5.** `static/` `templates/` 등등의 프론트앤드 관련 입니다.
**파일들을 앱 단위가 아니라 프로젝트 단위로 관리합니다.** `static/` `templates/` 디렉터리 아래에 각 앱별로 서브디렉터리를 둡시다. 계층 체계가 복잡해질 수록 HTML을 수정하고 확장하기 어려워지기 때문입니다.

### **참고 할만한 장고 프로젝트 구조 - PATH설정**

------

**1. configuration_root 경로**

구조와 이름을 변경해 주었다고 끝이 아닙니다. 앞에서도 언급했지만 **configuration_root** 의 이름을 변경하면, 의존되는 코드들을 변경해주어야 한다고 하였습니다.



➤ `manage.py` 와 `asgi.py`, `wsgi.py` 에서 `DJANGO_SETTINGS_MODULE`의 환경변수를 변경해줍니다.

```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.settings')
```

➤ settings.py의 위치도 settings 디렉터리 하위로 옮겨졌으니(1depth 만큼 깊어졌다.), `settings.py` 의 `BASE_DIR` 변수도 바꿔줍니다.

```python
# 1) 일반적인 경로 작성
BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
#  2) pathlib를 이용하여 경로 작성
from os.path import abspath, dirname

BASE_DIR = dirname(dirname(dirname(abspath(__file__))))
```

➤ `settings.py` 의 `ROOT_URLCONF`, `WSGI_APPLICATION` 변수도 수정해 줍니다.

```python
ROOT_URLCONF = 'config.urls'
WSGI_APPLICATION = 'config.wsgi.application'
```

**2. app 경로**

기본적인 장고 프로젝트 구조에서 보았듯이, 앱을 생성하면 **project_root** 하위에 앱 폴더가 생성됩니다. 하지만 여러 앱들을 `project_root/apps` 폴더 내부에 두고 관리하고자 하였으므로,
➤ `settings.py` 의 AppConfig도 그에 알맞은 경로로 적어줍니다.

```python
INSTALLED_APPS = [
    ...
    ...
    'apps.myapp', 
    ]
```

**3. templates 경로**

templates은 일반적으로 장고 프로젝트의 루트에서 앱과 같은 레벨에 존재합니다. 상당히 일반적인 관례이며 직관적이며 쉽게 쫓아갈 수 있는 패턴입니다.

그러나 우리는 templates은 앱 디렉터리 밖에 두어 프로젝트 단위로 관리하기로 했습니다. 따라서 Django가 해당 경로에서 파일들을 찾을 수 있도록 경로를 수정해줘야 합니다.
➤ `settings.py` 의 TEMPLATES의 DIRS에 경로를 추가시킵니다.

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
        	# File System Template Loader
        	os.path.join(BASE_DIR, 'templates')
        ],
        'APP_DIRS': True,
         ...
    },
]
```

##### 🚀 [(Django) how to overriding-templates](https://docs.djangoproject.com/ko/3.2/howto/overriding-templates/)

**4. static 경로**

`static` 도 마찬가지로 프로젝트 단위로 관리하기로 했습니다.
앱 단위로 관리할 때의 불편함 때문입니다.

```null
// 앱 단위로 관리시

my_project/
├─ app_name/
│  └─ static/
│     └─ app_name/
│        └─ main.css
└─ app_name2/
   └─ static/
      └─ app_name2/
         └─ main.css
```

앱 단위로 관리하게되면 위와 같은 디렉터리 구조를 사용하게 됩니다.

`collectsatic` 명령어가 수행될 때 여러 디렉터리로 나눠진 static 파일들은 `STATIC_ROOT` 에 지정된 곳으로 파일들이 복사 되는데, 만약 같은 이름의 파일명이 있다면 서로 덮어쓰게 되어 곤란해집니다.

따라서 각 앱 디렉터리의 static 디렉터리 안에서도 해당 앱의 이름을 가진 디렉터리를 또 만들어 충돌을 피하는 법을 쓰지만, 여러 번 디렉터리를 만드는 것은 계층구조도 이쁘지 않고 귀찮습니다.

```null
// 프로젝트 단위로 관리시

my_project/
├─ static/
├─ app_name1/
│  └─ main.css
└─ app_name2/
   └─ main.css
```

하지만 프로젝트 단위로 `static` 을 관리하면 여러 경로를 거칠 필요가 없어 편의성이 좋습니다.

그러면 마찬가지로 Django가 프로젝트 단위의 경로에서 파일들을 찾을 수 있도록 수정해주도록 합시다.
➤ `settings.py` 의 `STATICFILES_DIRS`에 경로를 추가시킵니다.

```python
# 프로젝트 전반적으로 사용되는 static 파일의 경로.
# File System Loader에 의해 참조되는 설정.

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]
```