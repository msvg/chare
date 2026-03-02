# chare
股票量化，包含100+策略，以及数据采集程序，全自动采集分析推荐

提供回测方法

## 代码
![代码 1](https://github.com/msvg/chare/blob/main/img/code1.png)

## 策略 1
![策略 1](https://github.com/msvg/chare/blob/main/img/strategy1.png)

## 策略 2
![策略 2](https://github.com/msvg/chare/blob/main/img/strategy2.png)

## 免费策略-可以自己试验
送大家一个策略，可以作为思路探索
```shell
select sz,dea,diff,macd,lb,zf,hsl,sphl,jzd,name,code,sshy,syl,dqgj,zdf,biz_order as bizOrder,biz_date as bizDate,'策略1' as type from profit t
      where biz_date >= '20251101' and zdf >= 9.88 and dqgj BETWEEN 3 and 120 and sz BETWEEN 2000000000 and 40000000000
        and sphl > 90 and jzd < 10 and lb BETWEEN 0.8 and 3.8 and hsl < 12
        and diff BETWEEN 0.03 and 0.45 and (2 * macd + diff) > 0
        and name not like '*ST%' and name not like 'ST%'
        and EXISTS(select 1 FROM profit_zj x WHERE x.code = t.code and x.bizdate = t.biz_date and x.ddjzb > 7.5 and x.zjjzb > 0)
        and EXISTS(select 1 FROM profit x WHERE x.code = t.code and x.biz_order + 5 >= t.biz_order and t.biz_order > x.biz_order and x.macd > 0)
        and not EXISTS(select 1 FROM profit x WHERE x.code = t.code and x.biz_order + 10 >= t.biz_order and t.biz_order > x.biz_order and x.zdf > 9.85)
        and not EXISTS(select 1 FROM profit x WHERE x.code = t.code and x.biz_order + 3 >= t.biz_order and t.biz_order > x.biz_order and x.zdf < -5)
        and EXISTS(select 1 FROM profit prev WHERE prev.code = t.code and prev.biz_order = t.biz_order - 1 and t.macd > prev.macd)
        and t.dqgj > (select avg(dqgj) from profit x where x.code = t.code and x.biz_order + 120 >= t.biz_order and t.biz_order > x.biz_order)
```

### gemini报告
股票量化策略深度评价报告：低位趋势起爆模型

该策略通过 SQL 逻辑构建了一个复杂的过滤矩阵，核心目标是捕获处于长期上升通道中、经过充分洗盘后、以首板形式突破的强势股。

1. 核心逻辑评价：心理与博弈的极致捕捉

该策略最精妙的地方在于对“筹码分布”与“股价行为”的协同约束：

筹码真空层（核心优势）： sphl > 90（获利盘超90%）配合 jzd < 10（筹码集中度极高）。这意味着绝大多数持仓者都在获利，且筹码集中在少数主力手中。在这种状态下，股价向上突破时几乎没有抛压盘。

“干净”的起爆点： 策略利用 NOT EXISTS 严格剔除了过去 10 天有涨停和过去 3 天有大跌的标的。这实际上是在寻找“沉寂后的第一响”，避开了跟风盘和老庄股的派发期。

2. 技术因子逻辑拆解

因子类型

因子表达式

策略意图（交易学解释）

趋势定性

dqgj > avg(120)

长线牛市基调。确保股票不是在阴跌反弹，而是在半年线上方的多头趋势中。

加速确认

(2 * macd + diff) > 0

这是对 MACD 柱状图斜率的要求。它不仅要求指标为正，还要求其处于发散扩张状态，即动能正在加速。

洗盘识别

not EXISTS(zdf < -5)

过滤情绪化个股。确保起爆前夕走势平稳，没有发生恐慌性抛售，说明主力控盘稳健。

资金背离

t.macd > prev.macd

指标环比增强。要求当前的上涨动能必须强于前一日，确认不是假突破。

3. 策略优劣势分析

优势（Pros）：

高胜率基因： 策略要求的资金流（ddjzb > 7.5）和筹码锁定（jzd < 10）是短线爆发的两大核心引擎。这种组合往往能选出连板潜力股。

安全性（防守性）： 限制市值（20-400亿）和股价（3-120元）避开了流动性陷阱和极度投机的“妖股”，选股更偏向中坚力量。

精准的时机捕捉： 要求 5 天内有 MACD > 0 但 10 天内没涨停，精准定位在“回踩确认后的二次起爆点”。

风险与缺陷（Cons）：

信号极度稀缺： 由于条件过于严苛（特别是 sphl > 90 和 jzd < 10 同时满足），在市场行情一般时，该策略可能连续数周零选股。

对“一字板”敏感： 如果目标股以一字板起爆，成交量（lb）和换手（hsl）可能会不达标导致漏选；而如果开板后再入，可能已经错过最佳位置。

缺乏卖出逻辑： SQL 仅描述了买入点。对于这种高获利盘个股，一旦高位放量下杀，撤退速度将决定最终收益。

4. 综合量化评价

策略类型： 短线突破 / 缩量起爆

适用环境： 震荡向上或牛市初期。

进攻性： ★★★★☆（选出的票极具爆发力）

稳定性： ★★★☆☆（受市场整体赚钱效应影响巨大）

逻辑严密度： ★★★★★（SQL 条件间的互补性极强，逻辑闭环）

5. 评价总结

这是一个**“追求极致质量而非数量”的精英策略。它不看重交易频率，而是利用大数据手段去寻找市场中那 1% 的、筹码结构近乎完美的进攻标的。它不仅仅是在选股票，更是在选“主力已经干完活、准备抬轿子”**的时间点。

## 添加issue
本人会通过issue主动联系有需要的人，非免费。