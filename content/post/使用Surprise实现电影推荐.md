---
title: 使用Surprise实现电影推荐
date: 2019-01-30 17:09:06
tags: [AI]
---

本文记录学习使用Surprise库实现[电影数据集MovieLens](http://files.grouplens.org/datasets/movielens/ml-latest-small.zip)的推荐系统。

<!-- more -->

## 安装

### pip

```
pip install numpy
pip install scikit-surprise
```

### conda

```
conda install -c conda-forge scikit-surprise
```

## 先来看一下Surprise官方的例子

```python
from surprise import SVD
from surprise import Dataset
from surprise.model_selection import cross_validate

# 加载movielens-100k数据集，如果本地没有则会自动下载
data = Dataset.load_builtin('ml-100k')

# 使用SVD算法
algo = SVD()

# 对数据集切成5份进行交叉验证
cross_validate(algo, data, measures=['RMSE', 'MAE'], cv=5, verbose=True)
```

会看到类似如下的输出结果：

```
Evaluating RMSE, MAE of algorithm SVD on 5 split(s).

            Fold 1  Fold 2  Fold 3  Fold 4  Fold 5  Mean    Std
RMSE        0.9311  0.9370  0.9320  0.9317  0.9391  0.9342  0.0032
MAE         0.7350  0.7375  0.7341  0.7342  0.7375  0.7357  0.0015
Fit time    6.53    7.11    7.23    7.15    3.99    6.40    1.23
Test time   0.26    0.26    0.25    0.15    0.13    0.21    0.06
```

## 基于用户的推荐

基本思想是先找到给定用户`User`的相似用户，然后取各相似用户的观影记录推荐给那个给定的用户`User`。下面使用代码来实现。

### 引入必要的包及类

```python
import pandas as pd
from surprise import Reader, Dataset, KNNBaseline
```

### 加载数据

```python
# 使用pandas读取评分数据，以及电影数据
# 评分数据包含四列：userId, movieId, rating, timestamp
# 电影数据包含三列：movieId, title, genres
ratings = pd.read_csv('ml-latest-small/ratings.csv') 
movies = pd.read_csv('ml-latest-small/movies.csv')

# 把两个csv的数据合并成一份数据，以movieId为基准
df = pd.merge(ratings, movies, on='movieId')
df.head()
```

合并后的数据格式如下表

<table>
  <thead>
    <tr>
      <th></th>
      <th>userId</th>
      <th>movieId</th>
      <th>rating</th>
      <th>timestamp</th>
      <th>title</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>4.0</td>
      <td>964982703</td>
      <td>Toy Story (1995)</td>
      <td>Adventure|Animation|Children|Comedy|Fantasy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5</td>
      <td>1</td>
      <td>4.0</td>
      <td>847434962</td>
      <td>Toy Story (1995)</td>
      <td>Adventure|Animation|Children|Comedy|Fantasy</td>
    </tr>
  </tbody>
</table>

看下整个数据集的统计信息：

```python
df.describe()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>userId</th>
      <th>movieId</th>
      <th>rating</th>
      <th>timestamp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>100836.000000</td>
      <td>100836.000000</td>
      <td>100836.000000</td>
      <td>1.008360e+05</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>326.127564</td>
      <td>19435.295718</td>
      <td>3.501557</td>
      <td>1.205946e+09</td>
    </tr>
    <tr>
      <th>std</th>
      <td>182.618491</td>
      <td>35530.987199</td>
      <td>1.042529</td>
      <td>2.162610e+08</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.500000</td>
      <td>8.281246e+08</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>177.000000</td>
      <td>1199.000000</td>
      <td>3.000000</td>
      <td>1.019124e+09</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>325.000000</td>
      <td>2991.000000</td>
      <td>3.500000</td>
      <td>1.186087e+09</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>477.000000</td>
      <td>8122.000000</td>
      <td>4.000000</td>
      <td>1.435994e+09</td>
    </tr>
    <tr>
      <th>max</th>
      <td>610.000000</td>
      <td>193609.000000</td>
      <td>5.000000</td>
      <td>1.537799e+09</td>
    </tr>
  </tbody>
</table>

### 创建训练数据

```python
# 需要使用Reader对象来指定输入数据的一些格式
reader = Reader(rating_scale=(0.5, 5.0))
# 通过surprise.Dataset类的load_from_df方法，加载pandas数据格式
# 只支持user, item, rating三列的数据，所以需要从pandas里面提取对应的三列
# 以rader指定的格式来加载数据
data = Dataset.load_from_df(df[['userId', 'movieId', 'rating']], reader)
# 把加载的数据全部作为训练数据生成训练数据集
trainset = data.build_full_trainset()
```

### 使用基于用户的算法

```python
# 这里基于用户的算法，所以要指定user_bases为True
# 如果是基于物品的算法，user_base指定为False即可
sim_options = {'name': 'pearson_baseline', 'user_based': True}
algo = KNNBaseline(sim_options=sim_options)
algo.fit(trainset)
```

### 实现推荐

假设以用户`1`为指定用户，查找与userId为1的相似用户

```python
# 首先需要把userId转换为surprise内部的id，因为真实的外部userId很有可能是不连续的，不利于计算
inner_user_id = algo.trainset.to_inner_uid(1)
# 通过内部用户id获取邻近（相似）的其他内部用户id
neighbor_users = algo.get_neighbors(inner_user_id, k=5)
print('neighbor users inner id: ', neighbor_users)
# 可能通过to_raw_uid把内部用户id转换为真实的用户id
for u in neighbor_users:
    print('real user id: ', algo.trainset.to_raw_uid(u))
```

#### 查看需要被推荐的用户的记录

```python
df[df.userId == 1].head()
```

指定的用户1的前5条记录如下表

<table>
  <thead>
    <tr>
      <th></th>
      <th>userId</th>
      <th>movieId</th>
      <th>rating</th>
      <th>timestamp</th>
      <th>title</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>4.0</td>
      <td>964982703</td>
      <td>Toy Story (1995)</td>
      <td>Adventure|Animation|Children|Comedy|Fantasy</td>
    </tr>
    <tr>
      <th>215</th>
      <td>1</td>
      <td>3</td>
      <td>4.0</td>
      <td>964981247</td>
      <td>Grumpier Old Men (1995)</td>
      <td>Comedy|Romance</td>
    </tr>
    <tr>
      <th>267</th>
      <td>1</td>
      <td>6</td>
      <td>4.0</td>
      <td>964982224</td>
      <td>Heat (1995)</td>
      <td>Action|Crime|Thriller</td>
    </tr>
    <tr>
      <th>369</th>
      <td>1</td>
      <td>47</td>
      <td>5.0</td>
      <td>964983815</td>
      <td>Seven (a.k.a. Se7en) (1995)</td>
      <td>Mystery|Thriller</td>
    </tr>
    <tr>
      <th>572</th>
      <td>1</td>
      <td>50</td>
      <td>5.0</td>
      <td>964982931</td>
      <td>Usual Suspects, The (1995)</td>
      <td>Crime|Mystery|Thriller</td>
    </tr>
  </tbody>
</table>

#### 查看其他一个相似用户的记录

```python
uid = algo.trainset.to_raw_uid(neighbor_users[0])
df[df.userId == uid].head()
```

某个相似用户的影评记录如下表：
<table>
  <thead>
    <tr>
      <th></th>
      <th>userId</th>
      <th>movieId</th>
      <th>rating</th>
      <th>timestamp</th>
      <th>title</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>203</th>
      <td>597</td>
      <td>1</td>
      <td>4.0</td>
      <td>941557863</td>
      <td>Toy Story (1995)</td>
      <td>Adventure|Animation|Children|Comedy|Fantasy</td>
    </tr>
    <tr>
      <th>363</th>
      <td>597</td>
      <td>6</td>
      <td>3.0</td>
      <td>940420695</td>
      <td>Heat (1995)</td>
      <td>Action|Crime|Thriller</td>
    </tr>
    <tr>
      <th>564</th>
      <td>597</td>
      <td>47</td>
      <td>4.0</td>
      <td>940361541</td>
      <td>Seven (a.k.a. Se7en) (1995)</td>
      <td>Mystery|Thriller</td>
    </tr>
    <tr>
      <th>769</th>
      <td>597</td>
      <td>50</td>
      <td>5.0</td>
      <td>940362491</td>
      <td>Usual Suspects, The (1995)</td>
      <td>Crime|Mystery|Thriller</td>
    </tr>
    <tr>
      <th>825</th>
      <td>597</td>
      <td>70</td>
      <td>2.0</td>
      <td>941559139</td>
      <td>From Dusk Till Dawn (1996)</td>
      <td>Action|Comedy|Horror|Thriller</td>
    </tr>
  </tbody>
</table>

可以看到两个用户之间还是有很大的相似之处的

### 合并推荐结果

最后就是把查找到的相似的用户所有的电影取出来，去掉已经看过的，剩下的就可以推荐给用户1了。

如果觉得结果太多，还可以把所有相似的用户的电影评分求平均，然后取评分最高的几个推荐给用户1。