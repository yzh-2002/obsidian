## Attention
前面讲的网络模型，例如`FC layer,Convolution layer,pooling layer`等在训练模型时，本质上是对$f({x_1,x_2,...})=\sum_{i}^{n}w_i x_i$进行建模，其中$x_i$表示样本第i维度的特征值（包括显示输入的样本特征以及通过`MLP`形成的交叉特征）。训练的是：`w`，见过的样本越多，越能更好的建立标签与特征间的关系。

`Attention`在训练模型时，本质上是对$f(\vec{x})=\sum_{i}^{n}\alpha(\vec{x},\vec{x_i})v_i$，其中i表示样本个数，训练的是：$\alpha$（注意力评分函数），见过的样本越多，越能更好的捕获输入样本与训练样本之间的联系（从这种角度来看，很像`KNN`）

![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202410121846779.png)

---


## Self-Attention
