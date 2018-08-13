---
title: 強化學習基礎：多臂吃角子老虎機問題 
date: 2018-08-13 10:33:58
tags:
- algo
- reinforcement learning
- epsilon-greedy 
- multi-armed bandit machine
---

很久沒更新了…

最近研究怎麼寫遊戲的 AI ，原本是想用強化學習，學習遊戲中較優玩家的策略，但這種方法是離線學習的，每天打完要 parse log 重新訓練， AI 才會進步。然後就看了一下強化學習，希望結合 on-line learning 和 off-line learning ，關於系統性的撰寫對局 bot 相關的知識，推薦看徐讚昇教授的電腦對局理論，可惜書中比較針對完全訊息(例如棋類)，以及兩人對局，對於我想寫的非完全訊息(撲克牌)，以及多人對局比較沒辦法直接應用

至於強化學習，看了很多複雜的演算法和嘴砲的文章原本看不太懂，最後看到了吃角子老虎機的問題，終於有一點感覺了，於是稍微整理一下

原理其實很簡單，我們有一隻 k 臂的吃角子老虎機，每支吐幣的機率不同，而我們不知道這個機率如何分布，我們如何在有限回合取得較大的報酬？
- 最簡單的想法是隨機拉臂
- 再來就是透過有限的測試，找到報酬最大的那支臂，狂拉
- 聽起來我們可以結合上述兩種策略，一面探索未知的報酬，一面著力在現有最大報酬的那支臂

所以演算法可以簡述如下，
- 有 ϵ 的機率隨機選臂，好比 ϵ = 0.1
- 紀錄每支臂拉的次數以及平均報酬，另外 1 - ϵ 的機率就拉平均報酬最大的那隻

我改寫了一個 python 2.7 的程式，去證明 ϵ-greedy 演算法的確比隨機拉臂能夠得到更多報酬

```
from numpy import random
from datetime import datetime
import unittest

def logger(log_type, msg):
    try:
        print ' {} {} {}'.format(log_type.upper(), datetime.now().strftime("%Y-%m-%d %H:%M:%S"),  msg)
    except Exception, e:
        print ' ERROR logging' + e.message

class BanditTest(unittest.TestCase):
    def test_basic(self):
        options = list(range(1,6))
        counts = dict(zip(options, [0]*5))
        qlist = dict(zip(options, [0]*5))

        rewards = dict(zip(options, [2, 3, 5, 1, 4]))
        probs = dict(zip(options, [.6, .5, .2, .7, .05]))

        netRewardBandit = bandit(100000, qlist, counts, options, rewards, probs)
        netRewardRandom = random_play(100000, options, rewards, probs)
        self.assertTrue(netRewardBandit > netRewardRandom)


def bandit(epochs, qlist, counts, options, rewards, probs):
    epsilon = .1
    netReward = 0

    for i in range(0, epochs):
        if random.random() < epsilon:
            option = random.choice(options)
        else:
            option = max(qlist, key=qlist.get)
    
        newReward = random.choice([rewards[option],0],p=[probs[option], 1-probs[option]])
        qlist[option] = (qlist[option] * counts[option] + newReward ) / (counts[option] + 1)
        counts[option] = counts[option] + 1
        netReward += newReward

    print ''
    logger('INFO', 'Bandit Algo Net Reward: {}, with Epochs: {}'.format(netReward, epochs))
    return netReward 

def random_play(epochs, options, rewards, probs):
    netReward = 0

    for i in range(0, epochs):
        option = random.choice(options)
        newReward = random.choice([rewards[option],0],p=[probs[option], 1-probs[option]])
        netReward += newReward

    logger('INFO', 'Random Algo Net Reward: {}, with Epochs: {}'.format(netReward, epochs))
    return netReward 

if __name__ == '__main__':
    unittest.main()
```

結果如下，
```
$ python bandit.py 

 INFO 2018-08-13 10:18:57 Bandit Algo Net Reward: 117576, with Epochs: 100000
 INFO 2018-08-13 10:19:04 Random Algo Net Reward: 91752, with Epochs: 100000
.
----------------------------------------------------------------------
Ran 1 test in 13.072s

OK

```

這種帶有隨機味道的算法，次數太少的時候都容易翻盤，隨機算法可能報酬更大，不過隨著次數增加，算法的表現應該會更好，這邊只做了粗淺的紀錄，也沒分析時間複雜度什麼的，衍伸來說，我們很容易可以改成 adaptive 的方法，只要維持平均報酬的 Q table ，以及每支搖臂(每個選項)的次數統計 count table ，我們就能讓 AI 在遊戲中有越來越好的選擇

此外，我們也可以設計一個多個 row 的 Q table / count table，當 AI 需要做決策時，可以依據不同的 game state 去執行 ϵ-greedy 算法，這樣想起來應用真的是蠻廣的，跟上述的例子比起來，上述的演算法只有一個 row 的 Q table / count table ，多 row 的情況就請有需要的讀者自行實作一下

