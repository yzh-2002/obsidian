1. 项目配置文件：`config`
2. 用户管理：`user`
3. 静态资源管理：`static`
6. 任务管理：`task`
7. ---------------------- 上面师兄负责，下面我负责
8. 文件管理：`file`
9. 拓扑展示：`graph`
10. 统计信息：`stat`

## 全局配置

1. 日志：
2. 全局异常处理：[参考链接](https://juejin.cn/post/7011901157429739551)

## user模块
> 用户管理模块

models中定义了User的结构，继承自Django内置的AbstractUser，除此之外添加了token字段。
views中定义了register，login，logout三个函数
### Authentication
> 自定义装饰器实现

view_func：装饰器装饰的视图函数
设置了装饰器，会在执行视图函数之前先执行TODO内的校验逻辑。

```python
def token_required():
    def decorator(view_func):
        def _wrapped_view(request, *args, **kwargs):
            # TODO 处理认证逻辑的代码
        return _wrapped_view
    return decorator
```
## static模块
> TODO：该模块函数定义不符合django的static设计

无models
views定义：
1. DetectServer（探测服务器）
	1. Add / Update / Delete
	2. Connection（ssh连接） / Deploy（安装scamper等软件）
	3. Display
2. StaticFiles
	1. Upload / Delete
	2. Display

## task模块


## file模块
views：
1. 获取scamper文件
	1. scamper：一个网络测量工具
2. 获取kapar文件
	1. [kapar](https://www.caida.org/catalog/software/kapar/)，类似于scamper
3. 获取clickhouse文件
	1. clickhouse：开源的列式数据库管理系统

## graph模块

## stat模块

