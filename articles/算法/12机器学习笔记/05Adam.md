### SGD

$$
\theta = \theta - \eta g
$$

如有两个变量x1,x2，x1梯度大，x2梯度很小；x1方向就会震荡严重，降低训练速度。

缺点：

* 下降速度慢，容易震荡，陷入鞍点

### Momentum

为了抑制SGD震荡，加入一阶动量，来控制不同方向的梯度，使各个时刻的梯度移动一致 。

一阶动量就是各个时刻梯度的指数加权平均，约等于$1/(1-\alpha)$个时刻的梯度的平均值。
$$
v = \alpha v + \eta g \\
\theta = \theta - v
$$

开始梯度可能会很大，可以设为 $\alpha=0.5$ ；当梯度不那么大时，可以设为 $\alpha=0.9$ 

### Nesterov

Nesterov赋予了动量项预知能力： 

1. SGD另一个问题是容易困在局部最优，解决办法：先跟着累积动量更新一步，看看一情况，根据
2. 动量的方法在最小点梯度过大，容易错过，不知道何时该减速，解决方法：根据之前的动量进行大步跳跃，然后计算梯度进行校正，从而实现参数更新。


$$
\hat{\theta} = \theta + \alpha v \\
v = \alpha v + \eta g \\
\theta = \theta - v
$$

# 自适应学习率

二阶动量：历史梯度的平方和

### AdaGrad

SGD对所有参数更新都是一样的，对于稀疏数据，我们希望对每个不同的参数调整不同的学习率：对频繁变化的参数小幅更新，而稀疏的参数大幅更新。
$$
\theta = \theta - \eta \frac{g_t}{\sqrt{\sum_i^t g_i}}
$$
默认：学习率$\eta = 0.01$ 

优势：在数据分布稀疏的场景，能更好利用稀疏梯度的信息，比标准的SGD算法更有效地收敛。

缺点：主要缺陷来自分母项的对梯度平方不断累积，随之时间步地增加，分母项越来越大，最终导致学习率收缩到太小无法进行有效更新。

### RMSprop

只关注过去一段时间窗口的下降梯度 
$$
v = \alpha v + (1 - \alpha)g_t^2 \\
\theta = \theta - \eta \frac{g_t}{\sqrt{v}+ \epsilon}
$$

默认：学习率$\eta = 0.001$，遗忘因子$\alpha = 0.9$ 

优势：能够克服AdaGrad梯度急剧减小的问题，在很多应用中都展示出优秀的学习率自适应能力。尤其在不稳定(Non-Stationary)的目标函数下，比基本的SGD、Momentum、AdaGrad表现更良好。

### AdaDelta

和RMSprop不同点在于没有参数：学习率$\eta$ ，改为自己学习的一个参数
$$
v = \rho v + (1- \rho)g \\
\theta = \theta - \sqrt{\frac{\Delta x_{t-1} + \epsilon}{v + \epsilon}} g  \\
\Delta x_t = \rho \Delta x_{t-1} + (1- \rho){g'}^2
$$

### Adam

本质RMSprop+Momentum

Adam不仅对累积梯度做指数加权平均，对梯度也进行指数加权平均
$$
m = \beta_1 m + (1- \beta_1)g \\
v = \beta_2 v + (1- \beta_2)g^2 \\
\hat{m} = \frac{m}{1 - \beta_1^t} \\
\hat{v} = \frac{v}{1 - \beta_2^t} \\
\theta = \theta - \eta \frac{m}{\sqrt{v + \epsilon}}
$$
经验参数：$\beta_1 = 0.9 \quad \beta_2 = 0.999$ 

为什么有参数修正：

当迭代次数较小时，m/v接近于0，这个有问题，所以进行了修正。

### Nadm

加上Adam+Nesterov
$$
\hat{\theta} = \theta - \alpha \frac{m}{\sqrt{v}} \\
g = f'() \\
...
$$


 

