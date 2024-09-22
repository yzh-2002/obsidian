## 从0到1

python的web框架：
1. Flask
2. Django
	1. 安装之后会提供`django-admin`，类似于前端脚手架，帮忙创建项目文件结构
	2. `django-admin startproject xxx`创建项目

项目默认文件说明：
1. `manage.py`
2. xxxx
	1. `__init__.py`
	2. `settings.py`：项目配置
		1. 数据库链接
		2. ...
	3. `urls.py`：
	4. `asgi.py`
	5. `wsgi.py`

project 和 apps的区别：
1. 通常一个project包含多个apps
2. app可以视作一个大型项目的不同功能拆分
3. `python manage.py startapp xxx`

应用默认文件说明：
1. xxxx
	1. `admin.py`
	2. `apps.py`
	3. `models.py`：
	4. `views.py`：
	5. `tests.py`
	6. `migrations`

快速上手：
1. 在项目中注册应用
	1. 项目目录下`settings`的`INSTALLED_APPS`添加，应用的名称在`apps.py`的config类中配置
2. 编写URL和视图函数的对应关系
	1. 项目目录下`urls.py`的`urlpatterns`中配置
3. 启动django项目
	1. `python manage.py runserver`

模板（templates）：
也即视图函数想要返回一个html，可以在templates文件夹（一般放到应用文件夹下）下定义模板html文件，视图函数通过`return render(request,"模板html名称")`即可