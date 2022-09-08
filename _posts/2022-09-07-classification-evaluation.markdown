---
layout: post
title:  "分类模型的评判标准"
date:   2022-09-07 00:00:00 +0800
categories: machine-learning
---
分类的结果，可以总结有四种，使用混淆矩阵展示：行表示实际类别，列表示预测类别。

<table>
  <thead>
    <tr>
      <td colspan="4">Confusion Matrix</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td colspan="2" rowspan="2">Total</td>
      <td colspan="2">Predicted</td>
    </tr>
    <tr>
      <th>Positive</th>
      <th>Negative</th>
    </tr>
    <tr>
      <td rowspan="2">Actual</td>
      <th>Positive</th>
      <td style="background-color:#ccffcc">TP</td>
      <td style="background-color:#ffcccc">FN</td> 
    </tr>
    <tr>
      <th>Negative</th>
      <td style="background-color:#ffcccc">FP</td>
      <td style="background-color:#ccffcc">TN</td>
    </tr>
  </tbody>
</table>

其中，

- True Positive (**TP**)：模型正确预测了正例；
- True Negative (**TN**)：模型正确预测了负例；
- False Positive (**FP**)：模型错误预测为了正例（本应该是 0，却预测成了 1）；
- False Negative (**FN**)：模型错误预测为了负例（本应该是 1，却预测称了 0）；

**Accuracy**

准确率。是对预测结果的整体评价。一共做了 100 个预测，其中 80 个是对的，准确率为 80%。

$$
\text{accuracy} = \frac{
    \text{number of correct predictions}
}{
    \text{number of total predictions}
}
$$

用混淆矩阵表示：

$$
\text{accuracy} = \frac{
    \text{TP} + \text{TN}
}{
    \text{TP} + \text{TN} + \text{FP} + \text{FN}
}
$$

**Precision**

精度。在全部预测为正例的结果中，真正正确的占多少。结果中一共有 100 个正例，其中 80 个是真正的正例，精度为 80%。或者说，模型认为的正例中，有多少是真正的正例。

$$
\text{precision} = \frac{
    \text{TP}
}{
    \text{TP + FP}
}
$$

**Recall**

召回。全部的正例，有多少被正确找了出来。

$$
\text{recall} = \frac{
    \text{TP}
}{
    \text{TP + FN}
}
$$

**F1-score (F-score)**

通过调整分类的阈值（threshold），可以影响 precision 和 recall 的值。精度与召回通常是负相关的，F-score 可以综合二者给出一个评判标准：

$$
\text{F1} = \frac{
    2
}{
    \frac{1}{\text{precision}} + \frac{1}{\text{recall}}
}
= 2 \frac{
    \text{precision} \cdot \text{recall}
}{
    \text{precision} + \text{recall}
}
$$

**ROC 曲线（Receiver Operating Characteristic Curve）**

ROC 曲线描述的是分类模型在各个阈值下性能。横坐标为 TPR（True Positive Rate），纵坐标为 FPR（False Positive Rate）。

$$
\text{TPR} = \frac{\text{TP}}{\text{TP + FN}}
$$

$$
\text{FPR} = \frac{\text{FP}}{\text{FP + TN}}
$$

AUC（Area Under Curve） 为 ROC 曲线下的面积，AUC 越大模型性能越高。
