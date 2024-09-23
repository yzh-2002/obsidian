## 从0到1

### 基本概念
python的web框架：
1. Flask：轻量级
2. Django：重量级，继承了Web开发所需的基本所有依赖
	1. 安装之后会提供`django-admin`，类似于前端脚手架，帮忙创建项目文件结构
	2. `django-admin startproject xxx`创建项目

项目默认文件说明：
1. `manage.py`：
2. xxxx
	1. `__init__.py`
	2. <font color="#ff0000">`settings.py`：项目配置</font>
		1. 数据库链接
		2. ...
	3. `urls.py`：
	4. `asgi.py`
	5. `wsgi.py`：Web Server Gateway Interface，即Python服务器网关接口

**project 和 apps的区别：**
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

从上可知，Django采用MVT（model-view-template）架构
1. model：负责与数据库交互，定义数据结构
2. view：负责处理业务逻辑和相应用户请求
3. template：负责将数据渲染为用户可视界面

较之传统的MVC架构来说，MVT中的View相当于MVC中的Controller，Template则相当于MVC中的View。

### 数据库配置

`python manage.py migrate`：该命令会查看`INSTALLED_APPS`配置，然后根据`settings.py`中的数据库配置和项目各个应用下的数据库迁移文件（也即`migrations`文件夹下的内容）创建任何必要的数据库表。

上述一般是本地部署别人已开发的项目时，一键创建适配该项目的数据库。自己从0开始开发项目时，需要自己在`app/models.py`中定义数据结构，然后执行命令`python manage.py makemigrations app`

该命令会检测你对`modes.py`的修改，并且把修改的部分存储为一次迁移（一种记录，不会改变项目依赖数据库内容），也即`migrate`，之后我们可以执行上面提到的`python manage.py migrate`命令（自动执行数据库迁移并同步管理数据库结构）
