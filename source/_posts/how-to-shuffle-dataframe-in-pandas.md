---
title: Pandas Dataframe 隨機 shuffle 小技巧
date: 2018-05-29 14:51:33
tags:
- pandas
- dataframe
---

在做 learning 的時候會需要先把 pandas 的 dataframe 的 order 打亂，有幾種方法可以做到，稍微紀錄一下，我個人是比較喜歡 sklearn 的方法啦…
以下要在 jupyter notebook 或 python script 裡執行， assume 已經安裝 pandas 並 import

### mlcc 裡的方法
``` 
> california_housing_dataframe = california_housing_dataframe.reindex(np.random.permutation(california_housing_dataframe.index))
```

### sample 法
```
> california_housing_dataframe = california_housing_dataframe.sample(frac=1).reset_index(drop=True)
```

### sklearn shuffle 法
```
> from sklearn.utils import shuffle
> california_housing_dataframe = shuffle(california_housing_dataframe)
```

---

## Reference
- [python - Shuffle DataFrame rows - Stack Overflow](https://stackoverflow.com/questions/29576430/shuffle-dataframe-rows?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
