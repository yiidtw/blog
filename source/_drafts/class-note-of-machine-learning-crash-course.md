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


## 如何訓練一個 model
### train set, test set & validation set
### high bias vs high variance
### 如何衡量一個 model

## 最小化 loss function
### gradient decent
### back propagation

## feature engineering

## 正規化：避免過於複雜的 model
- L1
- L2
- Early Stop
- Dropout

