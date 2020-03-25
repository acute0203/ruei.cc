---
layout: post
title: 模擬退火法(Simulated Annealing)
categories:
  - Data Mining
date: 2017-03-14 23:07:41
tags: [Data Science,A.I.,I.T.,python]
---

模擬分子在高溫狀態能無規則性的遊走(Random Walk)，在溫度下降到凝固之前，將解推向Global Minimum。該模型大致步驟為：
1.在目前解附近隨機產生鄰近解待選，若較優則選擇該解，亦引入溫度和當前解為參數，使他有一定的機率跳離Local Minimum。
2.設置降溫機制，並能設置在某溫度的停留時間。
3.回到step1繼續執行，直到達成脫離條件，使解接近可能的最佳解。
<!--more-->
本文採用經典的旅行業務員問題(Traveling Salesman Problem)作為模擬。
程式碼如下：
```python
import math
import random
import time

def calcTwoCityDistance(a, b):
R = 3963  # radius of Earth (miles)
lat1, lon1 = math.radians(a[0]), math.radians(a[1])
lat2, lon2 = math.radians(b[0]), math.radians(b[1])
return math.acos(math.sin(lat1) * math.sin(lat2) +
math.cos(lat1) * math.cos(lat2) * math.cos(lon1 - lon2)) * R

def random2CityArriveOrder(visitCityOrderList):
a = random.randint(0, len(visitCityOrderList) - 1)
b = random.randint(0, len(visitCityOrderList) - 1)
#swap list
visitCityOrderList[a], visitCityOrderList[b] = visitCityOrderList[b], visitCityOrderList[a]
return visitCityOrderList

def calcTotalDistance(distanceMatrix,visitCityOrderList):
totalDistance = 0
for i in range(len(visitCityOrderList)):
totalDistance += distanceMatrix[visitCityOrderList[i-1]][visitCityOrderList[i]]
return totalDistance

def initializeDistanceMatrix(cities):
distanceMatrix = {}
for ka, va in cities.items():
distanceMatrix[ka] = {}
for kb, vb in cities.items():
if kb == ka:
distanceMatrix[ka][kb] = 0.0
else:
distanceMatrix[ka][kb] = calcTwoCityDistance(va, vb)
return distanceMatrix

cities = {
'New York City': (40.72, 74.00),
'Los Angeles': (34.05, 118.25),
'Chicago': (41.88, 87.63),
'Houston': (29.77, 95.38),
'Phoenix': (33.45, 112.07),
'Philadelphia': (39.95, 75.17),
'San Antonio': (29.53, 98.47),
'Dallas': (32.78, 96.80),
'San Diego': (32.78, 117.15),
'San Jose': (37.30, 121.87),
'Detroit': (42.33, 83.05),
'San Francisco': (37.78, 122.42),
'Jacksonville': (30.32, 81.70),
'Indianapolis': (39.78, 86.15),
'Austin': (30.27, 97.77),
'Columbus': (39.98, 82.98),
'Fort Worth': (32.75, 97.33),
'Charlotte': (35.23, 80.85),
'Memphis': (35.12, 89.97),
'Baltimore': (39.28, 76.62)
}
#initialize model parameters
highestTemperature=25000.0
lowestTemperature=2.5
coldRate=0.9
tempIterateCount=100
#initialize a random solution
visitCityOrderList = list(cities.keys())
random.shuffle(visitCityOrderList)
#initialize progress parameters
twoCitiesDistanceMatrix=initializeDistanceMatrix(cities)
temperature=highestTemperature
totalIterate=0

print "Distance before sorting:"+str(calcTotalDistance(twoCitiesDistanceMatrix,visitCityOrderList))
print "Path before calculate:"+"->".join(visitCityOrderList)
startTime = time.time()
while temperature>lowestTemperature and totalIterate<=50000:
#record previous city order list and distance
previousVisitCityOrderList=visitCityOrderList[:]
previousTotalDistance=calcTotalDistance(twoCitiesDistanceMatrix,visitCityOrderList)
#random a possible order list
visitCityOrderList=random2CityArriveOrder(visitCityOrderList)
totalDistance=calcTotalDistance(twoCitiesDistanceMatrix,visitCityOrderList)
#if not accept then replace to previous distance and solution
diff=totalDistance-previousTotalDistance
if diff > 0.0 and math.exp(-diff / temperature) < random.random():
visitCityOrderList=previousVisitCityOrderList
totalDistance=previousTotalDistance
#add counts with iterations
totalIterate+=1
#check the temperature should be cold down or not
if totalIterate%tempIterateCount==0:
temperature*=coldRate
endTime = time.time()
print ""
print "Distance after sorting:"+str(totalDistance)
print "Path after calculate:"+"->".join(visitCityOrderList)
print "Total spend time:"+str(endTime-startTime)+" s"
'''
Distance before sorting:23002.0310634
Path before calculate:Columbus->Jacksonville->Indianapolis->San Antonio->Philadelphia->San Francisco->Dallas->Chicago->San Jose->Baltimore->Memphis->Austin->Los Angeles->Detroit->Phoenix->San Diego->Houston->Fort Worth->Charlotte->New York City

Distance after sorting:8193.51376652
Path after calculate:New York City->Columbus->Phoenix->San Diego->Los Angeles->San Jose->San Francisco->Chicago->Detroit->Indianapolis->Memphis->Dallas->Fort Worth->Austin->San Antonio->Houston->Jacksonville->Charlotte->Baltimore->Philadelphia
Total spend time:0.234772920609 s
'''
```
參考:
1.[模擬退火法(Simulated Annealing)](http://jjcommons.csie.isu.edu.tw/research/download/SA.pdf)
2.[Python module for simulated annealing](https://github.com/perrygeo/simanneal)
3.[Simulated annealing applied to the traveling salesman problem - codecapsule](http://codecapsule.com/2010/04/06/simulated-annealing-traveling-salesman/)
4.老師上課書面講義
