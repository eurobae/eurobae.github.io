---
title: "Maximum Likelihood Estimation"
permalink: /references/mle/
last_modified_at: 2022-01-01
classes: wide
toc: true
---

본 포스트는 Rice University의 Richard W. Evans가 2018년 7월 QuantEcon에 게시한 [Maximum Likelihood Estimation](https://notes.quantecon.org/submission/5b3b102eb9eab00015b89f8e)를 바탕으로 만들어졌습니다.

# 1. General Characterization of a Model and Data Generating Process

Maximum Likelihood Estimation(MLE), 그리고 이후 포스트에서 다룰 Generalized Method of Moments(GMM)와 Simulated Method of Moments(SMM) 추정법들은 데이터의 특성들 중 일부를 설명하기 위해 모델 파라미터의 값을 

<br>

# 2. Data and Likelihood


```python
import pandas as pd
import numpy as np
import scipy.stats as sts
import requests

import matplotlib.pyplot as plt
```


```python
# Econ381totpts.txt 데이터 불러오기
url = ('https://raw.githubusercontent.com/rickecon/Notebooks/' +
       'master/MLE/data/Econ381totpts.txt')
data = pd.read_csv(url, header=None, names=["points"])
# data.points.to_numpy()
data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>275.50</td>
    </tr>
    <tr>
      <th>1</th>
      <td>351.50</td>
    </tr>
    <tr>
      <th>2</th>
      <td>346.25</td>
    </tr>
    <tr>
      <th>3</th>
      <td>228.25</td>
    </tr>
    <tr>
      <th>4</th>
      <td>108.25</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>156</th>
      <td>235.00</td>
    </tr>
    <tr>
      <th>157</th>
      <td>102.20</td>
    </tr>
    <tr>
      <th>158</th>
      <td>112.30</td>
    </tr>
    <tr>
      <th>159</th>
      <td>130.60</td>
    </tr>
    <tr>
      <th>160</th>
      <td>60.20</td>
    </tr>
  </tbody>
</table>
<p>161 rows × 1 columns</p>
</div>



히스토그램으로 분포를 시각화해 보았습니다.


```python
plt.hist(data, bins=30, density=True, edgecolor='k')
plt.title("ECON 381 scores: 2011-2012", fontsize=16)
plt.xlabel("Points")
plt.ylabel("Percent")
plt.xlim([0, 550])
plt.show()
```


    
![png](output_9_0.png)
    



```python

```


```python

```


```python

```
