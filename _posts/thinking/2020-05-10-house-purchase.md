---
layout: article
title: "购房历程"
modified: 2020-05-10
author: Voyager
toc: false
comments: true
categories: thinking
summary: 完成了最近的一件大事——购房
---

# 成都二手房购房
这次购房是找的链家，过程比较顺利，从签订合同到今天为止一个月，当然还有后续步骤，但是房产证已经拿到了。先记录一下通过中介购房流程

```flow
st=>start: Start
pre=>operation: 准备资格证明(征信,购房资格，收入证明,银行流水)
verify=>condition: 链家校验资格
contract=>operation: 签订购房合同
e=>end
st->pre->verify->contract->e
verify(yes)->e
verify(no)->pre
```

