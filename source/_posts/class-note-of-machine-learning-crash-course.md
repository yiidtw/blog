---
title: machine learning crash course(mlcc) 修課心得
tags:
  - mlcc
  - machine learning
date: 2018-06-14 23:55:21
---


之前斷斷續續也修了幾門 ML/DL 的課，甚至有過一些小實作，不過這門課稍微走過一遍，許多觀念有連在一起的感覺，有關 tensorflow 實作的部分也不少，還蠻推薦的。也推薦和 Brandon Rohrer 的投影片一起看，兩個份量都不多，mlcc 大概一個禮拜空閒的時間可以掃完，後者大概三天。這兩者是我目前覺得最適合初學者釐清觀念的線上教材，核心都有講到，時間又不常。先附連結
- [Machine Learning Crash Course](https://developers.google.com/machine-learning/crash-course/) 
- [資料科學．機器．人](https://brohrer.mcknote.com/zh-Hant/)

以下是我很粗略的筆記，關於如何開始一個 ML/DL 的 project 所需要的最小知識背景，不含 tf api 操作，也不涉及數學證明

---

## 機器學習能解決的問題
- regression：預測值、機率
- classification：二元或多種分類
- clustering：自動分群
- anomely detection：異常值檢測

比較常遇到的是帶有 label 的監督式學習，應用在 regression 和 classification 的問題，舉個例子
- 根據地區人口、經緯度…等，與房價的資料，建立預測房價模型
- 根據手寫影像與人為 label 的資料，辨識數字

往往我們可以先從簡單、線性的模型開始建立，再漸進複雜，針對不同問題，可以參考微軟的 Algorithm Cheat Sheet，見下圖

![Microsoft Azure Machine Learnning: Algorithm Cheat Sheet](https://docs.microsoft.com/zh-tw/azure/machine-learning/studio/media/algorithm-cheat-sheet/machine-learning-algorithm-cheat-sheet-small_v_0_6-01.png)

## 線性迴歸與神經網路

ML 的演算法可大致分為 distance-based 或是 tree-based ，在 feature 夠好的情況下，不管用哪個 model 理論上都可以給出夠好的結果。 mlcc 沒有提到 tree-based 的演算法，所以以下的小結也僅針對 distance-based 的算法。 distance-based  無論在多高維的空間，本質上都要求該 model 的 loss function 最小化，當 model 過於複雜時，最小化的過程都是尋找 loss function 梯度下降的方向，迭代(iteration)求取 loss function 的最小值，此種方法稱為 gradient decent，來得到最佳的 model 。而梯度下降的速率，稱為 learning rate ，會大大決決定結果的好壞

- linear regression
    - 最簡單也最快的模型
    - 如果只有一個 feature ，而有二元分類，則 y=wx+b  可以視為 2D 平面上用一條線把兩群點分開
    - loss function 是 MSE(Mean Square Error)
    - linear regression 有 exact form ，可以不靠迭代求取最小值
    - 如果要用線性模型學習顯然為非線性的樣本，可以透過 feature engineering 的方式以 cross product 取得新的特徵，照樣以線性模型學習
- logistic regression
    - 基本上就是把 linear regression 丟進 sigmoid function，使得預測值介於 0~1 之間，可以看成 probability
    - loss function 是 logloss，來自 information theory 中 entropy(熵) 的概念
    - 分類問題，可以 mapping 成對於要分的每一類，預測機率，有最高機率的該項即為分類確定
- neural network
    - 要學習簡單的 non-linear model 可以用 cross product 的方式；如果是複雜一點的 non-linear ，可以透過 nn
    - nn 的原理是疊加不同的 hidden layer ，包括線性(linear regression)與非線性 hidden layer 來學習 non-linearity
    - 非線性層是把上一層的 output 經過一個  activation function 達到非線性輸出的 neuron ，主流的 activation funciton 選擇有 ReLU
    - nn 降低 loss function 的演算法稱為 back-prop ，可以視為 gradient decent 的變形
    - 由於 back-prop 需要微分，我們會需要 loss function、activation function 最好是可微，至少是大部分可微

## 如何訓練一個 model

- 首先我們需要先定義問題，或是把問題 mapping 成機器學習可以解決的問題，詳見上述
- 再來要好好的清理資料，甚至進行 feature engineering (特徵工程)，等一下會再描述這件事
- 適當地把資料分成 training set, test set & validation set，其中 training set 要最多 ，而 validation set 最好不要動它，直到自覺 model 成熟，做最後 generalization(泛化) 的確認
- 我們會反覆調整 model 的參數，稱為 hyper-parameter (超參數)，再跑跑看，期望可以達到較成熟的 model 
    - 在 nn 裡，常常需要調整的是 batch size 和 learning rate
- 有兩件事我們很不希望發生，一個是 high bias ，表示 model 效力不足；一個是 high variance ，很有可能 model 的複雜度過高，產生 overfitting (過擬合) 的現象
    - high bias ，也稱為 underfitting ，的解決方式
        - 調整參數
        - 調整 model ，增加 layer 或維度，或 model complexity
        - 增加資料
    - high variance ，也稱為 overfitting ，的解決方式
        - overfitting 比 underfitting 更需要小心， overfitting 常常伴隨 low bias ，也就是 model 在 test set 上取得夠好的成果，卻在 genralization 時效果極差 
        - 在 loss-iteration 的圖中，overfitting 往往伴隨 iterations 越多，數值往上走(曲線上翹)的情形 (見下圖)
        - 避免 over fitting 的技巧，其中一種想法是降低 model complexity ，稱為 regularization (正規化) ，後面再加以描述
        - 如果是一個好的 model 遇到 overfitting ，加入大量高品質的資料會有效降低 overfitting
- 如何衡量一個 model
    - 可以利用 accuracy、precision 和 recall 來衡量是否是一個好的 model
    - 要先知道什麼是 True Positive(TP)、False Positive(FP)、False Negative(FN)、True Negative(TN)
    - 可以這樣想，前面的 True/False 是實際上有沒有發生，後面的 Positive/Negative 是 model 有沒有分類正確，~~如果有人跟你展現機器學習分類的成果，一個裝 B 的方法是，第一個問題就問他 FP rate 如何~~
    - Accuracy = (TP + TN) / (TP + TN + FP + FN)
    - Precision = TP / (TP + FP)
    - Recall = TP / (TP + FN)
    - TP-FP 畫出來的圖(TP 為 y 軸、 FP 為 x 軸) 稱為 ROC curve(Receiver Operating Characteristic curve)

![overfitting](https://i.imgur.com/bPT3Xxa.png)
photo credit: Google mlcc

## feature engineering

mlcc 提供了幾個 good feature 的 properties

- the DOs
    - 一個好的 feature 應該在許多樣本裡有非零值
    - feature 應該要有清楚的意義，而非流水號、隨機的值，或較難 debug 的值，例如一個人的年齡應該用歲數表達，而非 unix time 表示
- the DONTs
    - 不應該有 magic number ，例如不該使用 -1 代表這個值不存在或缺省資料，應該用另一個 boolean feature 表達各筆資料是否有這個欄位的值
    - feature value 不該隨時間改變，例如股票的市值就隨時間改變，違反了 model 的假設需要穩定性
    - 不該採用不理性、誇張的離群值，可以用 bucket 作 binning
- 常用技巧
    - one-hot encoding
    - binning
    - cross product
    - 上述技巧結合使用

## 正規化：避免過於複雜的 model

前面有提到正規化的目的在於降低 model complexity ，來提供可能解決 overfitting 的方法

- L2
    - for simplicity
    - 為了簡化 model complexity
    - 對 weight^2 懲罰
    - 被懲罰的 weight 會趨近極小但不為 0
    - 常用
- L1
    - for sparsity
    - 為了消除稀梳矩陣中較少出現的 weight
    - 對 |weight| (weight 的絕對值) 懲罰
    - 被懲罰的 weight 可為 0
- Dropout
    - nn 裡用來 purge 掉一部分的 weight，詳見[資料科學．機器．人的投影片](https://brohrer.mcknote.com/zh-Hant/how_machine_learning_works/how_neural_networks_work.html)
- Early Stop
    - 提早停止 training iterations

以上，先暫記到這
