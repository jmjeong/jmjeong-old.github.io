---
layout: post
title: Django-appsales
description: 
category: 
tags: python
---
{% include JB/setup %}

django-appsales는 [iTunesConnects](http://itunesconnect.apple.com)의 application sales report를 다운로드 받아
DB에 넣어서 여러 view로 sales report를 보기 위한 프로그램입니다. 프로그램의
최신 소스는 [Github](https://github.com/jmjeong/django-appsales)에 있습니다.

## Installation

### 사전 필요 사항

이 프로그램을 구동하기 위해서는 다음과 같은 프로그램이 필요합니다.

- django 1.2.3 above
- django-extensions
- django-pagination
- pyofc2 (for chart)
- south (for schema migration)

Python package가 인스톨되어 있으면 
`easy_install` 명령을 이용하여 시스템에 필요 package를 인스톨할 수 있습니다.

	sudo easy_install django django-extensions django-pagination pyofc2 south

App sales data를 자동으로 받아오기 위해서 java 모듈이 install되어 있어야 합니다. 
Ubuntu 상에서는 다음과 같이 인스톨할 수 있습니다. 

	sudo apt-get install openjdk-6-jre-headless        

### Downloads

	git clone git://github.com/jmjeong/django-appsales.git appsales

## 설치

### 설정

`settings.py`에서 필요한 설정을 조정합니다. 

자동화를 위해서 `APPSTORE_ID`와 `APPSTORE_PW` 항목을 설정합니다.

{% highlight python %}
 ACCOUNT_INFO = [
     {                                   # account 1
         'VENDOR_ID'   :   ,             # Vendor ID(8x...) the entity which you want to download the report
         'APPSTORE_ID' : '',             # iTunes Store AppStore ID
         'APPSTORE_PW' : '',             # iTunes Store AppStore PW
       
         # Directory where sales data is stored
         'DATA_DIR' : os.path.join(os.path.dirname(__file__), 'app1-sales-rawdata')
         },
     # {                                 # account 2
     #     'VENDOR_ID'   :   ,           # Vendor ID(8x...) 
     #     'APPSTORE_ID' : '',           # iTunes Store AppStore ID
     #     'APPSTORE_PW' : '',           # iTunes Store AppStore PW
       
     #     # Directory where sales data is stored
     #     'DATA_DIR' : os.path.join(os.path.dirname(__file__), 'app2-sales-rawdata')
     #     },
     ]
 
 # for admob integration
 #
 ADMOB_INFO = {
     'client_key' : '',                  # API key
     'email' : '',                       # id
     'passwd' : '',                      # passwd
     }
{% endhighlight %}

### 기본설치 

다운로드 받은 appsales directory에서 `./manage.py syncdb`를 입력합니다.
프로그램 수행 중간에 admin 계정 설정을 묻는 항목이 나옵니다. 

{% highlight console %}
 jmjeong-ui-MacBook-Pro:appsales jmjeong$ ./manage.py syncdb
 Creating table auth_permission
 Creating table auth_group_permissions
 Creating table auth_group
 Creating table auth_user_user_permissions
 Creating table auth_user_groups
 Creating table auth_user
 Creating table auth_message
 Creating table django_content_type
 Creating table django_session
 Creating table django_site
 Creating table sales_app
 Creating table sales_date
 Creating table sales_country
 Creating table sales_sales
 Creating table django_admin_log
 
 You just installed Django's auth system, 
           which means you don't have any superusers defined.
 Would you like to create one now? (yes/no): yes
 Username (Leave blank to use 'jmjeong'): 
 E-mail address: jmjeong@gmail.com
 Password: 
 Password (again): 
 Superuser created successfully.
 Installing index for auth.Permission model
 Installing index for auth.Group_permissions model
 Installing index for auth.User_user_permissions model
 Installing index for auth.User_groups model
 Installing index for auth.Message model
 Installing index for sales.Sales model
 Installing index for admin.LogEntry model
 Installing json fixture 'initial_data' from absolute path.
 Installed 245 object(s) from 1 fixture(s)
 jmjeong-ui-MacBook-Pro:appsales jmjeong$ 

 jmjeong-ui-MacBook-Pro:appsales jmjeong$ ./manage.py migrate 
{% endhighlight %}

### 서버구동

`django` 기본 서버를 이용하여 web server를 구동할 수 있습니다. 

{% highlight console %}
 jmjeong-ui-MacBook-Pro:appsales jmjeong$ ./manage.py runserver
 Validating models...
 0 errors found
 
 Django version 1.2.3, using settings 'appsales.settings'
 Development server is running at http://127.0.0.1:8000/
 Quit the server with CONTROL-C.
{% endhighlight %}

시스템 서버 구동은 [Django deployment](http://docs.djangoproject.com/en/dev/howto/deployment/)를 참고하십시오.

### 웹브라우저에서 접속

`http://localhost:8000`으로 접속하여, 앞서 만든 계정으로 login하면 아래와 같은 화면을 볼 수
있습니다.

![](http://farm4.staticflickr.com/3694/13171583753_4af9d62755_o.png)

### 데이타 입력

`./manage.py runjob populate`를 이용해서 데이타를 입력합니다.
DB에 import되는 data는 [iTunesConnect](http://itunesconnect.apple.com)에서 다운로드 받은 Sales data입니다. 

![](http://farm8.staticflickr.com/7301/13171599493_8956f31c8c_o.png)

`settings.py`에 정의된 `DATA_DIR`에 위치한 파일 중에서 정의된 Prefix를 가진 파일을 읽어 들입니다.

	jmjeong-ui-MacBook-Pro:appsales jmjeong$ ./manage.py runjob populate
	Populate data files in [/Users/jmjeong/django/appsales/sales-rawdata]
	[2010/11/08] is now processing...

`DATA_DIR`에 있는 파일 중에서 `S_D_mmddyyyy.txt`이나 `S_D_mm-dd-yyyy.txt`와 같은 형태의 파일만 읽어서
처리합니다. 이미 처리된 파일은 중복처리가 되지 않기 때문에 이 명령은 여러번 사용해도 괜찮습니다.

## 자동화

### Daily Report 받기

[AppDailySales](http://appdailysales.googlecode.com/)는 iTunes Connect web site로부터 daily sales data를 자동으로 다운로드 받는
python script입니다. `./utils/appdailysales.py`는 [AppDailySales](http://appdailysales.googlecode.com/)에 아래와 같은 수정을 하였습니다.

- 이미 다운로드 받은 report는 받지 않기
- Option과는 상관없이 현재 iTunes Connect에서 모든 daily sales report를 받기

`./manage.py runjob download`를 하면 `settings.py`에 설정된 Id, Pw를 이용하여 sales data를 받아서
=DATA_DIR=에 저장을 합니다.

	jmjeong-ui-MacBook-Pro:appsales jmjeong$ ./manage.py runjob download
	Report file downloaded: 
	['/Users/jmjeong/django/appsales/sales-rawdata/S_D_11-09-2010.txt'] 

### 각 나라별 Review 받기

`./manage.py runjob download-review` 명령을 수행하면 US, KR, HK, JP, AU, DE, GB 나라의 Review를
받아서 DB에 등록합니다.

### Crontab에 등록

`cron.sh` script는 iTunesConnect site로부터 sales data를 download하여 DB에 저장하는 script입니다.
`crontab -e`를 이용하여 system의 crontab에 등록합니다.

	@daily /path/to/cron.sh
	@daily /path/to/manage.py runjob download-review

### Screenshots

- 메인 페이지

![](http://farm3.staticflickr.com/2743/13171821624_1aef04587a_o.png)

- 항목별로 Sort

![](http://farm4.staticflickr.com/3741/13171666213_f4f2c2f0d8_o.png)

- Application별 통계

![](http://farm3.staticflickr.com/2652/13171567595_09a5f72a0b_o.png)

## Credits

프로그램에서 사용하는 나라별 Icon Set은 [Country Flag Icon Set](http://www.gosquared.com/liquidicity/archives/1493)에서 가져왔습니다.
