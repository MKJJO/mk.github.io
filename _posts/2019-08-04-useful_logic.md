---
layout: post
title: "[Python] 필요할때 꺼내쓰기 좋은 유용한 로직 아이디어"
subtitle:
author: "MK"
comments: true
sitemap :
  changefreq : daily
  priority : 1.0
categories : [Etc]
tags: [IDEA]
---




#### 1. 반복될수록 가산값(노이즈)의 영향력을 점점 작게 만든다.
---
```python
for i in range(1000):
  a = argmax(Q(s,a)) + random_value / (i+1))
```

[Ref.] 강화학습에서의 Exploit VS Exploration: add random noise

<br><br>
#### 2. 반복될수록 누적값을 점점 작게 만든다.
---
γ : 감마라는 변수 ex) 0.9
감마의 거듭제곱을 곱해나간다.

R ~t~ = r ~t~ + γr ~t+1~ + γ ^2^ r ~t+2~ + ... + γ ^n-t^ r ~n~
      = r ~t~ + γR ~t+1~

[Ref.] 강화학습에서의 Discounted future reward
처음의 리워드는 그대로 받고 그 다음 순으로 갈수록 감마의 거듭제곱만큼의 비율이 반영된다.
