# Adaboost

Boosting算法需要解决的问题：

1. 如何计算误差e?
2. 如何计算若学习器权重$\alpha$?
3. 如何更新样本权重？

### Adaboost二分类算法流程

设样本个数为n，迭代器个数为m, 基分类器f, 强分类器F

AdaBoost算法步骤：

1. 初始化样本权重，$w = (w_{m1}, \cdots,w_{mn})$, $w_{mi}=1/n$ 
2. 共进行M轮学习，第m轮为：
    * 使用样本得到基分类器$f_m(x)$ 
    * 计算误差：$ e_m = \sum_{i=1}^{n} w_{mi}I(f_{m} \neq y_i)$ 
    * 计算$f_m$的权重：$\alpha_m = \frac{1}{2}log \frac{1-e_m}{e_m}$ 
    * 更新样本权重：$w_{m+1, i} = \frac{w_{mi}}{Z}exp(-\alpha_m y_i f_m(x_i))$ ,其中z是归一化因子
3. 得到最终分类器：$F_m = \sum_{i=1}^{m} \alpha_i f_i$ 



### 推导过程

定义指数函数为损失函数：
$$
L(y, F_m) = exp(-yF_m)
$$
所以要求解的是：
$$
(\alpha_m, f_m) = argminL = argmin \sum_{i=1}^{N} exp(-y_i (F_{m-1} + \alpha_mf_m))
$$
令：
$$
w_{mi} = exp(-y_i F_{m-1}) \\
e_m = \sum_{i=1}^{N} w_{mi}I(f_m(x_i) \neq y_i)
$$
于是有：
$$
\begin{align}L &= \sum exp(-y_iF_{m-1})exp(-y_i\alpha_m f_m) \\
&=\sum w_{mi}exp(-y_i \alpha_m f_m) \\
&=\sum w_{mi}e^{-\alpha}I(y_i = f_m(x_i)) + \sum w_{mi}e^{\alpha}I(y_i \neq f_m(x_i)) \\
&= e^{-\alpha} \sum  w_{mi} + (e^{\alpha}-e^{-\alpha})\sum w_{mi}I(y_i \neq f_m(x_i)) \\
&=e^{-\alpha} \sum  w_{mi} + (e^{\alpha}-e^{-\alpha})e_m \\
&=e^{-\alpha} + (e^{\alpha}-e^{-\alpha})e_m
\end{align}
$$

令$\alpha$导数为零，得：
$$
\alpha = \frac{1}{2} ln \frac{1-e_m}{e_m}
$$
上面$w_{mi}$实际为$\widehat {w_{mi}}$ ,而 $w_{m+1, i}$为：
$$
\begin{align} w_{m+1, i} &= exp[-y_i (F_{m-1}+\alpha_m f_m)] \\
&=\widehat{w_{mi}} exp[-y_i \alpha_m f_m ]
\end{align}
$$
最终迭代式子为：
$$
w_{m+1, i} = \frac{w_{mi}}{Z}exp(-y_i \alpha_m f_m)
$$
其中z是规范化因子，是所有分子之和。

### Adboost回归

回归变种有很多，只不过都是在解决上面是三个问题。

# GBDT

提升树：boosting方法都是

Adaboost思想：根据误差更新样本权重，训练多个弱学习器

GBDT思想：拟合上一轮损失

若损失函数是平方损失、指数函数，每一步优化简单，但是对于一般损失函数，如何计算损失？

大牛Freidman提出用损失函数的负梯度来拟合本轮损失的近似值

为什么可以拟合负梯度？有两种解释：

**损失函数的数值优化可以看成是在函数空间，而不是在参数空间** 参考Freidman论文

1. SGD优化的是参数的负梯度方法
2. 将函数代替参数：$-g_m(x) = -\frac{\partial L(y_i, F_{m-1})}{\partial F_{m-1}}$

**泰勒展开**

$f(x) = f(x_0) + f'(x_0)(x-x_0) + \frac{1}{2}f''(x_0)(x-x_0)^2 + Rn(x)$ 

将损失函数一阶泰勒展开：

$L[y_i, F_{m-1}+\beta_m f_m(x_i)] = L(y_i, F_{m-1}) + \beta_m f_m \frac{\partial L(y_i, F_{m-1})}{\partial F_{m-1}}$  

由于步长$\beta>0$ ，所以令$f_m = - \frac{\partial L(y_i, F_{m-1})}{\partial F_{m-1}}$ , 损失就会越来越小。

### GBDT回归

损失：平方损失

算法流程：

1. 初始化学习器$F_0$ 
2. 共进行M轮学习，第m轮为：
    * 计算损失，平方损失就是：$\hat{y} = y_i - F_{m-1}(x_i)$ 	
    * 得到第m颗树，其叶子可表示为$R_{mj}$ , j表示叶子个数
    * 计算每个叶子最佳拟合值：$c_{mj}=argmin \sum\limits_{x_i \in R_{mj}} L(y_i, F_{m-1} + c)$ ，其实就是（$\hat{y}$ 的平均值）
    * 得到强学习器：$F_m = F_{m-1} + \sum_j c_{mj}I(x \in R_{mj})$ 
3. 得到最终强学习器

### GBDT二分类

损失函数：指数损失，GBDT退化为Adaboost

损失：负二项对数似然损失函数$L(y, F_m) = log(1+exp(-yF_m))$ 

负梯度为：$\hat{y_i} = \frac{y_i}{1+exp(y_iF_{m-1})}$ 

最优叶子节点值，一般用近似值代替：$c_{mj} = \frac{\sum_{R_j} \hat{y_i}}{\sum_{Rj} |\hat{y_i}|(1-|\hat{y_i}|)}$  

### GBDT多分类

损失：交叉熵：$L(y, F_m) = - \sum y_k log p_k$ 

对于多分类问题，在构建基学习器时，我们要为每个类别k创建一棵回归树。

负梯度：第m轮，第i个样本被分类为k： $\hat{y_{ik}} = y_{ik} - p_{m-1,k}(x_i)$ 

叶子节点近似值为：$c_{mjk} = \frac{k-1}{k} \frac{\sum_{R_{mjk}}r_{mik} }{sum_{R_{mjk}}|r_{mik}|(1-|r_{mik}|)}$ 









