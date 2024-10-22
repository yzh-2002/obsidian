> `tensorflow`的版本与`CUDA`版本严格匹配，所以需要安装多版本的`CUDA`，具体对应关系参考：官方测试构建配置[链接](https://tensorflow.google.cn/install/source_windows?hl=zh-cn#tested_build_configurations)
## 环境配置
> 本机NVIDIA GPU驱动版本：`560.94`

卸载原先安装的`cuda`组件：>程序和功能
保留以下四个，其余均可卸载：
1. `NVIDIA PhysX 系统软件`
2. `NVIDIA GeForce Experience`
3. `NVIDIA 图形驱动程序`
4. `NVIDIA Control Panel`

cuda组件主要涉及两部分：
1. `cuda toolkit`：有最低显卡驱动要求，通常把电脑显卡驱动更到最新即可
	1. 安装过程有UI，点一点即可
	2. `cupti`一般跟`cuda toolkit`一起安装
2. `cudnn`：与`cuda`版本一一对应，[参考链接](https://developer.nvidia.com/rdp/cudnn-archive)
	1. 下载安装包，解压之后将对应`bin,include,lib`合并到对应cuda文件夹即可
3. 配置环境变量
	1. `cuda`安装之后会自动添加到环境变量，但`cudnn`和`cupti`还未添加
	2. `%CUDA_PATH_Vxx%\lib\x64`
	3. `%CUDA_PATH_Vxx%\include`
	4. `%CUDA_PATH_Vxx%\extras\CUPTI\lib64`
	5. `%CUDA_PATH_Vxx%\bin`（这个一般随着`cuda toolkit`安装已经被配置过）

**多版本的`cuda`切换时，只需要移动环境变量配置表中的位置即可**

踩坑：
1. `_Internal: Blas GEMM launch failed_`
	1. `tf-1.x`版本不支持`RTX 30系`显卡进行GPU训练，可以使用NVIDIA维护的`nvidia-tf1.15`，但仅支持Linux
	2. 切到`tf-2.1`以上版本便可以解决问题
2. `tf-2`需要兼容旧版本代码时可执行`tf.compat.v1.disable_eager_execution()`，实测很有用