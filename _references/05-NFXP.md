---
title: "Rust's Nested Fixed Point Algorithm"
permalink: /references/nfxp/
last_modified_at: 2022-02-12
classes: wide
toc: true
use_math: true
comments: true
---

본 포스트는 John Rust가 1987년 Econometrica에 게재한 [Optimal Replacement of GMC Bus Engines: An Empirical Model of Harold Zurcher](https://www.jstor.org/stable/1911259)에 기반하여 만들어졌습니다.

<br>

## 1. Introduction

Rust는 Econometrica 1987년 논문을 통해 Harold Zurcher라는 한 버스 정비 담당자의 버스 엔진 교체 행태를 설명하기 위한 regenerative optimal stopping 모델을 제안했습니다. Rust의 귀무가설은 "Zurcher가 매 순간 버스 엔진을 교체할지 말지 결정하는 데 있어 optimal stopping rule을 따른다"인데, 요컨대 optimal stopping rule은 우리 분석가들이 관찰하거나(예. 주행거리) 혹은 관찰할 수 없는 여러 state variable들의 함수일 것입니다.

만약 지금 당장 엔진을 교체한다면 즉각적으로 교체 비용이 발생할 것이고, 교체하지 않는다면 언제일지 모르는 미래에 버스가 고장나서 예상치 못한 비용을 지불해야 할 수 있습니다. 분명 Zurcher는 각 시점마다 엔진을 교체할 때와 교체하지 않을 때 발생 가능한 비용 중 최소 비용을 부담하기 위해 선택을 내릴 것이고, 이에 대한 optimal stopping rule은 stochastic dynamic programming 문제의 solution으로 표현될 것입니다.

<br>

## 2. Model

\begin{equation}
    u(x,d,\theta) =
        \begin{cases}
            -RC - c(0,\theta) & \text{if } d=1 \\
            -c(x,\theta)      & \text{if } d=0
        \end{cases}
\end{equation}

Optimal value function $V_{\theta}$

$$\begin{equation}
\begin{split}
    V_{\theta}(x,\epsilon) &= \max_{d \in D(x)} {[u(x,d,\theta) + \epsilon(d) 
                                               + \beta \int{V_{\theta}(x',\epsilon') \pi(dx',d\epsilon' | x,\epsilon,\theta)}]} \\
                           &= \max_{d \in D(x)} {[u(x,d,\theta) + \epsilon(d)
                                               + \beta EV_{\theta}(x,\epsilon,d)]}
\end{split}
\end{equation}$$

Conditional choice probability

\begin{equation}
    P(d | x,\theta) = \frac{ \exp{\{ u(x,d,\theta) + \beta EV_{\theta}(x,d) \}} }
                           { \sum_{d' \in D(x)}{ \exp{\{ u(x,d',\theta) + \beta EV_{\theta}(x,d') \}} } }
\end{equation}

$$\begin{equation}
\begin{split}
    EV_{\theta}(x,d) &= T_{\theta}(EV_{\theta}(x,d)) \\
                     &= \int{\log{[ \sum_{d' \in D(x')}{\exp{\{ u(x',d',\theta) + \beta EV_{\theta}(x',d') \}}} ] p()}}
\end{split}
\end{equation}$$

<br>

## 3. Data

https://github.com/OpenSourceEconomics/rust-data

<br>
