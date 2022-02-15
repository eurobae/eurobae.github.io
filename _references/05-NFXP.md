---
title: "Rust's Nested Fixed Point Algorithm"
permalink: /references/nfxp/
last_modified_at: 2022-02-15
classes: wide
toc: true
use_math: true
comments: true
---

본 포스트는 John Rust가 1987년 Econometrica에 게재한 [Optimal Replacement of GMC Bus Engines: An Empirical Model of Harold Zurcher](https://www.jstor.org/stable/1911259)에 기반하여 만들어졌습니다.

<br>

## 1. Introduction

Rust는 Harold Zurcher라는 한 버스 정비 담당자의 버스 엔진 교체 행태를 설명하기 위한 regenerative optimal stopping 모델을 제안했습니다. Rust의 귀무가설은 "Zurcher가 매 순간 버스 엔진을 교체할지 말지 결정하는 데 있어 optimal stopping rule을 따른다"인데, optimal stopping rule은 분석가들이 관찰하거나(예. 주행거리) 혹은 관찰할 수 없는 여러 state variable들의 함수일 것입니다.

만약 지금 당장 엔진을 교체한다면 즉각적으로 교체 비용이 발생할 것이고, 교체하지 않는다면 언제일지 모르는 미래에 버스가 고장나서 예상치 못한 비용을 지불해야 할 수 있습니다. 따라서 Zurcher는 각 시점마다 엔진을 교체할 때와 교체하지 않을 때 발생 가능한 비용 중 최소 비용을 부담하기 위해 선택을 내릴 것이고, 이에 대한 optimal stopping rule은 stochastic dynamic programming 문제의 solution으로 표현 가능합니다.

<br>

## 2. Model

$$\begin{equation}
    u(x,d,\theta) =
        \begin{cases}
            -RC - c(0,\theta) & \text{if } d=1 \\
            -c(x,\theta)      & \text{if } d=0
        \end{cases}
\end{equation}$$

$$\begin{equation}
    p(x_{t+1} | x_t,d_t,\theta) =
        \begin{cases}
            g(x_{t+1}-0   | \theta) & \text{if } d=1 \\
            g(x_{t+1}-x_t | \theta) & \text{if } d=0
        \end{cases}
\end{equation}$$

Optimal value function $V_{\theta}$는 아래 Bellman's equation의 unique solution입니다.

$$\begin{equation}
\begin{split}
    V_{\theta}(x,\epsilon) &= \max_{d \in D(x)} {[u(x,d,\theta) + \epsilon(d) 
                                               + \beta \int{V_{\theta}(x',\epsilon') \pi(dx',d\epsilon' | x,\epsilon,\theta)}]} \\
                           &= \max_{d \in D(x)} {[u(x,d,\theta) + \epsilon(d)
                                               + \beta EV_{\theta}(x,\epsilon,d)]}
\end{split}
\end{equation}$$

Conditional Independence Assumption (CIA)

$$\begin{equation}
    \pi(x_{t+1},\epsilon_{t+1} | x_t,\epsilon_t,d_t,\theta)
        = p(x_{t+1} | x_t,d_t,\theta) q(\epsilon_t | x_t,\theta)
\end{equation}$$

Conditional choice probability $P(d \vert x,\theta)$는 우리가 잘 아는 multinomial logit 공식을 통해 표현됩니다.

$$\begin{equation}
    P(d | x,\theta) = \frac{ \exp{\{ u(x,d,\theta) + \beta EV_{\theta}(x,d) \}} }
                           { \sum_{d' \in D(x)}{ \exp{\{ u(x,d',\theta) + \beta EV_{\theta}(x,d') \}} } }
\end{equation}$$

Contraction mapping

$$\begin{equation}
\begin{split}
    EV_{\theta}(x,d) &= T_{\theta}(EV_{\theta}(x,d)) \\
                     &= \int{\log{[ \sum_{d' \in D(x')}{\exp{\{ u(x',d',\theta) + \beta EV_{\theta}(x',d') \}}} ] p(dx'|x,d,\theta)}}
\end{split}
\end{equation}$$

<br>

## 3. Data

Ref: https://github.com/OpenSourceEconomics/rust-data


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```


```python
# raw data 읽어오기
g1 = np.loadtxt("./assets/05-NFXP/data/g870.asc").reshape(15,36)
g2 = np.loadtxt("./assets/05-NFXP/data/rt50.asc").reshape(4,60)
g3 = np.loadtxt("./assets/05-NFXP/data/t8h203.asc").reshape(48,81)
g4 = np.loadtxt("./assets/05-NFXP/data/a530875.asc").reshape(37,128)
g5 = np.loadtxt("./assets/05-NFXP/data/a530874.asc").reshape(12,137)
g6 = np.loadtxt("./assets/05-NFXP/data/a452374.asc").reshape(10,137)
g7 = np.loadtxt("./assets/05-NFXP/data/a530872.asc").reshape(18,137)
g8 = np.loadtxt("./assets/05-NFXP/data/a452372.asc").reshape(18,137)
```


```python
pd.DataFrame(g2)
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>50</th>
      <th>51</th>
      <th>52</th>
      <th>53</th>
      <th>54</th>
      <th>55</th>
      <th>56</th>
      <th>57</th>
      <th>58</th>
      <th>59</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2386.0</td>
      <td>5.0</td>
      <td>81.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>133099.0</td>
      <td>136493.0</td>
      <td>139512.0</td>
      <td>140871.0</td>
      <td>142747.0</td>
      <td>145533.0</td>
      <td>148197.0</td>
      <td>149386.0</td>
      <td>151441.0</td>
      <td>153767.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2387.0</td>
      <td>5.0</td>
      <td>81.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>128650.0</td>
      <td>133058.0</td>
      <td>136447.0</td>
      <td>140778.0</td>
      <td>143963.0</td>
      <td>150465.0</td>
      <td>152264.0</td>
      <td>155275.0</td>
      <td>158648.0</td>
      <td>161748.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2388.0</td>
      <td>5.0</td>
      <td>81.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>125177.0</td>
      <td>127820.0</td>
      <td>129933.0</td>
      <td>132691.0</td>
      <td>135919.0</td>
      <td>138935.0</td>
      <td>139956.0</td>
      <td>142125.0</td>
      <td>144834.0</td>
      <td>147206.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2389.0</td>
      <td>5.0</td>
      <td>81.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>114397.0</td>
      <td>118541.0</td>
      <td>123256.0</td>
      <td>126303.0</td>
      <td>130168.0</td>
      <td>131540.0</td>
      <td>134278.0</td>
      <td>135206.0</td>
      <td>138467.0</td>
      <td>142009.0</td>
    </tr>
  </tbody>
</table>
<p>4 rows × 60 columns</p>
</div>




```python
data = None

g_id = 1
for g in [g1,g2,g3,g4,g5,g6,g7,g8]:
    # 헤더: 버스 ID, 구매 일자, 교체 시점 및 odometer readings, 최초 관측 일자 (일자들은 연/월 구분)
    head = pd.DataFrame(g[:,:11], columns=["id","purc_m","purc_y",
                                            "repl_1_m","repl_1_y","repl_1_odo",
                                            "repl_2_m","repl_2_y","repl_2_odo",
                                            "begin_m","begin_y"]).astype(int)
    
    # ym: year-month pairs
    y, m = int(head.begin_y.unique()), int(head.begin_m.unique())-1
    ym = []
    for i in range(pd.DataFrame(g[:,11:]).shape[1]):
        if m ==12:
            y += 1
            m = 1
        else:
            m += 1
        ym.append(str(y)+"-"+str(m))
    
    # 본문: 버스-일자-레벨 패널 데이터 생성
    g_ref = pd.DataFrame(g[:,11:], index=g[:,0].astype(int), columns=ym).stack().reset_index()
    g_ref.columns = ["id","date","odo"]
    g_ref.insert(loc=0, column="group", value=g_id) # 그룹 ID
    
    # 헤더 정보와 패널로 변환한 본문 결합
    head["date"] = head.apply(lambda row: str(row["repl_1_y"])+"-"+str(row["repl_1_m"]), axis=1)
    repl_1 = head[["id","date","repl_1_odo"]].query("repl_1_odo != 0")
    g_ref = pd.merge(g_ref, repl_1, how="left")

    head["date"] = head.apply(lambda row: str(row["repl_2_y"])+"-"+str(row["repl_2_m"]), axis=1)
    repl_2 = head[["id","date","repl_2_odo"]].query("repl_2_odo != 0")
    g_ref = pd.merge(g_ref, repl_2, how="left")

    # 종속변수 replace: 엔진 교체 여부 (0 또는 1)
    g_ref["replace"] = g_ref.apply(lambda row: 1 if row["repl_1_odo"]==row["repl_1_odo"] or row["repl_2_odo"]==row["repl_2_odo"] \
                                   else 0, axis=1)
    g_ref["replace_cum"] = g_ref.groupby("id").replace.cumsum()
        
    # 설명변수 mileage: mileage at replacement
    tmp = g_ref.groupby("id").repl_1_odo.max().to_frame("odo_1").reset_index()
    g_ref = pd.merge(g_ref, tmp, how="outer")
    tmp = g_ref.groupby("id").repl_2_odo.max().to_frame("odo_2").reset_index()
    g_ref = pd.merge(g_ref, tmp, how="outer")
    
    def x_ref(row):
        if row["replace"]==1 and row["replace_cum"]==1:
            return row["repl_1_odo"]
        elif row["replace"]==1 and row["replace_cum"]==2:
            return row["repl_2_odo"] - row["odo_1"]
        elif row["replace"]==0 and row["replace_cum"]==0:
            return row["odo"]
        elif row["replace"]==0 and row["replace_cum"]==1:
            return row["odo"] - row["odo_1"]
        elif row["replace"]==0 and row["replace_cum"]==2:
            return row["odo"] - row["odo_2"]
    g_ref["mileage"] = g_ref.apply(x_ref, axis=1)
    
    # 불필요한 변수 제거
    g_ref.drop(columns=["odo","repl_1_odo","repl_2_odo","replace_cum","odo_1","odo_2"], inplace=True)
    
    # concat
    if data is None:
        data = g_ref
    else:
        data = pd.concat([data, g_ref]).reset_index(drop=True)
    
    g_id += 1
    
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
      <th>group</th>
      <th>id</th>
      <th>date</th>
      <th>replace</th>
      <th>mileage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>4403</td>
      <td>83-5</td>
      <td>0</td>
      <td>504.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>4403</td>
      <td>83-6</td>
      <td>0</td>
      <td>2705.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>4403</td>
      <td>83-7</td>
      <td>0</td>
      <td>7345.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>4403</td>
      <td>83-8</td>
      <td>0</td>
      <td>11591.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>4403</td>
      <td>83-9</td>
      <td>0</td>
      <td>16057.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>15563</th>
      <td>8</td>
      <td>4256</td>
      <td>85-1</td>
      <td>0</td>
      <td>138474.0</td>
    </tr>
    <tr>
      <th>15564</th>
      <td>8</td>
      <td>4256</td>
      <td>85-2</td>
      <td>0</td>
      <td>139320.0</td>
    </tr>
    <tr>
      <th>15565</th>
      <td>8</td>
      <td>4256</td>
      <td>85-3</td>
      <td>0</td>
      <td>140616.0</td>
    </tr>
    <tr>
      <th>15566</th>
      <td>8</td>
      <td>4256</td>
      <td>85-4</td>
      <td>0</td>
      <td>141292.0</td>
    </tr>
    <tr>
      <th>15567</th>
      <td>8</td>
      <td>4256</td>
      <td>85-5</td>
      <td>0</td>
      <td>142426.0</td>
    </tr>
  </tbody>
</table>
<p>15568 rows × 5 columns</p>
</div>




```python
# Table 2a
np.round(data.query("replace==1").groupby("group").mileage.describe())
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
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>27.0</td>
      <td>199733.0</td>
      <td>37459.0</td>
      <td>124800.0</td>
      <td>174550.0</td>
      <td>204800.0</td>
      <td>230650.0</td>
      <td>273400.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>33.0</td>
      <td>257336.0</td>
      <td>65477.0</td>
      <td>121300.0</td>
      <td>215000.0</td>
      <td>264100.0</td>
      <td>292400.0</td>
      <td>387300.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>11.0</td>
      <td>245291.0</td>
      <td>60258.0</td>
      <td>118000.0</td>
      <td>229600.0</td>
      <td>250600.0</td>
      <td>282250.0</td>
      <td>322500.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7.0</td>
      <td>150786.0</td>
      <td>61007.0</td>
      <td>82400.0</td>
      <td>107450.0</td>
      <td>125500.0</td>
      <td>197750.0</td>
      <td>237200.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>27.0</td>
      <td>208963.0</td>
      <td>48981.0</td>
      <td>121000.0</td>
      <td>178650.0</td>
      <td>207700.0</td>
      <td>237200.0</td>
      <td>331800.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>19.0</td>
      <td>186700.0</td>
      <td>43956.0</td>
      <td>132000.0</td>
      <td>162900.0</td>
      <td>182100.0</td>
      <td>188400.0</td>
      <td>297500.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Table 2b
bus_nr = data.groupby("id").replace.sum()
bus_nr = bus_nr[bus_nr == 0].index
np.round(data[data.id.isin(bus_nr)].groupby(["group","id"]).last().mileage.groupby("group").describe())
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
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>15.0</td>
      <td>100117.0</td>
      <td>12929.0</td>
      <td>65643.0</td>
      <td>98580.0</td>
      <td>101423.0</td>
      <td>106280.0</td>
      <td>120151.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.0</td>
      <td>151182.0</td>
      <td>8530.0</td>
      <td>142009.0</td>
      <td>145907.0</td>
      <td>150486.0</td>
      <td>155762.0</td>
      <td>161748.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>21.0</td>
      <td>250766.0</td>
      <td>21325.0</td>
      <td>199626.0</td>
      <td>244599.0</td>
      <td>254183.0</td>
      <td>267988.0</td>
      <td>280802.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5.0</td>
      <td>337222.0</td>
      <td>17802.0</td>
      <td>310910.0</td>
      <td>326724.0</td>
      <td>347549.0</td>
      <td>348475.0</td>
      <td>352450.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1.0</td>
      <td>326843.0</td>
      <td>NaN</td>
      <td>326843.0</td>
      <td>326843.0</td>
      <td>326843.0</td>
      <td>326843.0</td>
      <td>326843.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>3.0</td>
      <td>265264.0</td>
      <td>33332.0</td>
      <td>232395.0</td>
      <td>248376.0</td>
      <td>264356.0</td>
      <td>281698.0</td>
      <td>299040.0</td>
    </tr>
  </tbody>
</table>
</div>


