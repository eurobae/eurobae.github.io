---
title: "Maximum Likelihood Estimation"
permalink: /references/mle/
last_modified_at: 2022-01-01
classes: wide
toc: true
---

본 포스트는 Rice University의 Richard W. Evans가 2018년 7월 QuantEcon에 게시한 [Maximum Likelihood Estimation](https://notes.quantecon.org/submission/5b3b102eb9eab00015b89f8e){: target="_blank"}를 바탕으로 만들어졌습니다.

<br>

# 1. General Characterization of a Model and Data Generating Process

Maximum Likelihood Estimation(MLE), 그리고 이후 포스트에서 다룰 Generalized Method of Moments(GMM)와 Simulated Method of Moments(SMM) 추정법들은 데이터의 특성들 중 일부를 설명하기 위해 모델 파라미터의 값을 

<br>

# 2. Comparions of Data and Distributions


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```


```python
# Econ381totpts.txt 데이터 불러오기
data = pd.read_csv("https://raw.githubusercontent.com/rickecon/Notebooks/master/MLE/data/Econ381totpts.txt",
                   header=None, names=["points"])
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


    
![png]("/assets/references/mle_01.png")


데이터에서 보여진 분포와 유사할 것으로 생각되는 Truncated Normal 분포 2개를 생성해 보겠습니다.


```python
import scipy.stats as sts
```


```python
# Truncated Normal distribution pdf 생성 함수
def trunc_norm_pdf(xvals, mu, sigma, cutoff=None):
    if cutoff is None:
        prob_notcut = 1.0
    else:
        prob_notcut = sts.norm.cdf(cutoff, loc=mu, scale=sigma)
        
    pdf_vals = 1 / (sigma * np.sqrt(2*np.pi)) * np.exp(- (xvals-mu)**2 / (2*sigma**2) / prob_notcut)
    return pdf_vals
```


```python
dist_pts = np.linspace(0,450,500)

mu_1, sig_1 = 300, 30
plt.plot(dist_pts, trunc_norm_pdf(dist_pts, mu_1, sig_1, 450),
         linewidth=2, label="#1: $\mu$=300, $\sigma$=30")

mu_2, sig_2 = 400, 60
plt.plot(dist_pts, trunc_norm_pdf(dist_pts, mu_2, sig_2, 450),
         linewidth=2, label="#2: $\mu$=400, $\sigma$=60")

plt.legend()
plt.show()
```


    
![png]("/assets/references/mle_02.png")
    


두 Normal 분포에 대해서 Log-Likelihood를 계산해 보겠습니다.


```python
def log_lik_norm(xvals, mu, sigma, cutoff=None):
    pdf_vals = trunc_norm_pdf(xvals, mu, sigma, cutoff)
    log_lik_val = np.log(pdf_vals).sum()
    return log_lik_val
```


```python
print("Log-likelihood #1:", log_lik_norm(data.values, mu_1, sig_1, 450))
print("Log-likelihood #2:", log_lik_norm(data.values, mu_2, sig_2, 450))
```

    Log-likelihood #1: -1552.807742613499
    Log-likelihood #2: -1121.180734031188
    

<br>

# 3. Solving a Maximization (or Minimization) Problem


```python
import scipy.optimize as opt
```
