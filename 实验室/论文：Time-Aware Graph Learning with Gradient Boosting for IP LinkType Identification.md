
~~[neurocomputing](https://www.sciencedirect.com/journal/neurocomputing/publish/guide-for-authors) 期刊，elsevier通用模版，全年随时可投递，无截稿日期~~
~~中科院二区TOP期刊，CCF-C~~

写作格式：
1. 单列格式（投稿单列，发表双列）
2. Page
3. Abstract：
	1. 不超过250字
4. Keywords：
	1. 1～7个关键字
5. Highlights（非必要）
6. Math formulae：
	1. 对于简单的分数项，使用斜线而不是水平线
	2. 斜体代表变量
	3. 方程式与文本分开，且按文本引用顺序连续编号
7. Tables
8. 图片
	1. 图片需要有标题+图片说明
	2. 根据图像在文章中出现的顺序对其进行编号

IEEE Communications Letters：


## 参考文献


```tex
% natbib包:LaTeX文档系统中一个用于文献引用的强大扩展包
% 此处采用numbers设置引用格式，除此之外还可以设置：authoryear，...
\usepackage[numbers]{natbib} 
% 引用处设置
\cite{}
% 指定参考文献列表的格式样式（忽略了.bst文件后缀）
\bibliographystyle{elsarticle-num-names}
% 指定参考文献来源（忽略了.bib文件后缀）
\bibliography{cas-refs}
```

## 公式，表格，图片


```latex
\begin{figure} % 由begin+end包裹的图片环境
\centering 
% 插入图片命令，width=.9\textwidth 设置图片宽度为文本宽度的90%
\includegraphics[width=.9\textwidth]{figs/cas-munnar-2024.jpg} 
% 添加图片标题
\caption{The beauty of Munnar, Kerala. (See also Table \protect\ref{tbl1}).} 
% 为图片设置一个标签，便于文本中使用\ref{FIG:1}去引用
\label{FIG:1} 
\end{figure}
```

图片文件命名：
1. figure：FIG1/2/3.xxx
2. schema：SC1/2/3.xxx
3. plate：PL1/2/3.xxx

数学公式：

```latex
% 行内公式，使用$$包裹，而不是使用\( \)
% 
\begin{equation}
f(x) = (x+a)(x+b)
\end{equation}
```

## 模型结构

1. 特征提取
	1. 静态特征（节点特征）：IP地址（数值），子网掩码（one-hot），ASN，ISP... 
	2. 动态特征（边特征）：RTT的最小值和方差
		1. 定义一个函数$\mathcal{R}_{v_a,v_b}(x) = \sum_{j=0}^{M} a_j^{v_a,v_b} x^j + \epsilon_{v_a,v_b}(x)$来拟合两个节点之间RTT与跳数x的关系，M为最高多项式次数，该函数的参数的获取可以看作回归任务的解
		2. 将求得的函数参数$a_m$进行标准化得到$z_m$，<font color="#ff0000">每一条边均有一组独特的参数</font>
		3. 每条边的动态特征：RTT最小值，RTT均值，标准化后的多项式参数
2. GBDT：
	1. 初始特征经GBDT输出得到新的节点特征
	2. 新的节点特征作为T-GNN的输入
3. T-GNN：
	1. GNN：
		1. $\mathcal{S}_v' = \frac{1}{M_v} \left(\sum_{i: v_i = v} \mathcal{S}_{\text{head}, i}' + \sum_{j: v_j = v} \mathcal{S}_{\text{tail}, j}' \right)$，$M_v$表示点v相关链路总数，$\mathcal{S}_{\text{head}, i}'$是GBDT输出的特征
		2. $\mathbf{H}_{v}^{t} = \text{ReLU} \left( \text{BN} \left( \text{AGG} \left( \mathcal{S}_v', \sum_{u \in \mathcal{N}(v)} W_1 * \mathbf{H}_{u}^{t-1}, \sum_{e \in \mathrm{E}(v)} W_k * \mathrm{E}_{ve}' \right) + b \right) \right)$，$\mathbf{H}_{v}^{t}$表示经GNN更新后的当前节点表示，综合考虑了原先节点特征$\mathcal{S}_v'$，邻居节点特征，边特征
			1. AGG表示聚合函数，例如：mean，sum，max，attention....
	2. 时间T是怎么引入的？
4. TFN
	1. 链路特征向量表示：$C_{v_av_b}^t = \mathbf{H}_{v_a}^{(L,t)} \oplus \mathbf{H}_{v_b}^{(L,t)} \oplus \mathbf{E}_{v_av_b}^t$
	2. $\hat{y}_{v_av_b} = \text{TFN}\left(\sum_{t=T-T_w}^{T} \gamma^t C_{v_av_b}^t \right)$，考虑一定时间窗口内的所有链路特征之和来预测链路类型
	3. 交叉熵损失

## 实验结果

1. Feature Importance
2. Impact of Polynomial Coefficients on TGBN Performance
	1.  the number of polynomial coefficients：5，10，15，20
	2. original dataset：sichuan，shanghai
	3. new dataset：guangzhou

实验结果图绘制：
1. 配色，可网上搜索科研配色
2. 字体：全部采用Times

## 论文作者

莫李思 杨子涵 蔡泽森 程歆睿 施科任 戴瑞婷（通信/讯）

通信作者：负责与期刊编辑和评审人沟通的作者


```latex
% [1,3] 对应\affiliation编号
\author[1,3]{Lisi Mo}[type=editor,style=chinese]
\comark[1] % 标记为通讯作者
\fnmark[1] % 添加脚注标记
\ead{} % 联系方式
\ead[url]{}
\credit{} % 作者贡献
\affiliation[1]{} %机构信息
```



```latex
\begin{document}
\let\WriteBookmarks\relax % 禁用PDF书签的自动生成,通常与`hyperref`包相关（用于超链接和PDF属性）
\def\floatpagepagefraction{1}
\def\textpagefraction{.001}
\shorttitle{xxx} % 定义文档的简短标题，通常用于页眉或PDF元数据
\shortauthors{xxx} % 定义文档的简短作者列表
```

## 投递

1. 参考文献中出现特殊符号导致编译产出文件乱码：[解决方案](https://blog.csdn.net/Time_Memory_cici/article/details/134587103)
2. 