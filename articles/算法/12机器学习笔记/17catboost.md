参考： https://www.jianshu.com/p/49ab87122562 

俄罗斯大兄弟2017年发表： https://arxiv.org/pdf/1706.09516.pdf 

catboost三个优点：

1. 自动处理类别型特征（categorical features)，自动转成数值型+频率等超参数。
2.  使用了组合类别特征，利用了特征之间的联系。
3. 基模型是对称树，同时计算leaf-value方式和传统不一样，传统式计算平均数，catboost采用了其方法，能防止模型过拟合。

库：pip install catboost

from catboost import CatBoostClassifier

from catboost import CatBoostRegressor

通用参数：

* learning_rate=0.03

* iterations=1000

* max_depth=6

* reg_lambda：L2正则化系数，默认3，可以是任何正值

* one_hot_max_size：对于所有具有多个不同值小于或等于给定参数值的特性，使用one-hot编码。

* loss_function：

    * 分类：默认'Logloss', CrossEntropy, Multiclass
    * 回归：默认'RMSE', MAE, MAPE

    

性能参数：

* thread_count: 默认使用所有线程-1

属性：

* feature_importances_
* best_score_

fit参数：

* cat_features=None：是类别属性的列
* sample_weights=None：样本权重
* logging_level=None：输出日志
* plot=False：训练过程，是否显示
* eval_set=None：验证集（X, y）

代码：

```python
from catboost import CatBoostClassifier
import matplotlib.pyplot as plt
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn import metrics


cancer = load_breast_cancer()
x = cancer.data
y = cancer.target
x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=0.2, random_state=10)
# 指定类别属性列
categorical_columns_indices = []


model = CatBoostClassifier(learning_rate=0.01, 
                           n_estimators=1000,
                           max_depth=4,
                           one_hot_max_size=2,
                           loss_function='Logloss',
                           eval_metric='AUC',
                           use_best_model=True,
                           silent=True,
                           random_seed=100
                          )
model.fit(x_train, y_train, cat_features=categorical_columns_indices, eval_set=(x_test, y_test), plot=False)
# 特征重要性
fea_ = model.feature_importances_
fea_name = model.feature_names_
plt.figure()
plt.barh(fea_name, fea_, height=0.5)
plt.show()
y_pre = model.predict(x_test)
# y_pre = model.predict_proba(x_test)
print(metrics.roc_auc_score(y_test, y_pre))
```

