# scale 度量
scale度量，是数据空间到图形空间的转换桥梁，负责原始数据到 [0, 1] 区间数值的相互转换工作。针对不同的数据类型对应不同类型的度量。

## 度量功能
度量用于完成以下功能：

1. 将数据转换到 [0, 1] 范围内，方便将数据映射到位置、颜色、大小等图形属性；

2. 将归一化后的数据反转回原始值。例如 `分类a` 转换成 0.2，那么对应 `0.2` 需要反转回 `分类a`；

3. 划分数据，用于在坐标轴、图例显示数值的范围、分类等信息。

Scale 的功能非常简单易理解，但是在 G2 的数据处理流程中起着非常重要的承接作用，通过阅读 [G2 数据处理流程](../tutorial/dataflow.md)章节，可以更好得理解度量 Scale。

## 度量分类
度量的类型是由原始数据的值类型所决定的，所以在介绍度量的类型之前，需要了解下 BizCharts 对数据的分类方式。

在 BizCharts 中我们按照数值是否连续对数据进行分类：

1. 分类（非连续）数据，又分为有序分类和无序分类；

2. 连续数据，时间也是一种连续数据类型。

示例:
```jsx
var data = [
  {"month":"一月","temperature":7,"city":"tokyo"},
  {"month":"二月","temperature":6.9,"city":"newYork"},
  {"month":"三月","temperature":9.5,"city":"tokyo"},
  {"month":"四月","temperature":14.5,"city":"tokyo"},
  {"month":"五月","temperature":18.2,"city":"berlin"}
]
var scale = {
  month: {
    alias: '月份' // 为属性定义别名
  },
  temperature: {
    alias: '温度' // 为属性定义别名
  }
};
<Chart scale={scale}/>
```

在上述数据中，`month` 代表月份，`temperature` 代表温度，`city` 代表城市，其中 `month` 和 `city` 都是分类类型数据，但是不同的是 `month` 作为月份是有序的分类类型，而 `city` 是无序的分类类型，而 `temperature` 是连续的数值类型。

根据上述的数据分类方式，提供了不同的度量类型：

| 数据类型 | 度量类型 |
| ------- | ------- |
| 连续 | linear、log、pow、time |
| 分类（非连续） | cat、timeCat |

另外还提供了 `identity` 类型的度量用于数据源中 **常量** 数据的操作。

对于 BizCharts 生成的所有度量对象，均拥有以下属性，这些属性均可以由用户进行配置。

```js
{
  type: string, // 度量的类型
  range: array, // 数值范围区间，即度量转换的范围，默认为 [0, 1]
  alias: string, // 为数据属性定义别名，用于图例、坐标轴、tooltip 的个性化显示
  ticks: array, // 存储坐标轴上的刻度点文本信息
  tickCount: number, // 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值
  formatter: function, // 回调函数，用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴、图例、tooltip 上的显示
}
```

## 度量类型
根据数据的类型，支持以下几种度量类型：

| 类型 | 说明 |
| :- | :- |
| identity | 常量类型的数值，也就是说数据的某个字段是不变的常量。|
| [linear](#linear) | 连续的数字 [1,2,3,4,5]。|
| [cat](#cat) | 分类, ['男','女']。|
| [time](#time) | 连续的时间类型。|
| [timeCat](#timeCat) | 非连续的时间，比如股票的时间不包括周末或者未开盘的日期。|
| [log](#log) | 连续非线性的 Log 数据 将 [1,10,100,1000] 转换成[0,1,2,3]。|
| [pow](#pow) | 连续非线性的 pow 数据 将 [2,4,8,16,32] 转换成 [1,2,3,4,5]。|

<span id="linear"> </span>

### linear
连续的数据值，如这一组数据：[1, 2, 3, 4, 5]，除了通用的属性外，还包含以下自有属性：
```jsx
{
  nice: boolean, // 默认为 true，用于优化数值范围，使绘制的坐标轴刻度线均匀分布。例如原始数据的范围为 [3, 97]，如果 nice 为 true，那么就会将数值范围调整为 [0, 100]
  min: number, // 定义数值范围的最小值
  max: number, // 定义数值范围的最大值
  tickCount: number, // 定义坐标轴刻度线的条数，默认为 5
  tickInterval: number, // 用于指定坐标轴各个刻度点的间距，为原始数据值的差值，tickCount 和 tickInterval 不可以同时声明
}
```

| 属性名 | 说明 | 值类型 |
| :- | :- |
| type | scale类型。 | String |
| formatter | 用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴 axis、图例 legend、tooltip 上的显示。| Function |
| range | 输出数据的范围，默认[0, 1]，格式为 [min, max]，min 和 max 均为 0 至 1 范围的数据。| Array |
| alias | 该数据字段的显示别名，一般用于将字段的英文名称转换成中文名。| String |
| tickCount | 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值。| Number |
| ticks | 用于指定坐标轴上刻度点的文本信息，当用户设置了 ticks 就会按照 ticks 的个数和文本来显示。| Array |
| nice | 默认为 true，用于优化数值范围，使绘制的坐标轴刻度线均匀分布。例如原始数据的范围为 [3, 97]，如果 nice 为 true，那么就会将数值范围调整为 [0, 100]| Boolean |
| min | 定义数值范围的最小值 | Number/Float |
| max | 定义数值范围的最大值 | Number/Float |
| tickInterval | 用于指定坐标轴各个标度点的间距，是原始数据之间的间距差值，tickCount 和 tickInterval 不可以同时声明。 | Number/Float |

<span id="cat"></span>

### cat

分类类型数据的度量。除了拥有通用的度量属性外，用户还可以设置 `values` 属性：

```jsx
{
  values: array, // 指定当前字段的分类值
}
```

BizCharts 在生成 cat 类型的度量时，`values` 属性的值一般都会从原始数据源中直接获取，但对于下面两种场景，需要用户手动指定 values 值：

1. 需要指定分类的顺序时，例如：type 字段原始值为 ['最大', '最小', '适中']，我们想指定这些分类在坐标轴或者图例上的顺序为 ['最小','适中','最大']。这时候 cat 度量的配置如下：

```jsx
var data  = [
  {a: 'a1', b:'b1', type: '最小'},
  {a: 'a2', b:'b2', type: '最大'},
  {a: 'a3', b:'b3', type: '适中'}
];
var scale = {
  type:{
    type:'cat',
	values: ['最小', '适中', '最大']
  }
}
<Chart scale={scale}/>
```

如果不声明度量的values字段，那么默认的顺序是：‘最小’，‘最大’，‘适中’。

2. 如果数据中的分类类型使用枚举的方式表示，那么也需要指定 values。

Example:

```jsx
var data  = [
  {a: 'a1', b:'b1', type: '最小'},
  {a: 'a2', b:'b2', type: '最大'},
  {a: 'a3', b:'b3', type: '适中'}
];
var scale = {
  type:{
    type:'cat',
	values: ['最小', '适中', '最大']
  }
}
<Chart scale={scale}/>
```

** 此处必须指定 'cat' 类型，values 的值必须按照索引跟枚举类型一一对应。**

| 属性名 | 说明 | 值类型 |
| :- | :- |
| type | scale类型。 | String |
| formatter | 用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴 axis、图例 legend、tooltip 上的显示。| Function |
| range | 输出数据的范围，默认[0, 1]，格式为 [min, max]，min 和 max 均为 0 至 1 范围的数据。| Array |
| alias | 该数据字段的显示别名，一般用于将字段的英文名称转换成中文名。| String |
| tickCount | 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值。| Number |
| ticks | 用于指定坐标轴上刻度点的文本信息，当用户设置了 ticks 就会按照 ticks 的个数和文本来显示。| Array |
| values | 具体的分类的值，一般用于指定具体的顺序和枚举的对应关系 | Array |

<span id="log"> </span>

### log

连续非线性的 log 类型度量，该度量会将 [1, 10, 100, 1000] 先转换成 [0, 1, 2, 3] 然后再进行归一化操作。log 类型的数据可以将非常大范围的数据映射到一个均匀的范围内。

log 度量是 linear 的子类，支持所有通用的属性和 linear 度量的属性，特有的属性如下：

```jsx
{
  base: number, // log 的基数，默认是 2
}
```

| 属性名 | 说明 | 值类型 |
| :- | :- |
| type | scale类型。 | String |
| formatter | 用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴 axis、图例 legend、tooltip 上的显示。| Function |
| range | 输出数据的范围，默认[0, 1]，格式为 [min, max]，min 和 max 均为 0 至 1 范围的数据。| Array |
| alias | 该数据字段的显示别名，一般用于将字段的英文名称转换成中文名。| String |
| tickCount | 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值。| Number |
| ticks | 用于指定坐标轴上刻度点的文本信息，当用户设置了 ticks 就会按照 ticks 的个数和文本来显示。| Array |
| nice | 是否将 ticks 进行优化，变更数据的最小值、最大值，使得每个 tick 都是用户易于理解的数据 | Boolean |
| min | 定义数值范围的最小值 | Number/Float |
| max | 定义数值范围的最大值 | Number/Float |
| tickInterval | 用于指定坐标轴各个标度点的间距，是原始数据之间的间距差值，tickCount 和 tickInterval 不可以同时声明。 | Number/Float |
| base | Log 的基数，默认是2 | Number/Float |

#### log 度量的使用场景

对于以下场景，建议将数据的度量类型指定为 log 类型：

1. 散点图中数据的分布非常广，同时数据分散在几个区间内是，例如分布在 0 - 100， 10000 - 100000，1千万 - 1亿内，这时候适合使用 log 度量；
2. 热力图中数据分布不均匀时也会出现只有非常高的数据点附近才有颜色，此时需要使用 log 度量，对数据进行 log 处理。

<span id="pow"> </span>

### pow

连续非线性的 pow 类型度量，该度量将 [2, 4, 8, 16, 32] 先转换成 [1, 2, 3, 4, 5] 然后再进行归一化操作。

pow 类型的度量也是 linear 类型的一个子类，除了支持所有通用的属性和 linear 度量的属性外也有自己的属性：

```jsx
{
  exponent: number, // 指数，默认是 2
}
```
| 属性名 | 说明 | 值类型 |
| :- | :- |
| type | scale类型。 | String |
| formatter | 用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴 axis、图例 legend、tooltip 上的显示。| Function |
| range | 输出数据的范围，默认[0, 1]，格式为 [min, max]，min 和 max 均为 0 至 1 范围的数据。| Array |
| alias | 该数据字段的显示别名，一般用于将字段的英文名称转换成中文名。| String |
| tickCount | 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值。| Number |
| ticks | 用于指定坐标轴上刻度点的文本信息，当用户设置了 ticks 就会按照 ticks 的个数和文本来显示。| Array |
| nice | 是否将 ticks 进行优化，变更数据的最小值、最大值，使得每个 tick 都是用户易于理解的数据。 | Boolean |
| min | 定义数值范围的最小值 | Number/Float |
| max | 定义数值范围的最大值 | Number/Float |
| tickInterval | 用于指定坐标轴各个标度点的间距，是原始数据之间的间距差值，tickCount 和 tickInterval 不可以同时声明。 | Number/Float |
| exponent | 指数，默认 2 | Number |


<span id="time"> </span>

### time
是 linear 度量的一种，连续的时间度量类型，默认会对数据做排序。
| 属性名 | 说明 |
| :- | :- |
|nice |	是否将 ticks 进行优化，变更数据的最小值、最大值，使得每个 tick 都是用户易于理解的数据|
|min|	最小值|
|max	|最大值|
|mask	|数据的格式化格式 默认：'yyyy-mm-dd',|
|tickCount|	坐标点的个数，默认是 5，但不一定是准确值。|
|tickInterval|	用于指定坐标轴各个标度点的间距，是原始数据之间的间距差值，time 类型需要转换成时间戳，tickCount 和 tickInterval 不可以同时声明。|
|alias|	别名|
|range	|输出数据的范围，默认[0, 1]，格式为 [min, max]，min 和 max 均为 0 至 1 范围的数据。|
|formatter	|回调函数，用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴 axis、图例 legend、tooltip 上的显示。|
|ticks|	用于指定坐标轴上刻度点的文本信息，当用户设置了 ticks 就会按照 ticks 的个数和文本来显示。|
> 说明：mask 的占位符标准同 [moment](https://momentjs.com/docs/#/displaying/format/);

目前 G2 会自动识别如下形式的时间格式，当用户需要生成 time 类型的度量时，建议将原始时间数据转换为如下形式：

1. 时间戳，如 1436237115500；
2. 时间字符串： '2015-03-01'，'2015-03-01 12:01:40'，'2015/01/05'，'2015-03-01T16:00:00.000Z'。

还有一种离散事件类型，参见[timeCat](#timeCat)

| 属性名 | 说明 | 值类型 |
| :- | :- |
| type | scale类型。 | String |
| formatter | 用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴 axis、图例 legend、tooltip 上的显示。| Function |
| range | 输出数据的范围，默认[0, 1]，格式为 [min, max]，min 和 max 均为 0 至 1 范围的数据。| Array |
| alias | 该数据字段的显示别名，一般用于将字段的英文名称转换成中文名。| String |
| tickCount | 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值。| Number |
| ticks | 用于指定坐标轴上刻度点的文本信息，当用户设置了 ticks 就会按照 ticks 的个数和文本来显示。| Array |
| nice | 是否将 ticks 进行优化，变更数据的最小值、最大值，使得每个 tick 都是用户易于理解的数据。 | Boolean |
| min | 定义数值范围的最小值 | Number/Float |
| max | 定义数值范围的最大值 | Number/Float |
| tickInterval | 用于指定坐标轴各个标度点的间距，是原始数据之间的间距差值，time 类型需要转换成时间戳，tickCount 和 tickInterval 不可以同时声明。 | Time |
| mask | 数据的格式化格式 默认：'yyyy-mm-dd' | String |


<span id="timeCat"> </span>

### timeCat

timeCat 度量对应时间数据，但是不是连续的时间类型，而是有序的分类数据。例如股票交易的日期，此时如果使用 time 类型，那么由于节假日没有数据，折线图、k 线图就会发生断裂，所以此时需要使用 timeCat 类型度量将日期转换为有序的分类数据，该度量默认会对数据做排序。

timeCat 是 cat 度量的子类，除了支持所有通用的属性和 [cat](#cat) 度量的属性外也有自己的属性:

```jsx
{
  mask: string, // 指定时间的显示格式，默认：'YYYY-MM-DD'
}
```

| 属性名 | 说明 | 值类型 |
| :- | :- |
| type | scale类型。 | String |
| formatter | 用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴 axis、图例 legend、tooltip 上的显示。| Function |
| range | 输出数据的范围，默认[0, 1]，格式为 [min, max]，min 和 max 均为 0 至 1 范围的数据。| Array |
| alias | 该数据字段的显示别名，一般用于将字段的英文名称转换成中文名。| String |
| tickCount | 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值。| Number |
| ticks | 用于指定坐标轴上刻度点的文本信息，当用户设置了 ticks 就会按照 ticks 的个数和文本来显示。| Array |
| nice | 是否将 ticks 进行优化，变更数据的最小值、最大值，使得每个 tick 都是用户易于理解的数据。 | Boolean |
| mask | 数据的格式化格式 默认：'yyyy-mm-dd' | String |


