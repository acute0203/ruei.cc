---
layout: post
title: Association Rule - Apriori Algorithm
categories:
  - Data Mining
date: 2017-03-12 14:11:20
tags: [Python,Data Science,Data Mining]
---

經典的解決啤酒尿布問題演算法。
<!--more-->
具體的作法如下：
1.將所有的項目做成每個unique Item。
2.執行組合產生Sub Item Set。
3.若Sub Item Set的子項目已經被剔除則先移除該Sub Item Set。
3.根據Sub Item Set對其計次。
4.低於Min Support的對其剔除。
5.若待計次的Sub Item Set已經沒東西則脫離迴圈，否則回到step2。
6.根據Minimum Support和Minimum Confidence對不符合條件的進行剔除。

其中Support的算法為該Sub Item Set出現在Database的總次數/Database的資料總數(Denote the frequency of the rule within transactions)。
support(A=>B)=support({A,B})

Confidence的算法為該Sub Item Set出現在Database的總次數/已知該Sub Item Set的總數(Denote the percentage of transactions containing A which also contains B)。
Confidence(A=>B)=support({A,B})/support({A})

另外，由於Apriori演算法是建立在物品存在正關聯的基礎上，因此需要再引入一個Correlation做參考，Correlation算法為
Corr(A,B)=Confidence(A=>B)/Support(B)
當Corr>1時則為正關聯，該Rule較具參考價值，當介於0~1之間則為負關聯，較不具參考加值。

```python
import itertools
#database  data
dataSet={'100':['a','b','c'],'200':['a','c'],'400':['a','d'],'500':['b','e','f']}
#minimum confidence value
minConfidenceScore=0.5
#minimum support value
minSupportScore=0.5
#initialize the needed variable
dtSet={}
supportSet={}
lastDeletedTradeList=[]
#get all unique trading items
allItemsList=sorted({x for v in dataSet.itervalues() for x in v})
allItemLen=len(allItemsList)
totalRecord=float(len(dataSet))
combineChooseNo=1
#algorithm start
while True:
    #combination
    combinationList=[]
    for combineTuple in list(itertools.combinations(allItemsList, int(combineChooseNo))):
        combinationList.append(set(list(combineTuple)))
    #delete the combinationSet which does not need to be analytics
    if len(lastDeletedTradeList)&gt;0:
        for deletedTrade in lastDeletedTradeList:
            combinationList[:]=[combinationSet for combinationSet in combinationList if not deletedTrade.issubset(combinationSet)]
    #The database shoud be scan only if the length of candidate's item set does not equal to 0
    if len(combinationList)&gt;0:
        for tID,items in dataSet.iteritems():
            for combinationSet in combinationList:
                if combinationSet.issubset(set(items)):
                    dtSet[tuple(combinationSet)]=dtSet.get(tuple(combinationSet),0)+1
        for k, v in dtSet.iteritems():
            if v/totalRecord &gt;=minSupportScore:
                supportSet[k]=v/totalRecord
            else:
                lastDeletedTradeList.append(set(list(k)))
        combineChooseNo+=1
    else:
        break
#print out final analytics result
for supportA in supportSet:
    for supportB in supportSet:
        if supportA==supportB:
            continue
        elif set(list(supportA)).issubset(set(list(supportB))):
            if supportSet[supportB]/supportSet[supportA]&gt;=minConfidenceScore:
                confidence=supportSet[supportB]/supportSet[supportA]
                corr=confidence/supportSet[supportB]
                print str(list(supportA))+'-&gt;'+str(list(set(list(supportB))-set(list(supportA))))+',Support:'+str(supportSet[supportB])+',Confidence:'+str(round(confidence,2))+',Corr:'+str(round(corr,2))
#Result
```

另外也可以擴展到高維度。也可視為分類問題(Classification)。
缺點是每次Create Item時都要去掃Database顯得很浪費時間，為改善效率可在期待的規則上做些限制。

參考：
老師書面講義
