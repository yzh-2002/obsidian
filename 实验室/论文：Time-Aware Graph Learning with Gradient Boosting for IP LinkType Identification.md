
[neurocomputing](https://www.sciencedirect.com/journal/neurocomputing/publish/guide-for-authors) 期刊，elsevier通用模版

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
