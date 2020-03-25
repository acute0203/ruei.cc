---
layout: post
title: Stock index sentimental analysis with ptt post
date: 2018-02-09 01:40:53
tags: [Data Science,Python,A.I.,sklearn]
categories: Data Mining
---
本篇為作者之前於課堂上自行實作的專案，僅為筆記用途，實驗目的為使用文章預測隔日股價可能落點。
<!--more-->
實驗分為三大部分：
1.文章：爬取PTT股票版的文章，並透過情緒字典轉換成該篇文章的分數，以日期計算當天所有文章累計的情緒分數。
2.股價：測試標的為台積電ADR，轉換為漲跌幅後以KMean分成十群（預設），以分群後的區間作為預測目標。
3.分類：模型使用貝式分類器，訓練方法為以預測當日前一天文章的情緒分數和股價作為訓練資料，因此預測時只要將前一天的情緒分數喂入分類器，便能預測當日股價。

儲存及讀取 JSON File 透過Utils物件。
```python
import json
class Utils():

    def write_json_to_file(json_data, json_file_path):
        with open(json_file_path, 'w') as out_file:
            json.dump(json_data, out_file)
        return 1

    def read_json_from_file(json_file_path):
        with open(json_file_path) as data_file:  
            data = json.load(data_file)
        return data
```

從PTT爬取文章，使用的資料庫為<a href="https://github.com/f496328mm/Crawler_and_Share">496328mm/Crawler_and_Share</a>，本專案直接clone該repository，並放置於libs資料夾中，使用的檔案為Crawler_and_Share資料夾中load_data_from_mysql.py裡的load_data_from_mysql，但在load時需設定不執行。情緒字典來源為<a href="https://github.com/ml-distribution/chinese-corpus">ml-distribution/chinese-corpus</a>，該字典沒有提供分數，因此若來源為正向情緒字典，則設定1分，若來源為負向情緒字典，則設定-1分，每一行所執行的動作可看註解介紹。
```python
import json,time
import os.path
import jieba,re
import datetime
import hashlib
from libs.Crawler_and_Share.load_data_from_mysql import load_data_from_mysql

class PostData():
    def __init__(self):
        self.post_data = load_data_from_mysql(data_name = 'Stock')
        print("Get data from DB done")

    def get_emotion_dict(self, path = "chinese-corpus/emotion-dic/taiwan/", dict_list = ['NTUSD_positive_simplified.txt','NTUSD_negative_simplified.txt']):
        #load emotion dict
        def get_emotion_dict_from_file(path, emotion_score):
            emotion_dict = {}
            with open(path, 'r') as f:
                for line in f:
                    key = line.strip()
                    emotion_dict[key] = int(emotion_score)
            return emotion_dict

        emotion_dict = {}
        for dl in dict_list:
            #using file name to check the file is positive dictionary or negative dictionary
            emo_score = 1 if 'positive' in dl.lower() else -1
            #combine with exist emotion dictionary
            emotion_dict={**emotion_dict, **get_emotion_dict_from_file(path + dl, emo_score)}
        return emotion_dict

    def calc_emotion_score(self, seg_list, emotion_dict):
        news_positive_score=0
        news_negative_score=0
        for word in seg_list:
            if word in emotion_dict:
                if emotion_dict[word] < 0:
                    news_negative_score += abs(emotion_dict[word])
                else:
                    news_positive_score += emotion_dict[word]
        return news_positive_score, news_negative_score

    def custom_clean_article(self ,article):
        article = re.sub('[\d]', '', article)
        article = re.sub('[\n]', '', article)
        return article

    def seg_article(self, article):
        return list(filter(None, jieba.cut(article, cut_all=True)))

    def make_post_data(self, json_file_path = "newsJSONData.json"):
        json_data = {"parsedMD5":[], "dayScore":{}, "startDate": time.strftime("%Y-%m-%d")}
        for post in self.post_data.iterrows():
            #calculate MD5 of article
            article_MD5 = hashlib.md5(post[1]['clean_article'].encode('utf-8')).hexdigest()
            #if the article have been parsed before, there's no need to parse again
            if article_MD5 not in json_data["parsedMD5"]:
                #get the date of the article
                article_date = post[1]['date']
                #change the date to regular format, EX:date format:2007-07-24 13:10:49 to 2007-07-24
                article_date_format = datetime.datetime.strptime(article_date, '%Y-%m-%d %H:%M:%S').strftime("%Y-%m-%d")
                #if the date is before 1911-01-01 ,means incorrect date
                if article_date_format < datetime.datetime.strptime('1911-01-01', '%Y-%m-%d').strftime("%Y-%m-%d"):
                    continue
                #update the oldest date
                if article_date_format < json_data["startDate"]:
                    json_data["startDate"] = article_date_format
                article_date = str(article_date_format)
                #if json do not contains the day's score, initialize day score
                if article_date not in json_data["dayScore"]:
                    json_data["dayScore"][article_date] = [0, 0]
                #use the value of clean_article as article
                article = post[1]['clean_article']
                #using custom cleaner to clean the data
                article = self.custom_clean_article(article)
                #segment the article
                seg_list = self.seg_article(article)
                #get emotion dictionary
                emotion_dict = self.get_emotion_dict()
                #calculate the emotion score
                news_positive_score, news_negative_score = self.calc_emotion_score(seg_list, emotion_dict)
                #add positive score
                json_data["dayScore"][article_date][0] += news_positive_score
                #add negative score
                json_data["dayScore"][article_date][1] += news_negative_score
                #add MD5 to list for record the document have been parsed before
                json_data["parsedMD5"].append(article_MD5)
        #write to json file
        Utils.write_json_to_file(json_data, json_file_path)
        print("The file have been made at " + json_file_path)
```

常見的分類模型行為可分為訓練及預測，因此可以直接實作本介面，在Main Function中不需有太大變動。
```python
class ClassificationModel:
    def __init__(self):
        pass

    def train(self,training_data, training_labels):
        pass

    def predict(self, predict_data):
        pass
```

ClassificationModel的實做，專案中以sklearn中所提供的GaussianNB作為實作對象。
```python
from sklearn.naive_bayes import GaussianNB

class GaussianNaiveBayesClassification(ClassificationModel):
    def __init__(self):
        self.model = GaussianNB()

    def train(self,training_data, training_labels):
        self.model.fit(training_data, training_labels)

    def predict(self, predict_data):
        return self.model.predict(predict_data)

```

從Google中讀取價格資料並作轉換，要注意的是Google所提供的資料隨時可能會停止提供，因此找到新的資料源時，只要重新實作本物件即可。
```python
import pandas_datareader.data as web

class PriceData():

    def __init__(self, start, end, company = "TSM", source = 'google'):
        #fetch data from google
        self.f = web.DataReader(company, source, start, end)
        dates = self._change_date_format()
        self.date_per_change = self._calc_date_price_percentage(dates)

    def _change_date_format(self):
        #check date format
        dates =[]
        for x in range(len(self.f)):
            newdate = str(self.f.index[x])
            newdate = newdate[0:10]
            dates.append(newdate)
        return dates

    def _calc_date_price_percentage(self, dates):
        #store date(key) and change percentage(value) into dictonary
        date_price_percentage={}
        #change to percentage
        last_day_index = 0
        for date in dates:
            current_day_index = self.f.loc[date]['Close']
            current_day_date=date
            if dates.index(current_day_date)!=0:
                #change to percentage
                #if it's the very first day, there's no index to compare so just pass it
                current_day_change_percentage=(current_day_index-last_day_index)*100 / last_day_index if last_day_index !=0 else 0
                #put into dictionary
                date_price_percentage[current_day_date] = current_day_change_percentage
            last_day_index = current_day_index
        return date_price_percentage

    def get_price_data(self):
        return self.date_per_change
```

主要業務邏輯的實作，詳細實作的邏輯已在註解中描述。但要注意的是，本實驗沒有設定測試資料集，僅將整個訓練資料集輸入做預測，如需設定測試資料集，僅須將training_emotion, clustering_result兩物件分割即可。
```python
from datetime import datetime,timedelta
from sklearn.cluster import KMeans
import numpy as np
import os,json,time
import datetime as dt
import os.path

def normalize_emotion_score(emotion_score_list):
    total_score=sum(emotion_score_list) + 1
    return [float(score) / total_score for score in emotion_score_list]

def pare_training_data(date_price, date_article_score):
    training_index=[]
    training_emotion=[]
    for date in date_article_score:
        date_format=datetime.strptime(date, "%Y-%m-%d")
        #add one day
        next_day=str((date_format + timedelta(days = 1)).strftime("%Y-%m-%d"))
        if (next_day in date_price) and (normalize_emotion_score(date_article_score[date])!=0):
            training_index.append([date_price[next_day]])
            training_emotion.append(normalize_emotion_score(date_article_score[date]))
    return training_index,training_emotion

#clustering the stock price with n cluster, where n is cluster number
def price_clustering(training_index, n = 10):
    X = np.array(training_index)
    kmeans = KMeans(n_clusters=n, random_state=0).fit(X)
    clustering_result = kmeans.labels_
    clustering_center = kmeans.cluster_centers_
    return np.array(clustering_result), clustering_center

def main():
    #read date score from json
    json_file_path = "newsJSONData.json"
    #if the file does not exist, then it will load the data from database, which was provided by
    #https://github.com/jwlin/ptt-web-crawler
    if not os.path.exists(json_file_path):
        pdata = PostData()
        pdata.make_post_data(json_file_path)
    json_data = Utils.read_json_from_file(json_file_path)
    #initial date we need to fetch
    start_date = json_data['startDate']
    date_article_score = json_data['dayScore']
    #EX : date_article_score={"2016-12-05":[4,3],"2016-12-06":[1,1],"2016-12-07":[3,10],"2016-12-08":[3,9]}
    #set end date at 2018-02-07
    end_date = datetime.strptime('2018-02-07', '%Y-%m-%d').strftime("%Y-%m-%d")
    print("Article start date:" + str(start_date))
    print("Set up analyze end date:" + str(end_date))
    #init data
    data = PriceData(start_date, end_date, company = "TSM",source = 'google')
    #get date price
    date_price = data.get_price_data()
    #combine date price and date score
    training_index, training_emotion = pare_training_data(date_price, date_article_score)
    #clustering the price into n group, where n by default is set to 10.
    #it will retuurn the clustering result of each data and the center of each group
    clustering_result, clustering_center= price_clustering(training_index, n = 10)

    #classification
    #initialize
    cm = GaussianNaiveBayesClassification()
    #Using clustering result to train classification model.
    cm.train(training_emotion, clustering_result)

    #using the end date to test it can work or not
    end_date = str(end_date)
    if end_date in date_article_score and (normalize_emotion_score(date_article_score[end_date])!=0):
        today_emotion_score = normalize_emotion_score(date_article_score[end_date])
        #working correct
        print("Index close price might close to "+str(clustering_center[cm.predict(np.array([today_emotion_score]))]))
    else:
        #working wrong
        print("Something Wrong...")
        print(date_article_score)
        print()

if __name__ == "__main__":
    main()
```

詳細專案內容可於<a href="https://github.com/acute0203/Stock_index_sentimental_analysis_with_ptt_post">stock_index_sentimental_analysis_with_ptt_post</a>

註：參數仍需調整，本程式僅實驗性質。
