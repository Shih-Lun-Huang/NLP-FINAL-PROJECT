# NLP-FINAL-PROJECT
### 簡介：
文字探勘與自然語言處理的期末小專題

指導教授: 吳政隆教授/呂明頴教授

學生: 黃士倫

### 題目：
使用美國大選時期新聞資料集(True.csv)，透過自然語言處理課程的程式碼回答以下三個問題：

![題目](https://github.com/Shih-Lun-Huang/NLP-FINAL-PROJECT/blob/main/img/pb.jpg)

*p.s.題目並未描述的很詳細，可自行發揮*

因此將題目的理解如下：
1. 第一題：將每篇新聞的文章內容透過情感分析**標記正、負、中立**。
2. 第二題：將每篇文章依據彼此的關聯相似程度，決定此資料集的新聞有**多少個主題(群)**。
3. 第三題：找出哪些新聞具有**偏誤**(bias)。這裡根據資料集`subject`欄位與分群結果，找出該新聞可能不適合被歸類在該主題的新聞。

### 流程圖：

![流程圖](https://github.com/Shih-Lun-Huang/NLP-FINAL-PROJECT/blob/main/img/workflow.png)

### 文字前處理方法：
* step.1 刪除重複資料<br/>
* step.2 轉小寫<br/>
  - ex. What -> what…
* step.3 去除標點符號<br/>
  - 去除：!“#$%&\‘()*+,-./:;<=>?@[\\]^_`{|}~“”
* step.4 去除多餘字<br />
  - 去除：\xa0<br />
  - 去除 Stopwords : am、and…

* step.5 文字正規化<br/>
  - 去除字根 （stemming）: ex. started/starting -> start
  - 單複數轉換: （lemmatizing）: ex. papers -> paper

* step.6 去除稀有字<br/>
  - 非必要（降低維度提升效率，這裡只留**詞頻 >1** 的詞）

### 第一題：
- 作法：使用 nltk 的 `SentimentIntensityAnalyzer()` 計算情感分數，該文章如果：
  
    正面分數 > 負面分數就將新聞文章視為正面<br />
    正面分數 < 負面分數就將新聞文章視為負面<br />
    正面分數 = 負面分數就將新聞文章視為中立

- 各情感標籤如下：
  ![情感分析結果](https://github.com/Shih-Lun-Huang/NLP-FINAL-PROJECT/blob/main/img/senti.png)

### 第二題：
- 作法：將新聞內文透過`CountVectorizer()` 轉換成tf-idf矩陣，得到文章向量，透過`t-SNE`降至2維並使用`AgglomerativeClustering`(階層式分群) + `silhouette_score`(輪廓分析法)找出適合的分群數量，決定所有文章大致有多少群（每一群視為一個主題），最後視覺化。p.s.另有嘗試K-means+手肘法找最佳分群數但效果不佳
* 結果：
  - 輪廓分析法決定最佳分群數:
  ![最佳分群數](https://github.com/Shih-Lun-Huang/NLP-FINAL-PROJECT/blob/main/img/group.png)
  - 視覺化結果:
  ![視覺化結果](https://github.com/Shih-Lun-Huang/NLP-FINAL-PROJECT/blob/main/img/tsne.png)
### 第三題：
- 作法：透過視覺化結果可以發現部分群集存在兩個主題(subject)**比例懸殊**的現象，將**該群懸殊比例且較少文章主題**視為**偏誤**（代表在該主題下，有少部分的文章應該適合另一個主題）
* 結果：
  - 下圖整理出`subject`在每個主題的比例，可以發現有些主題`subject`比例懸殊。
  ![懸殊](https://github.com/Shih-Lun-Huang/NLP-FINAL-PROJECT/blob/main/img/bias.png)
  <br /><br />
  訂定bias文章的規則：
    ```python
        if (sub != 'politicsNews'):
            if labels in [0,3,9,13]:#該主題subject比例懸殊且較少
                return 1 #is bias
            else:
                return 0 #not bias
        else:
            if labels in [1,10,15,19]:
                return 1  
            else:
                return 0
    ```
  - 透過bias文章的規則找出的文章共**109**篇。
  ![總共](https://github.com/Shih-Lun-Huang/NLP-FINAL-PROJECT/blob/main/img/bs_total.jpg)