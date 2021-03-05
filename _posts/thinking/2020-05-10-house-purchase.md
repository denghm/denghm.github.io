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
contract=>operation: 链家，交易双方，签订定金合同，缴纳定金15万
op1=>operation: 办理房产转让，获取购房资格
contract2=>operation: 签订购房合同，约定补充合同，缴纳中介费
op2=>operation: 天府中心，交易双方，申请贷款
op3=>operation: 房产中心，首付款资金监管，过户
op4=>operation: 房产中心，隔天领取房产证
op5=>operation: 发放尾款
e=>end
st->pre->verify->contract->op1->contract2->op2->op3->op4->op5->e
verify(yes)->contract
verify(no)->pre
```

> 征信证明是人民银行提供，到招商银行自助打印
> 申请贷款时因为四海逸家产权面积小，只有中信，工商银行可以贷款
> 签订定金合同，链家就托管卖方的房产证，同时网站下架房源
> 
