---
title: machine learning crash course(mlcc) 修課心得
tags:
- mlcc
- machine learning

---

之前斷斷續續也修了幾門 ML/DL 的課，甚至有過一些小實作，不過這門課稍微走過一遍，許多觀念有連在一起的感覺，有關 tensorflow 實作的部分也不少，還蠻推薦的。也推薦和 Brandon Rohrer 的投影片一起看，兩個份量都不多，mlcc 大概一個禮拜空閒的時間可以掃完，後者大概三天。這兩者是我目前覺得最適合初學者釐清觀念的線上教材，核心都有講到，時間又不常。先附連結
- [Machine Learning Crash Course](https://developers.google.com/machine-learning/crash-course/) 
- [資料科學．機器．人](https://brohrer.mcknote.com/zh-Hant/)

以下是我很粗略的筆記，不涉及數學證明

---

## 機器學習能解決的問題
- regression：預測值、機率
- classification：二元或多種分類
- clustering：自動分群
- anomely detection：異常值檢測

比較常遇到的是帶有 label 的監督式學習，應用在 regression 和 classification 的問題，也就是說
- 根據地區人口、經緯度…等，與房價的資料，建立預測房價模型
- 根據手寫影像與人為 label 的資料，辨識數字

往往我們可以先從簡單、線性的模型開始建立，再漸進複雜，針對不同問題，可以參考微軟的 Algorithm Cheat Sheet，見下圖

![Microsoft Azure Machine Learnning: Algorithm Cheat Sheet](https://docs.microsoft.com/zh-tw/azure/machine-learning/studio/media/algorithm-cheat-sheet/machine-learning-algorithm-cheat-sheet-small_v_0_6-01.png)

## 線性迴歸與神經網路

ML 的演算法可大致分為 distance-based 或是 tree-based ，在 feature 夠好的情況下，不管用哪個 model 理論上都可以給出夠好的結果。 mlcc 沒有提到 tree-based 的演算法，所以以下的小結也僅針對 distance-based 的算法。 distance-based  無論在多高維的空間，本質上都要求該 model 的 loss function 最小化，當 model 過於複雜時，最小化的過程都是尋找 loss function 梯度下降的方向，迭代(iteration)求取 loss function 的最小值，此種方法稱為 grdient decent，來得到最佳的 model 。而梯度下降的速率，稱為 learning rate ，會大大決決定結果的好壞

- linear regression
    - 最簡單也最快的模型
    - 如果只有一個 feature ，而有二元分類，則 y=wx+b  可以視為 2D 平面上用一條線把兩群點分開
    - loss function 是 MSE(Mean Square Error)
    - linear regression 有 exact form ，可以不靠迭代求取最小值
    - 如果要用線性模型學習顯然為非線性的樣本，可以透過 feature engineering 的方式以 cross product 取得新的特徵，照樣以線性模型學習
- logistic regression
    - 基本上就是把 linear regression 丟進 sigmoid function，使得預測值介於 0~1 之間，可以看成 probability
    - loss function 是 logloss，來自 information theory 中 entropy(熵) 的概念
    - 分類
- neural network
    - 要學習簡單的 non-linear model 可以用 cross product 的方式；如果是複雜一點的 non-linear ，可以透過 nn
    - nn 的原理是疊加不同的 hidden layer ，包括線性(linear regression)與非線性 hidden layer 來學習 non-linearity
    - 非線性層是把上一層的 output 經過一個  activation function 達到非線性輸出的 neuron ，主流的 activation funciton 選擇有 ReLU
    - nn 降低 loss function 的演算法稱為 back-prop ，可以視為 gradient decent 的變形
    - 由於 back-prop 需要微分，我們會需要 loss function、activation function 最好是可微，至少是大部分可微

## 如何訓練一個 model
- 首先我們需要先定義問題，或是把問題 mapping 成機器學習可以解決的問題
- 再來要好好的清理資料，甚至進行 feature engineering
- train set, test set & validation set
- high bias vs high variance
- 如何衡量一個 model

## 最小化 loss function
### gradient decent
### back propagation

## feature engineering

## 正規化：避免過於複雜的 model
- L1
- L2
- Early Stop
- Dropout
