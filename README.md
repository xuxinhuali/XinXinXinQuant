# XinXinXinQuant
鑫鑫鑫坤，借鉴世界级量化对冲基金，构造的线性回测平台。

# 使用指南
1. 按照要求处理和配置好标的收益率文件、因子文件。
2. 传入一个json类型的数据即可自动回测。

## json数据示例
```json
{
    "type": "REGULAR",
    "settings": {
        "nanHandling": "ON",
        "instrumentType": "EQUITY",
        "delay": 0,
        "region": "USA",
        "universe": "TOP3000",
        "truncation": 0.08,
        "unitHandling": "VERIFY",
        "testPeriod": "P0D",
        "pasteurization": "ON",
        "language": "FASTEXPR",
        "neutralization": "INDUSTRY",
        "decay": 6,
        "visualization": "OFF"
    },
    "regular": "close"
}
```

## json数据说明
目录:


### type
type: 回测类型，目前正在开发REGULAR, 未来支持SUPER。
REGULAR即普通因子，SUPER即用普通因子构造而成的超级因子

### nanHanding
NaN 处理将 NaN 值替换为其他值。如果 NaNHandling：“On”，则根据运算符类型处理 NaN 值。对于时间序列运算符，如果所有输入都为 NaN，则返回 0。对于每组返回一个值的群组运算符（例如 group_median、group_count），如果一只股票的输入值为 NaN，则返回该组的值.

如果 NaNHandling：“Off”，则保留 NaN。对于时间序列运算符，如果所有输入都为 NaN，则返回 NaN。对于群组运算符，如果一只股票的输入值为 NaN，则返回 NaN。在这种情况下，研究人员应手动处理 NaN。默认设置 NaNHandling 值为“Off”。一些手动处理 NaN 值的方法可以复制 “On” 操作.

### instrumentType
instrumentType: 标的类型，目前正在开发FUND, EQUITY，未来支持所有类型的标的。

### delay
延迟指数据可用性相对于决策时间的时间差。换句话说，延迟（Delay）是指一旦我们决定持仓，我们可以交易股票的时间假设。

假设您在今天的交易结束前看到数据，决定要买入股票。我们可以选择积极的交易策略，在剩余时间内交易股票。在这种情况下，持仓基于当天可用的数据（今天）。这称为“延迟 0 回测”。

或者，我们可以选择一种保守的交易策略，并在第二天（明天）交易股票。然后，持仓是在明天实现的，基于今天的数据。在这种情况下，有一个 1 天的滞后。这称为“延迟 1 回测”。在表达式语言中，延迟是自动应用的，您不必担心它。

### regin
主要支持中国市场，只要有数据，理论上支持任何region。

### universe
标的池是由平台根据基础成交额字段和算法计算出的的交易工具集。例如，“CHN：TOP2000”表示美国市场上最流通的前 3000 只股票（根据最高日均交易额确定）.

### truncation
整体组合中每只股票的最大权重。当它设置为 0 时，没有限制。

允许输入的截断值：0 <= x <= 1 的浮点数（注意：截断的任何值超出此范围都可能影响/破坏回测）。

提示：截断旨在防止过度暴露于个别股票的波动。推荐的设置值为 0.05 到 0.1（涵盖 5%-10%）。

### unitHandling
目前强制为“VERIFY”，单位处理选项允许在运算符中使用到不兼容的量纲时发出警告。如果表达式使用不兼容的数据字段，例如试图将价格加上成交量，则会显示警告.

### testPeriod
测试集的范围。"P0D"表示只使用训练集。可传入"P1Y1M"表示使用最近1年1个月的数据作为测试集，不支持"P1Y1M1D"这种格式。

### pasteurization
消毒将不在 Alpha 股票池中的输入值替换为 NaN。当 Pasteurize =“On” 时，将为未在回测设置中选择的股票池中的工具将输入转换为 NaN。当 Pasteurize =“Off” 时，不会发生此操作，并且将使用所有可用的输入.

消毒数据仅具有 Alpha 股票池中工具的非 NaN 值。虽然消毒数据包含的信息较少，但在考虑横向或组操作时可能更合适。默认的 Pasteurize 设置为“On”。研究人员可以将其切换为“Off”，并使用Pasteurize (x) 运算符进行手动消毒。

假设使用以下设置：Universe TOP500，Pasteurize：“Off”。以下代码计算 Alpha 股票池中 sales_growth 所在行业排名与所有工具中 sales_growth 所在行业排名之间的差异
```text
group_rank(pasteurize(sales_growth),sector) - group_rank(sales_growth,sector)
```
第一组排名中的消毒运算符将输入消毒为 Alpha 股票池（TOP500）中的股票，而第二组等级排名将 sales_growth 在所有股票中进行排名.

### language
平台支持快速表达式"FASTEXPR"。具体可用操作符的文档暂未完成。未来可能支持Python。

### neutralization
中性化是用于使我们的策略市场/行业/子行业中性的操作。当中性化 =“MARKET”时，它执行以下操作:

Alpha = Alpha – mean(Alpha)

其中 Alpha 是权重向量.

实际上，它使 Alpha 向量的平均值为零。因此，与市场相比没有净头寸。换句话说，多头暴露完全抵消了空头暴露，使我们的策略市场中性。

当中性化 = “INDUSTRY”或“SUBINDUSTRY”时，Alpha 向量中的所有股票都分组到对应的行业或子行业中，并分别应用中性化。

### decay
通过将今天的值与前 n 天的衰减值相结合，执行线性衰减函数。它执行以下函数:

![decay公式.jpeg](docs%2Fimages%2Fdecay%B9%AB%CA%BD.jpeg)
 
允许输入的衰减值：整数“n”，其中 n >= 0。注意：使用负数或非整数值进行衰减将破坏回测。

提示：衰减可用于减少周转，但衰减值过大会削弱信号。

### visualization
其他的一些对因子的可视化评估，目前只支持"OFF"。

### regular
普通因子REGULAR的表达式，如"close", "-rank(close)"等。

