---
layout: post
title: "[Python] Zipline을 이용한 주식 거래 백테스팅"
subtitle:
author: "MK"
comments: true
sitemap :
  changefreq : daily
  priority : 1.0
categories : [Finance]
tags: [시계열분석]
---



`Quantopian`과 함께 사용되는 Zipline!
단점은 한국 상장기업을 대상으로 하기엔 쉽지가 않다.

우선 해외시장을 타겟으로 샘플코드를 실행해보자.


SPY 즉, S&P500 지수를 활용한 투자전략의 백테스팅이다.
해당 코드는 SPY에 대한 csv 형태의 데이터를 로드한 뒤 작업한다.

아래 링크에서 다운 받을 수 있다.<br>
https://pythonprogramming.net/static/downloads/datasets/SPY.csv


<br><br>
**필요한 라이브러리**

```python
from zipline.api import order, record, symbol, set_benchmark
import zipline
import matplotlib.pyplot as plt
from datetime import datetime
import pandas as pd
from collections import OrderedDict
import pytz
```

## 1. 데이터 가져오기
가져온 데이터(data)는 dict 형태임에 유의하자

```python
full_file_path = "SPY.csv"
data = OrderedDict()
data['SPY'] = pd.read_csv(full_file_path, index_col=0, parse_dates=['date'])
data['SPY'] = data['SPY'][["open","high","low","close","volume"]]
print(data['SPY'].head())
```

| date       | close  |
|------------|---------|
| 1993-01-29 | 43.9375 |
| 1993-02-01 | 44.2500 |
| 1993-02-02 | 44.3437 |
| 1993-02-03 | 44.8125 |
| 1993-02-04 | 45.0000 |



<br>
pandas의 `penel`를 통해 데이터를 becktest에 전달하게 된다.

```python
panel = pd.Panel(data)
panel.minor_axis = ['open','high','low','close','volume']
panel.major_axis = panel.major_axis.tz_localize(pytz.utc) # 2018-01-02 -> 2018-01-02 00:00:00+00:00
print(panel)
```

<br><br>
## 2. 벤치마크 설정
---
벤치마크는 SPY 지수 자체로 설정하였다.

```python
def initialize(context):
    set_benchmark(symbol("SPY"))
```

<br><br>
## 3. 핸들데이터 셋팅
---
`order`를 통해 각 인터렉션마다 10개 단위의 SPY 지수 주문을 설정한다.
그리고 `record`를 통해 각 시점의 가격을 기록한다.

```python
def handle_data(context, data):
    order(symbol("SPY"), 10)
    record(SPY=data.current(symbol('SPY'), 'price'))
```




<br><br>
## 4. 백테스팅 실행
---
백테스팅할 시작, 종료 시점을 설정하고 시작 자산 규모를 셋팅한다.

```python
# 날짜 유의
perf = zipline.run_algorithm(start=datetime(2017, 1, 5, 0, 0, 0, 0, pytz.utc),
                      end=datetime(2018, 3, 1, 0, 0, 0, 0, pytz.utc),
                      initialize=initialize,
                      capital_base=100000,  # 기초자산 설정
                      handle_data=handle_data,
                      data=panel)
```


<br><br>
## 5. 결과 비교
---
테스팅 결과와 벤치마크인 SPY 지수의 수익률 비교를 위해 SPY 일일 수익률을 생성한다.

```python
data = OrderedDict()
data['SPY'] = pd.read_csv(full_file_path, index_col=0, parse_dates=['date'])
data['SPY'] = data['SPY'][["open","high","low","close","volume"]]
data['SPY'] = data['SPY'].resample("1d").mean()
data['SPY'].fillna(method="ffill", inplace=True)
print(data['SPY'].head())
```

이제 그래프를 통해 성과를 비교해보자.

**그래프 그리기**

```python
import matplotlib.pyplot as plt
from matplotlib import style

style.use("ggplot")

perf.portfolio_value.pct_change().fillna(0).add(1).cumprod().sub(1).plot(label='portfolio')
perf.SPY.pct_change().fillna(0).add(1).cumprod().sub(1).plot(label='benchmark')
plt.legend(loc=2)

plt.show()
```

![img_area](/img/posting/2019-07-21-002-result1.png){: .post-img}


알고리즘을 최대로 활용했을 때 누적된 수익률 그래프를 살펴보자.
단, 모든 기회마다 10개의 주식을 살 수 있다는 가정하의 결과이다.

```python
perf.max_leverage.plot()
plt.show()
```

![img_area](/img/posting/2019-07-21-002-result2.png){: .post-img}


<br><br>
### **Reference**
---
- <https://pythonprogramming.net/custom-data-zipline-local-python-programming-for-finance/>
- <https://www.zipline.io/beginner-tutorial.html>
