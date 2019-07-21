---
layout: post
title: "[Python] 이동평균 전략 주식 거래 백테스팅"
subtitle:
author: "MK"
comments: true
sitemap :
  changefreq : daily
  priority : 1.0
categories : [Finance]
tags: [시계열분석]
---



국내 주식 시계열 데이터를 얻을 수 있는 `FinanceDataReader`, 설치 방법은 터미널에서 아래 명령을 실행한다.

```
pip install -U finance-datareader
```

```python
import FinanceDataReader as fdr
fdr.__version__
```


자세한 내용은 가이드 페이지 참고
https://financedata.github.io/posts/finance-data-reader-users-guide.html



Zipline, TA-LIB 등 백테스팅을 지원하는 다양한 라이브러리가 있지만,
`Backtrader`라는 모듈을 활용하였다.


다른 백테스팅 프레임워크를 알고 싶다면 아래 링크 참고
https://www.quantstart.com/articles/backtesting-systematic-trading-strategies-in-python-considerations-and-open-source-frameworks


**<전략설명>**
10일 이동평균선이 20일 이동평균선을 돌파하면 **매수!**
20일 이동평균선이 10일 이동평균선을 돌파하면 **매도!**

```python
from backtesting import Backtest, Strategy
from backtesting.lib import crossover
from backtesting.test import SMA
import FinanceDataReader as fdr
fdr.__version__

class SmaCross(Strategy):
    def init(self):
        Close = self.data.Close
        self.ma1 = self.I(SMA, Close, 10)
        self.ma2 = self.I(SMA, Close, 20)

    def next(self):
        if crossover(self.ma1, self.ma2):
            self.buy()
        elif crossover(self.ma2, self.ma1):
            self.sell()

# 셀트리온, 2018년~2019년6월
data = fdr.DataReader('068270', '20180104','20190630')
print(data.head())

# 초기투자금 10000, commission 비율 0.002 임의 지정
bt = Backtest(data, SmaCross,
              cash=10000, commission=.002)

bt.run()
bt.plot()
```

시뮬레이션 결과는 다음과 같다.<br>
2018.1.1~2019.6.28 기간 중 이동평균 전략으로 투자시 최종 수익률은 104%이다.<br>
2018.11월 최고 159.8%까지 기록하였으며, 최저 수익률은 2019.4월 95.9% 였다.<br>
![img_area](/img/posting/2019-07-18-001-(1).PNG)<br><br>

어느 시점에 매수(▲)해야 할지, 매도(▼)해야 할지를 시각적으로 보여준다.<br>
연두색이 손익이 플러스인 구간, 붉은색이 손익이 마이너스인 구간이다.<br>
![img_area](/img/posting/2019-07-18-001-(2).PNG)<br><br>

아래 그래프를 통해 주가 흐름과 이동평균선 추이를 확인할 수 있다.<br>
![img_area](/img/posting/2019-07-18-001-(3).PNG)<br><br>

![img_area](/img/posting/2019-07-18-001-(4).PNG)<br>



<br>
### **Reference**
---
- <https://github.com/backtrader/backtrader>
- <https://community.backtrader.com/>
- <https://github.com/backtrader/>
