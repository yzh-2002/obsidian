## 从0到1

### 基本概念
python的web框架：
1. Flask：轻量级
2. Django：重量级，继承了Web开发所需的基本所有依赖
	1. 安装之后会提供`django-admin`，类似于前端脚手架，帮忙创建项目文件结构
	2. `django-admin startproject xxx`创建项目

项目默认文件说明：
1. `manage.py`：Django提供的命令行工具，常用于启动开发服务器 and 创建app
2. project文件夹
	1. `__init__.py`
	2. `settings.py`：全局配置
		1. 数据库配置
		2. 静态文件配置
		3. 应用配置
		4. 中间件配置
		5. ...
	3. `urls.py`：
	4. `asgi.py`
	5. `wsgi.py`：Web Server Gateway Interface，即Python服务器网关接口

**project 和 apps的区别：**
3. 通常一个project包含多个apps
4. app可以视作一个大型项目的不同功能拆分
5. `python manage.py startapp xxx`

应用默认文件说明：
1. app文件夹
	1. `admin.py`：注册app中的models到Django admin页面，<font color="#ff0000">个人开发不用管</font>
		1. Django admin页面是自带的可视化数据及接口管理界面，类似于Java的Swagger？
	2. `apps.py`：基本只用于定义app名称
	3. `models.py`：定义app中的数据库模型（也即表结构）
	4. `views.py`：定义app中视图函数
		1. 视图处理来自用户的请求并返回响应
		2. Django采用MVT架构，其中View类似于MVC架构中的Controller
	5. `tests.py`：单元测试，<font color="#ff0000">个人开发不用管</font>
	6. `migrations`

快速上手：
2. 在项目中注册应用
	1. 项目目录下`settings`的`INSTALLED_APPS`添加，应用的名称在`apps.py`的config类中配置
3. 编写URL和视图函数的对应关系
	1. 项目目录下`urls.py`的`urlpatterns`中配置
4. 启动django项目
	1. `python manage.py runserver`

模板（templates）：
也即视图函数想要返回一个html，可以在templates文件夹（一般放到应用文件夹下）下定义模板html文件，视图函数通过`return render(request,"模板html名称")`即可

从上可知，Django采用MVT（model-view-template）架构
5. model：负责与数据库交互，定义数据结构
6. view：负责处理业务逻辑和相应用户请求
7. template：负责将数据渲染为用户可视界面

较之传统的MVC架构来说，MVT中的View相当于MVC中的Controller，Template则相当于MVC中的View。

### 数据库配置

`python manage.py migrate`：该命令会查看`INSTALLED_APPS`配置，然后根据`settings.py`中的数据库配置和项目各个应用下的数据库迁移文件（也即`migrations`文件夹下的内容）创建任何必要的数据库表。

上述一般是本地部署别人已开发的项目时，一键创建适配该项目的数据库。自己从0开始开发项目时，需要自己在`app/models.py`中定义数据结构，然后执行命令`python manage.py makemigrations app`

该命令会检测你对`modes.py`的修改，并且把修改的部分存储为一次迁移（一种记录，不会改变项目依赖数据库内容），也即`migrate`，之后我们可以执行上面提到的`python manage.py migrate`命令（自动执行数据库迁移并同步管理数据库结构）

**Django自带的admin管理界面，本地在DEBUG为False时，无法加载静态资源。**
暂时没找到原因
