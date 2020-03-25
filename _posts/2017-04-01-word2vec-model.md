---
layout: post
title: Word2Vec-詞向量訓練
categories:
  - Deep Learning
date: 2017-04-01 11:52:49
tags: [gensim,Python,Deep Learning,A.I.,Data Science]
---

Word2Vec為一種Auto Encoder的運用，希望在字轉換為向量後，能將詞義（國家：英國 美國）分在較近的距離，透過Sequence Sentence的效果，將模型訓練出來。本文以gensim為例。
<!--more-->
將所載到的wikipedia歷史檔案（本文以[2017/03/20](https://dumps.wikimedia.org/zhwiki/)為例，注意：檔名要載zhwiki-xxxxxxxx-pages-articles.xml.bz2）轉換成text file。(約15分鐘)
```python
# -*- coding: utf-8 -*-
import logging
import sys
import io
from gensim.corpora import WikiCorpus

def main():
    logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
    wiki_corpus = WikiCorpus('zhwiki-20170320-pages-articles.xml.bz2', dictionary={})
    texts_num = 0

    with io.open('wiki_texts.txt','w',encoding='utf-8') as output:
        for text in wiki_corpus.get_texts():
            output.write(b' '.join(text).decode('utf-8') + '\n')
            texts_num += 1
            if texts_num % 10000 == 0:
                logging.info('已處理 %d 篇文章' % texts_num)
if __name__ == '__main__':
    main()
```
將txt檔內的簡中轉換為繁中。
```python
!opencc -i wiki_texts.txt -o wiki_zh_tw.txt -c s2tw.json
```
將轉換成text file的文字檔執行斷詞（斷詞檔案位置需自行調整）。（約半小時）
```python
# -*- coding: utf-8 -*-

import jieba,io
import logging

def main():

    logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

    # jieba custom setting.
    jieba.set_dictionary('dict.txt.big')

    # load stopwords set
    stopwordset = set()
    with io.open('stop_words.txt','r',encoding='utf-8') as sw:
        for line in sw:
            stopwordset.add(line.strip('\n'))

    output = io.open('wiki_seg.txt','w', encoding='utf-8')

    texts_num = 0

    with open('wiki_zh_tw.txt','r') as content :
        for line in content:
            words = jieba.cut(line, cut_all=False)
            for word in words:
                if word not in stopwordset:
                    output.write(word +' ')
            texts_num += 1
            if texts_num % 10000 == 0:
                logging.info('已完成前 %d 行的斷詞' % texts_num)
    output.close()

if __name__ == '__main__':
	main()
```
開始訓練word2vec模型，sg=1的意思為採用skip-gram(0為採用CBOW，差異為何請參考論文)，min_count=10為當字數小於10的全部濾掉，輸出維度為預設的100維。(訓練時間約1小時22分)
[模型檔案下載](https://drive.google.com/a/ntut.org.tw/file/d/0B6salY3K4d-YdlhBaFF3dml2UDg/view?usp=sharing)
```python
# -*- coding: utf-8 -*-

from gensim.models import word2vec
import logging

def main():

    logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
    sentences = word2vec.Text8Corpus('wiki_seg.txt')
    model = word2vec.Word2Vec(sentences, sg=1, min_count=10)

    # Save our model.
    model.save('med100sg.model.bin')

    # To load a model.
    # model = word2vec.Word2Vec.load('your_model.bin')

if __name__ == '__main__':
    main()
```
測試將"飲料"轉換為向量：
```python
import gensim
#load model
model =gensim.models.Word2Vec.load('med100sg.model.bin')
#print vector
print model.wv['飲料'.decode('utf8')]
#print two words similarity
#print model.similarity('飲料'.decode('utf8'),'飲品'.decode('utf8'))
'''
[ 0.08026034  0.60148948  0.11409546  0.04637846 -0.07124737  0.38036227
 -0.11268941 -0.32258135  0.44087785 -0.0557392  -0.04542508 -0.43523914
  0.61505497  0.53459483 -0.0102612   0.16648373  0.3576647   0.44838211
 -0.21971704 -0.63605112 -0.01227834 -0.02652366 -0.08550813 -0.17147315
 -0.59377855  0.08542504 -0.25061882  0.85948586 -0.17427251 -0.20334536
  0.04930361 -0.64287239 -0.2259514   0.00628208 -0.3145048   0.68653703
  0.17605111  0.25807437 -0.26888424 -0.07718194 -0.00229601  0.29866162
 -0.34397081 -0.26891148  0.14278045 -0.14120583  0.55518651 -0.09783766
 -0.16750532  0.07404964  0.16975726  0.18712912  0.19064516  0.42572707
  0.31868255  0.48656365 -0.54355508  0.02244744 -0.77072865  0.4510707
 -0.03568517  0.0275968   0.28584951 -0.19019137 -0.11111313  0.56912392
 -0.47382775 -0.0868965   0.1427363   0.33904704 -0.40692177  0.02225728
  0.74982983 -0.0628393  -0.2577942  -0.74789762  0.09483767  0.44719881
 -0.22698607  0.5958125  -0.69353586 -0.53651321  0.12555401  0.11928654
  0.56912136  0.34537703  0.64645219  0.28430912 -0.06034058 -0.01716865
  0.2866419  -0.03014112 -0.52810687  0.89003199 -0.14196934  0.53036964
 -0.45146766 -0.04470671 -0.04262736 -0.06707814]
'''
```
建議依最新的詞庫訓練模型。
參考：
[Efficient Estimation of Word Representations in Vector Space](https://arxiv.org/pdf/1301.3781.pdf)
[以 gensim 訓練中文詞向量](http://zake7749.github.io/2016/08/28/word2vec-with-gensim/)
[models.word2vec – Deep learning with word2vec](https://radimrehurek.com/gensim/models/word2vec.html)
[Gensim Word2vec简介](http://ju.outofmemory.cn/entry/80023)
