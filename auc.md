# 00 AUC指标的理解和计算

标签： 广告算法工程师

---

## 00 序言

介绍AUC之前先了解一些基本的指标定义

**混淆矩阵：一个总结分类器所得结果的矩阵，矩阵的大小为分类的类别数。**

二分类的混淆矩阵如下

| 预估 | 真实 | 真实 |
| :-: | :-: | :-: |
|   | 1 | 0 |
| 1 | True Postive(TP) | False Postive(FP) |
| 0 | False Negtive(FN) | True Negtive(TN) |

假阳性率（False Positive Rate）FPR：在所有真实的负样本中，预测结果为正的比例。即$FPR=\frac{FP}{FP+TN}$

真阳性率（True Positive Rate）TPR：在所有真实的正样本中，预测结果为正的比例。即$TPR=\frac{TP}{TP+FN}$

**ROC曲线是以假阳性率FPR为横坐标，真阳性率TPR为纵坐标绘制的曲线。**

绘制ROC曲线：将样本先按照预测分数从高到低排序，对于任意一个0-1之间的阈值，这些样本可以被预测为正或者负，按照预测的正负和真实的label可以计算出FPR和TPR。
![ROC曲线绘制过程](https://s3.bmp.ovh/imgs/2022/04/24/3ced3e20a04f2971.gif)

---

## 01 AUC指标的定义

**AUC: Area Under ROC Curve，即ROC曲线下的面积**

AUC一般用于衡量模型的排序能力，在构成ROC曲线上的点(FPR,TPR)越靠近左上角，FPR越小，TPR越大，那么ROC曲线下的面积就越大，正样本的打分基本要大于负样本的打分。

AUC的另一个定义：**随机从正样本和负样本中各选一个，分类器对于该正样本打分大于该负样本打分的概率**

---

## 02 AUC指标的计算

AUC指标的计算通常可以根据第二个定义来计算

$$
\begin{aligned}
&AUC=\frac{\sum{I(p_{pos}, p_{neg})}}{M*N} \\
&I(p_{pos}, p_{neg})=\begin{cases}
0& p_{pos} < p_{neg}\\
0.5& p_{pos}= p_{neg} \\
1 & p_{pos} > p_{neg}
\end{cases}
\end{aligned}
$$

这个计算公式也可以转化为

$$
\begin{aligned}
AUC=\frac{\sum_{i \in \text{true sample}} rank(i) - \frac{M \times (M+1)}{2}}{M \times N} \quad \text{rank(i)表示正样本i的排序}
\end{aligned}
$$

python实现代码

```python
def AUC(y_ture, y_pred):
    m = sum(y_true)
    n = len(y_true)
    pair = list(zip(y_true, y_pred))
    pair.sort(lambda x: x[1])
    neg_cnt = 0
    sum_rank = 0
    for label, _ in pair:
        if label == 1:
            sum_rank += neg_cnt
        else:
            neg_cnt += 1
    return sum_rank*1.0/(m*n)
```

SQL实现
```SQL
select 
  (rank_sum - 0.5*pos_num*(pos_num+1))/pos_num/neg_num as auc
from
  select
    ,sum(if (label>0, rank, 0)) as rank_sum
    ,sum(if (label==1, 1, 0)) as pos_num
    ,sum(if (label==0, 1, 0)) as neg_num
  from
  (
    select label, pctr
      ,ROW_NUMBER() over(order by pctr asc) as rank
    from table_log
  ) tmp
) final
```

---

## 03 AUC的优缺点

优点：

 - AUC指标本身和模型预测绝对值无关，只关注排序效果
 - AUC对正负类样本是否均衡并不敏感

缺点：

 - AUC只关注正负样本之间的排序，并不关心正样本内部，或者负样本内部的排序
 - AUC只反映了排序能力，没有反映预测精度，分析不出整体高估或低估的情况

---

## 04 AUC的改进GAUC

AUC指标反应的整体的排序能力，但有的时候需要从更加细粒度的维度去看模型的排序能力，所以引入GAUC这个指标

GAUC是计算每个用户的AUC，然后加权平均最后得到的Group AUC，这样可以减少不同用户间的排序结果不太好比较的影响。当然Group也可以不单单只是用户的Group，也可以是不同维度下的Group，类似时间，用户，item等。

GAUC的计算公式

$$
\begin{aligned}
GAUC = \frac{\sum_{g_i}{w_{g_i} \times AUC_{g_i}}}{\sum{w_{g_i}}}
\end{aligned}
$$
 
---

## 05 参考资料

- [看完这篇AUC文章，搞定任何有关AUC的面试不成问题~](https://zhuanlan.zhihu.com/p/360765777)
- [CTR 预测理论（十五）：分类评价指标 AUC 总结（优缺点、计算公式推导）](https://blog.csdn.net/Dby_freedom/article/details/89814644)
- [你真的理解AUC么【搜推广】](https://zhuanlan.zhihu.com/p/360572617)
- [【推荐算法】AUC的计算方法](https://www.cnblogs.com/tmpUser/p/15092467.html)
 
 
