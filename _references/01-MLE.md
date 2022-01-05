---
title: "Maximum Likelihood Estimation"
permalink: /references/mle/
last_modified_at: 2022-01-01
classes: wide
toc: true
use_math: true
comments: true
---

본 포스트는 Rice University의 Richard W. Evans가 2018년 7월 QuantEcon에 게시한 [Maximum Likelihood Estimation](https://notes.quantecon.org/submission/5b3b102eb9eab00015b89f8e){: target="_blank"}을 기본적으로 번역하되, 필요에 따라 일부 내용을 수정하거나 추가적인 내용들을 덧붙이면서 만들어졌습니다.

<br>

# 1. Maximum Likelihood Estimation

MLE 추정법은 우리가 관측하는 데이터가 어떤 미지의 특정 확률 분포를 따를 것이라는 강한 가정에 기초합니다. MLE의 목표는 데이터를 이용해서 바로 그 확률 분포가 어떻게 생겼을지 추론하는 데 있고, 이를테면 분포의 평균이나 분산과 같은 파라미터 $\theta$를 pin down합니다.

구체적인 추정 방식은 이름에서 알 수 있듯이 likelihood function $L(\theta)$를 극대화하는 $\hat \theta$를 찾도록 전개됩니다. Likelihood는 데이터가 관찰되었을 때 미지의 분포의 파라미터가 $\theta$일 가능성을 뜻하며, 달리 말하면 관측된 데이터가 특정 $\theta$에서 나왔을 확률입니다.

예를 들어 연속형 랜덤 변수 $X$가 $\theta$에 의존하는 $f$라는 PDF(probability density function)를 가진다면, $X=x$라는 하나의 값이 관찰될 때 다음과 같이 likelihood function을 정의할 수 있습니다. (이산형 랜덤 변수라면 probability mass function이겠고요.)

\begin{equation}
    L(\theta | x) = f(x | \theta)
\end{equation}

그리고 i.i.d인 관측치가 $(x_1, ..., x_N)$과 같이 $N$개라면 $L(\theta)$는 PDF의 곱으로 표현됩니다.

\begin{equation}
    L(\theta | x_1, ..., x_N) = \prod_{i=1}^{N} {f(x_i | \theta)}
\end{equation}

Log가 strictly increasing function이기 때문에 $L(\theta)$를 극대화하는 것과 $L(\theta)$를 log 변환한 log-likelihood function $l(\theta)$를 극대화하는 것은 동치입니다. $l(\theta)$는 아래와 같이 곱이 아닌 합으로 표현되겠습니다. ([Log-likelihood가 유리한 이유](https://stats.stackexchange.com/questions/70972/why-we-always-put-log-before-the-joint-pdf-when-we-use-mlemaximum-likelihood/70975#70975){: target="_blank"})

\begin{equation}
    l(\theta | x_1, ..., x_N) = \sum_{i=1}^{N} {\ln{f(x_i | \theta)}}
\end{equation}

$L(\theta)$나 $l(\theta)$를 극대화하는 $\theta$를 찾을 때 깔끔한 analytical solution이 있으면 좋겠지만, 없는 경우에는 Newton-Raphson algorithm과 같은 numerical method가 필요합니다.

우선 Normal 분포를 시작으로 analytical solution이 있는 경우를 살펴보도록 하겠습니다. 평균이 $\mu$, 분산이 $\sigma^2$인 Normal 분포의 PDF는 다음과 같이 주어집니다.

\begin{equation}
    f(x_i | \mu, \sigma^2) = \frac{1}{\sigma\sqrt{2\pi}} \exp{\{ -\frac{(x_i-\mu)^2}{2\sigma^2} \}}
\end{equation}

위에서 정의된 대로 $N$개의 관측치에 대해 $l(\mu,\sigma^2)$는 다음과 같습니다.

\begin{split}
    l(\mu,\sigma^2)
    &= \sum_{i=1}^{N}{ \ln{( \frac{1}{\sigma\sqrt{2\pi}} \exp{\{ -\frac{(x_i-\mu)^2}{2\sigma^2} \}} )} } \\
    &= -N \ln{\sigma} - \frac{N}{2} \ln{2\pi} - \frac{1}{2\sigma^2} \sum_{i=1}^{N}{(x_i-\mu)^2}
\end{split}

이어서 $l(\mu,\sigma^2)$를 극대화하는 $\mu$를 찾기 위해 FOC를 풀면 아래와 같이 $\hat{\mu}$를 구할 수 있습니다.

\begin{equation}
    \frac{\partial}{\partial\mu} l(\mu,\sigma^2) = \frac{1}{\sigma^2} \sum_{i=1}^{N}{(x_i-\mu)} = 0
\end{equation}

\begin{equation}
    \hat{\mu} = \frac{1}{N} \sum_{i=1}^{N}{x_i}
\end{equation}

<br>

# 2. Comparing Observed Data and Distributions


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

import scipy.stats as sts
import scipy.optimize as opt
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


    
<img src="/assets/references/mle_01.png">
    


데이터에서 보여진 분포와 유사할 것으로 생각되는 Truncated Normal 분포 2개를 생성해 보겠습니다.


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


    
<img src="/assets/references/mle_02.png">
    


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

Also see
- https://python.quantecon.org/mle.html
