> 详细操作可见：https://zelikk.blogspot.com/2022/01/cloudcone-vps-v2ray-1g-2t-20g-512mb-999.html
   TODO：将上述链接自己整理成文档

1. 购买一个海外ip的vps
	1. ip选择？美国？香港？
	2. vps选择？
		1. vps：Virtual Private Server 一种将一台物理服务器通过虚拟化技术分割成多个相互独立的虚拟服务器的服务
2. 购买一个域名
	1. 用于申请tls证书：加密传输 & 流量伪装
		1. tls：客户端与服务端之间建立安全连接，可对数据包进行加密传输
	2. 加密传输后的数据包与https流量完全一致，难以被识别为vpn流量
	3. CDN & 反向代理：增强ip隐蔽性
3. 添加cloudfare的DNS
	1. 将购买域名的dns解析托管到cloudfare上，状态为：“灰云”
	2. 此时<font color="#ff0000">域名解析结果会直接指向vps</font>
4. 搭建 WebSocket + TLS 协议的v2ray
	1. 不固定，可根据用户喜好选择
	2. v2ray的通信协议是vmess，除此之外还有：Shadowsocks、Trojan、VLESS、WireGuard....
5. 打开 Cloudflare的 CDN
	1. 开启cloudfare的反向代理功能，状态为：“黄云”
	2. 此时访问域名的流量会<font color="#ff0000">经cloudfare网络进行中转</font>，再到vps，从而达到隐藏ip的目的
6. 搭配其它协议并存
