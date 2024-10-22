## git flow
> 一种`git`开发规范，同时也是一款工具（原理很简单，封装了一些git命令）、

项目一般有两条长期分支：`master`和`develop`，前者用于存储对外发布的版本，后者用于日常开发
三条短期分支：
1. `feature`：从`develop`分出来，用于功能开发，完成之后合入`develop`
2. `release/test`：较少遇到，用于正式发版之前测试验证（其实可以直接在develop上进行）
3. `bugfix`：从`master`分出来，用于紧急修复bug，修改完成之后再合进`master`和`develop`

```shell
git checkout -b feature/xxx develop
git checkout develop
git merge --no-ff feature/xxx 
```

`--no-ff`的作用可见下图：
1. 在合并分支上创建一个新的提交，同时保持分支独立性
2. 不添加，默认的`merge`策略会导致合并分支上有开发分支上的所有提交记录，导致合并分支的提交记录比较混乱
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202410132107702.png)

---

## git conflict

1. 不同分支`merge`：功能开发分支`feature/xxx`对文件a修改后想`merge`到`develop`分支，但先前有一个同事也对a进行修改并提前合并到`develop`，此时merge就会产生冲突
	1. 养成习惯：修改代码之前先`git pull`，同步完之后再修改代码
2. `pull or push`：
	1. `merge`之前，尝试`git pull`同步代码时便发现已经冲突
	2. `push`冲突一般很少遇见，因为每次开发都会单独拉一个分支，该分支只有自己负责

如何解决？
1. `git pull`冲突，则丢弃本地冲突的文件，采用远程文件覆盖本地文件，然后再重新修改
2. `git stash`

## ssh
> 常用于安全连接远程主机

原理：
1. 远程主机收到用户的登录请求，将公钥发给用户
2. 用户使用该公钥加密登录密码并发送给远程主机
3. 远程主机用私钥解密登录密码，正确便同意用户登录

风险：中间人攻击
用户向远程主机申请登录时被人拦截，其冒充远程主机把伪造的公钥发送给用户，用户无法确认该公钥是否是远程主机签发的（不像`https`协议存在证书中心`CA`）

1. 密码登录
	1. 第一次申请登录远程主机，会出现下面提示`The authenticity of host 'host (xx.xx....)' can't be established.RSA key fingerprint is xx:xx.....Are you sure you want to continue connecting (yes/no)?`
	2. 让用户自己确认该公钥是否属于远程主机（还是中间人），如何确认呢？一般远程主机会在自己网站贴出公钥指纹
	3. 用户确认之后会将该公钥添加到`.ssh/known_hosts`内，表明该公钥值得信赖，后续便不再询问
	4. 之后用户输入登录密码即可
2. 免密登录（公钥登录）
	1. 用户将自己主机上的公钥存储在远程主机`.ssh/authorized_keys`上即可
	2. 登录时，远程主机向用户发送一段随机字符串，用户用私钥加密后发给远程主机
	3. 远程主机使用实现存储的公钥解密，如果解密后字符串一致，则该用户可信

个人主机可以使用`ssh-keygen`生成一对公钥`id_rsa.pub`和密钥`id_rsa`

`github`上配置`ssh`，就是免密登录的一种应用。