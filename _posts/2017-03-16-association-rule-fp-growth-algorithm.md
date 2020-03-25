---
layout: post
title: Association Rule - FP-Growth Algorithm
categories:
  - Data Mining
date: 2017-03-16 16:42:05
tags: [Python,Data Science,Data Mining]
---

Apriori algorithm的改良演算法，Apriori最大的詬病是每次Create candidate item set都必須去掃Database，造成執行緩慢，而FP-Growth只需要掃描兩次Database，大大增加了執行效率。
<!--more-->
其步驟如下：
1.將所有的交易項目做成unique data set。（第一次掃Database）
2.針對每個項目計次。
3.依項目計次個數排序，並將小於min support的剔除。
4.將所有的交易資料依上述結果再做一次編輯（不包含在上表的剔除，並依次數由高到低排列）。（第二次掃Database）
5.依交易資料編入FPTree，若不存在該項目則Create Node,存在則在該Node上計數+1，並創建Link List用以紀錄Node位置。
6.將需要的Condition輸入，依LinkList所記錄的Nodes往樹上緣找尋Item並計次（依該分支Node的次數是為計次次數）。
7.將大於Min Support的次數留下，便是所找尋的關連規則。

```python
minSupport=3
class FPTree():
    def __init__(self):
        #start a root node
        self.root=FPNode(itemName='root',parentNode=None)
        #trace route linklist table
        #should be look like {'a':[node1,node2],'b':[node3,node4]}
        #which means if condition is a, it will trace from node 1,node 2 toward root
        self._route={}

    def add(self,transactionRecordList):
        #always start from root
        node=self.root
        print 'Get transaction '+str(transactionRecordList)+' to analytics..'
        print 'start from ''+node.getItemName()+'''
        for transaction in transactionRecordList:
            print 'checking ''+transaction+'' in ''+node.getItemName()+''...'
            if node.isInChildSet(transaction):
                print '''+transaction+'' is in node ''+node.getItemName()+'''
                node=node.getFromChildSet(transaction)
                print 'change node to ''+node.getItemName()+'''
            else:
                #create a new node
                newNode=FPNode(itemName=transaction,parentNode=node)
                print 'new node ''+newNode.getItemName()+'' have been created,parent node is ''+newNode.getParentNode().getItemName()+'''
                node.addToChildSet(transaction,newNode)
                print 'node '+node.getItemName()+''s child set:'+str(node.getChildSet())
                if transaction not in self._route:
                    self._route[transaction]=list()
                self._route[transaction].append(newNode)
                #print 'new route have been add,current route's key '+str(transaction)+',path '+str(self._route[transaction])
                node=newNode
                print 'change node to ''+node.getItemName()+'''
            node.addCount()
            print 'node count is '+str(node.getCount())

    def getRoute(self):
        return self._route

    def conditionSearch(self):
        for condition,nodeList in self._route.iteritems():
            condictionDict={}
            for node in nodeList:
                checkNode=node
                originalNode=node
                while True:
                    checkNode=checkNode.getParentNode()
                    if checkNode:
                        condictionDict[checkNode.getItemName()]=condictionDict.get(checkNode.getItemName(),0)+originalNode.getCount()
                    else:
                        #toward root
                        break
            for item, support in condictionDict.iteritems():
                if support&gt;=minSupport and item !='root':
                    print 'condition:'+condition+', item:'+str(item)+', support:'+str(support)
            #print condictionDict
class FPNode():
    def __init__(self,itemName,parentNode):
        self.itemName=itemName
        self.count=0
        self._child={}
        self.parentNode=parentNode

    def addCount(self):
        self.count+=1

    def isInChildSet(self,transaction):
        return transaction in self._child

    def getParentNode(self):
        return self.parentNode

    def getItemName(self):
        return self.itemName

    def getCount(self):
        return self.count

    def addToChildSet(self,transaction,node):
        self._child[transaction]=node

    def getFromChildSet(self,transaction):
        return self._child[transaction]

    def getChildSet(self):
        return self._child.keys()

simpRecordDataList = {100:['a','b','c','d','e','f','g','h'],  
           200:['a','f','g'],  
           300:['b','d','e','f','j'],  
           400:['a','b','d','i','k'],  
           500:['a','b','e','g']}
#create head table
headerTableCandidateItemList=list(set(item for TID,sublist in simpRecordDataList.iteritems() for item in sublist))
headerTableCandidateFreqList=[0]*len(headerTableCandidateItemList)
for TID,sublist in simpRecordDataList.iteritems():
    for transaction in sublist:
        headerTableCandidateFreqList[headerTableCandidateItemList.index(transaction)]+=1
headerTable=[]
for (itemCount,item) in sorted(zip(headerTableCandidateFreqList,headerTableCandidateItemList),reverse=True):
    if itemCount&gt;=minSupport:
        headerTable.append([item,itemCount])
print 'HEADER TABLE'
for ht in headerTable:
    print ht
#recounstruct trading data
reconstructSimpRecordDataList = {}
for TID,subList in simpRecordDataList.iteritems():
    reconstructSimpRecordDataList[TID]=[]
    for [item,itemCount] in headerTable:
        if item in subList:
            reconstructSimpRecordDataList[TID].append(item)
#print reconstructSimpRecordDataList
fpt=FPTree()
print ''
for TID,tranList in reconstructSimpRecordDataList.iteritems():
    print 'Sending '+str(TID)
    fpt.add(tranList)
fpt.getRoute()
print '\nResult:'
fpt.conditionSearch()
'''
Result:
condition:a, item:b, support:3
condition:e, item:b, support:3
condition:d, item:b, support:3
condition:g, item:a, support:3
'''
```
參考：
1.老師上課講義
2.[不產生候選集的關聯規則挖掘算法FP-Tree-壹讀](https://read01.com/BnxAkB.html)
3.[enaeseth/python-fp-growth-Github](https://github.com/enaeseth/python-fp-growth)
4.[[ ML In Action ] Unsupervised learning : Efficiently finding frequent itemsets with FP-growth](https://puremonkey2010.blogspot.tw/2013/09/ml-in-action-unsupervised-learning.html)
