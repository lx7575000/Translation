## 比例尺

D3的比例尺方法：

- 传入一组数据（通常是数值、时间或类目）
- 然后返回值（可以使坐标、颜色、长度或半径）

比例尺通常用于将数值转换为对应的视觉属性，例如坐标地址、长度、颜色等。

例如我们有以下数据

```
[ 0, 2, 3, 5, 7.5, 9, 10 ]
```

可以通过创建比例尺方法：

```JavaScript
const myScale = d3.scaleLinear()
	.domain([0, 10])
	.range([0, 600])
```

使用D3创建`myScale`方法，可以传入0到10之间的数值（**domain**，定义域），方法会遍历转换为0到600之间的值域（**range**）。

我们可以使用`myScale`来计算坐标值

```
myScale(0);   // returns 0
myScale(2);   // returns 120
myScale(3);   // returns 180
...
myScale(10);  // returns 600
```
![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/EC39D4E27AF6F426998A8A993E588E35.jpg)

比例尺方法主要可用来将数据转换为视觉元素，例如坐标、长度、颜色等。

例如，我们可以将数据转换为：

  - 数值转换为长度在0到500之间的柱图
  - 数值转换为坐标位置在0到200之间的折线图
  - 使用百分比转换为连续的颜色值
  - 转换为x轴的坐标值


### 构建比例尺

创建线性比例尺：

```
const myScale = d3.scaleLinear()
```

我们可以重新配置比例尺的**作用域**和值域范围。


```
myScale
  .domain([0, 100])
  .range([0, 800]);
```

现在，`myScale`方法允许传入0到100之间的参数，返回线性地返回0到800之间的值。

```js
const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
const scale = d3.scaleLinear()
scale.domain([0, 10])
  .range([0, 500])

d3.select("svg")
  .selectAll('circle')
  .data(data)
  .enter().append('circle')
  .attr('r', 3)
  .attr('cx', d => scale(d))

d3.select('svg')
  .selectAll("text")
  .data(data)
  .enter().append('text')
  .attr('y', -8)
  .attr('x', d => scale(d))
  .text(d => d)
```

**效果图**

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/77ABF4C7DE077DA3AABB50477118CB96.jpg)

### 连续值（输入输出）的比例尺

本小结，会详细介绍连续输入、输出值的比例尺类型

#### 线性比例尺（scaleLinear）

线性比例尺是最常见的比例尺，它可以用来将数值转换为坐标位置和长度属性。因此它是最必学的比例尺：

它使用线性函数（`y = m * x + b`）在定义域和值域间进行插值。

```JS
const linearScale = d3.scaleLinear()
        .domain([0, 10])
        .range(['yellow', 'red'])

linearScale(0);   // returns "rgb(255, 255, 0)"
linearScale(5);   // returns "rgb(255, 128, 0)"
linearScale(10);  // returns "rgb(255, 0, 0)"
```

如上例，线性比例尺可以用来将数值转换为渐变的颜色值，用在需要色彩渐变效果时是非常有效的。也可以考虑使用`scaleQuantize`, `scaleQuantile` 和 `scaleThreshold`等方法。

#### scalePow

`scalePow`依据`(y = m * x ^ k + b)`方法来传入作用域间的数据进行计算转换为值域数据。其中k值通过`.exponent()`方法设定。

```js
const powerScale = d3.scalePow()
        .exponent(0.5)
        .domain([0, 100])
        .range([0, 30])

powerScale(0);   // returns 0
powerScale(50);  // returns 21.21...
powerScale(100); // returns 30
```

#### scaleSqrt

`scaleSqrt`方法可视作`scalePow`方法的一个实例（k值设定为0.5）。通常被用来将圆的面积进行数值转换（不是半径！）。当我们使用圆的尺寸表示数据时，通常使用圆的面积而不是半径来表示数值大小会更恰当。

```js
const sqrtScale = d3.scaleSqrt()
  .domain([0, 100])
  .range([0, 30]);

sqrtScale(0);   // returns 0
sqrtScale(50);  // returns 21.21...
sqrtScale(100); // returns 30
```

```js
const data = [0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100]

const sqrtScale = d3.scaleSqrt()
sqrtScale.domain(d3.extent(data))
          .range([0, 30])

const linearScale = d3.scaleLinear()
linearScale.domain(d3.extent(data))
            .range([0, 700])

d3.select('svg')
  .selectAll('circle')
  .data(data)
  .enter().append('circle')
  .attr('r', d => sqrtScale(d))
  .attr('cx', d => linearScale(d))

```

**效果图**

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/76B8DDF6B72ECDBE35D5E90BB815044E.jpg)

#### 对数比例尺（scaleLog）

对数比例尺通过对数函数`(y = m * log(x) + b)`进行插值，当数据具有指数性质时，使用它非常有效。

```js
const logScale = d3.scaleLog()
logScale.domain([10, 100000])
  .range([0, 600])

logScale(10);     // returns 0
logScale(100);    // returns 150
logScale(1000);   // returns 300
logScale(100000); // returns 600
```

```js
const data = [10, 100, 1000, 10000, 100000]
const logScale = d3.scaleLog()
logScale.domain([10, 100000])
  .range([0, 600])

d3.select('svg')
  .selectAll('text')
  .data(data)
  .enter().append('text')
  .attr('x', d => logScale(d))
  .text(d => d)
```

**效果图**

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/88115E5C690B5E7E771A6B8E117D6D32.jpg)


#### scaleTime

`scaleTime`比例尺与线性比例尺`scaleLinear`相似，不同点在于scaleTime的作用域要求传入一组时间值（当绘制以时间为x轴的折线图时，该方法相当好用）。

```js
const timeScale = d3.scaleTime()
timeScale
  .domain([new Date(2016, 0, 1), new Date(2017, 0, 1)])
  .range([0, 700])

timeScale(new Date(2016, 0, 1));   // returns 0
timeScale(new Date(2016, 6, 1));   // returns 348.00...
timeScale(new Date(2017, 0, 1));   // returns 700
```

```js
const data = [new Date(2016, 0, 1), new Date(2016, 3, 1), new Date(2016, 6, 1), new Date(2017, 0, 1)]
const timeScale = d3.scaleTime()
  .domain([new Date(2016, 0, 1), new Date(2017, 0, 1)])
  .range([0, 700])

d3.select('svg')
	.selectAll('circle')
	.data(data)
	.enter()
	.append('circle')
	.attr('r', 2)
	.attr('cy', 8)
	.attr('cx', function(d) {
		return timeScale(d);
	});

d3.select('svg')
	.selectAll('text')
	.data(data)
	.enter()
	.append('text')
	.attr('x', function(d) {
		return timeScale(d);
	})
	.text(function(d) {
		return d.toDateString();
	});
```

**效果图**

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/8BA5AA3083122EF3600EEE863BA2436F.jpg =814x60)

#### 顺序比例尺(scaleSwquential)

顺序比例尺用于将连续值映射到预设（或自定义）的插值器再由其转换为值域范围内的值输出。（插值器是一个方法，接收0到1之间的值作为传入参数，然后输出两数值之间内的插值数字、颜色、字符串等）

D3提供了许多预设的插值器方法，包括许多颜色类型的。例如我们可以使用`d3.interpolateRainbow`方法创建彩虹色比例尺：

```js
const sequentialScale = d3.scaleSequential()
  .domain([0, 100])
  .interpolator(d3.interpolateRainbow)

sequentialScale(0);   // returns 'rgb(110, 64, 170)'
sequentialScale(50);  // returns 'rgb(175, 240, 91)'
sequentialScale(100); // returns 'rgb(110, 64, 170)'
```

```js
const linearScale = d3.scaleLinear()
  .domain([0, 100])
  .range([0, 600])

const sequentialScale = d3.scaleSequential()
  .domain([0, 100])
  .interpolator(d3.interpolateRainbow)

const myData = [0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100]

d3.select('svg')
  .selectAll('circle')
  .data(myData)
  .enter().append('circle')
  .attr('r', 10)
  .attr('cx', d => linearScale(d))
  .style('fill', d => sequentialScale(d))
```

**效果图**

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/A3FD7BC43B87EE5CA1273F14486736FA.jpg =660x74)

切记，插值器会决定输出范围，因此你不需要指定值域范围。下面我们会展示几个颜色插值器：

```js
const linearScale = d3.scaleLinear()
	.domain([0, 100])
	.range([0, 600]);

const sequentialScale = d3.scaleSequential()
	.domain([0, 100]);

const interpolators = [
	'interpolateViridis',
	'interpolateInferno',
	'interpolateMagma',
	'interpolatePlasma',
	'interpolateWarm',
	'interpolateCool',
	'interpolateRainbow',
	'interpolateCubehelixDefault'
]

const data = d3.range(0, 100, 2) // 生成数组，0到100直接，数值间隔为2

function dots(d) {
  sequentialScale
    .interpolator(d3[d])

  d3.select(this)
    .append('text')
    .attr('y', -12)
    .text(d)

  d3.select(this)
    .selectAll('rect')
    .data(data)
    .enter().append('rect')
    .attr('x', d => linearScale(d))
    .attr('width', 11)
    .attr('height', 30)
    .style('fill', d => sequentialScale(d))
}

d3.select('svg')
  .selectAll('g.interpolator')
  .data(interpolators)
  .enter().append('g')
  .classed('interpolator', true)
  .attr('transform', (d, i) => `translate(0, ${i * 70})`)
  .each(dots)

```

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/B94D1E47287872078676C8004E533C6A.jpg =659x582)


### 区间

`scaleLinear`、`scalePow`、`scaleSqrt`、`scaleLog`、`scaleTime`和`scaleSequential`允许传入超出值域范围的数据。

```js
const linearScale = d3.scaleLinear()
linearScale.domain([0, 10])
  .range([0, 100])

linearScale(20);  // returns 200
linearScale(-10); // returns -100
```

如上例我们向比例尺中传入了超出定义域范围的值，如果我们需要严格限制传入作用域的值，我们可以使用`clamp`方法来限制输出值域。

```
linearScale.clamp(true)

linearScale(20);  // returns 100
linearScale(-10); // returns 0
```

当然也可以使用`.clamp(false)`关闭输出范围限制。


### Nice

如果定义域范围时由真实数据计算得到的（例如使用d3.extent方法），那么这个时候得到的定义域起始点会很生硬（0.12 ~ 38.82这样让强迫症看起来很不爽的范围）。尽快对于一些粗线条的人来说这不是什么大问题，但如果这些值用在制作坐标轴估计你就会坐不住了~

```js
const data = [0.243, 0.584, 0.987, 0.153, 0.433];
const extent = d3.extent(data);

const linearScale = d3.scaleLinear()
  .domain(extent)
  .range([0, 100]);

const axis = d3.axisBottom(linearScale)

d3.select('svg')
  .append('g.axis')
  .call(axis)
```
**效果图**

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/90C7A7FEB981E17F299A9DEE3F1D21AC.jpg =627x63)

想你之所想，急你之所急。D3提供了`.nice()`方法会帮你把定义域数值范围弄的更平滑好看些。_切记， 每次更新domain时候，都要重新调用`nice`方法_

```
linearScale.nice()
```
**效果图**

![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/00AC53CCB0E60DAE78601D008A330DD2.jpg =680x85)

#### 多区间（Multiple segments）
`scaleLinear`,`scalePow`,`scaleSqrt`, `scaleLog`和`scaleTime`的作用域（domain）通常是由两个值（起始、终值）组成的。但现实应用场景中，我们会有用到需要传入多个值将作用域分为多个区间范围的需求。

```js
const linearScale = d3.scaleLinear()
  .domain([-10, 0, 10])
  .range(['red', '#ddd', 'blue']);

linearScale(-10);  // returns "rgb(255, 0, 0)"
linearScale(0);    // returns "rgb(221, 221, 221)"
linearScale(5);    // returns "rgb(128, 128, 255)"

```

```js
const xScale = d3.scaleLinear()
  .domain([-10 , 10])
  .range([0, 600])

const linearScale = d3.scaleLinear()
  .domain([-10, 0, 10])
  .range(['red', '#ddd', 'blue'])

const myData = [-10, -8, -6, -4, -2, 0, 2, 4, 6, 8, 10]

d3.select('svg')
  .append('g')
  .attr("transform", 'translate(40, 40)')
  .selectAll('circle')
  .data(myData)
  .enter().append('circle')
  .attr('r', 10)
  .attr('cx', d => xScale(d))
  .style('fill', d => linearScale(d))
```
**效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/958F68E12E5B5EEDE95DB98FEBAB4257.jpg =661x59)

通常我们会使用多个段来区分正负值的情况（上面的例子），我们也可以使用更多的分块，只要作用域和值域具有相同的**区间个数**。

#### 反向（Inversion）

`.invert()`方法允许我们定义比例尺的输入、输出值（定义域要求为数值类型）,然后使用`invert`方法传入数值将定义域、值域转换。

```js
const linearScale = d3.scaleLinear()
  .domain([0, 10])
  .range([0, 100]);

linearScale.invert(50);   // returns 5
linearScale.invert(100);  // returns 10
```

### 非连续比例尺

#### scaleQuantize

`scaleQuantize`接收连续定义域区间中的数值作为参数，返回不连续的离散量值。

```js
const quantizeScale = d3.scaleQuantize()
  .domain([0, 100])
  .range(['lightblue', 'orange', 'lightgreen', 'pink'])

quantizeScale(10);   // returns 'lightblue'
quantizeScale(30);  // returns 'orange'
quantizeScale(90);  // returns 'pink'
```

如上例，值域中每个值映射到定义域中一个等大小的块值。

u作为传入给比例尺的参数值：

  - 0 ≤ u < 25 映射给 ‘lightblue’
  - 25 ≤ u < 50 映射给 ‘orange’
  - 50 ≤ u < 75 映射给 ‘lightgreen’
  - 75 ≤ u < 100 映射给 ‘pink’

```js
const quantizeScale = d3.scaleQuantize()
  .domain([0, 100])
  .range(['lightblue', 'orange', 'lightgreen', 'pink'])

const linearScale = d3.scaleLinear()
  .domain([0, 100])
  .range([0, 600])

const myData = d3.range(0, 100, 1)
d3.select('svg')
  .selectAll('rect')
  .data(myData)
  .enter().append('rect')
  .attr('x', d => linearScale(d))
  .attr('height', 30)
  .attr('width', 5)
  .style('fill', d => quantizeScale(d))
```

**效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/D2D06F62089CA3DD9EBB213C9A6E9141.jpg =629x49)


#### scaleQuantile(分位比例尺)

`scaleQunatile`将连续数值映射为离散的值，定义域为**一组**数字。

```js
const myData = [0, 5, 7, 10, 20, 30, 35, 40, 60, 62, 65, 70, 80, 90, 100]

const scaleQuantile = d3.scaleQuantile()
  .domain(myData)
  .range(['lightblue', 'orange', 'lightgreen'])

quantileScale(0);   // returns 'lightblue'
quantileScale(20);  // returns 'lightblue'
quantileScale(30);  // returns 'orange'
quantileScale(65);  // returns 'lightgreen'
```

**🌰**

```js

const scaleQuantile = d3.scaleQuantile()
  .domain(myData)
  .range(['lightblue', 'orange', 'lightgreen'])

const linearScale = d3.linearScale()
  .domain([0 , 100])
  .range([0, 600])

const myData = [0, 5, 7, 10, 20, 30, 35, 40, 60, 62, 65, 70, 80, 90, 100]

d3.select('svg')
  .selectAll('circle')
  .data(myData)
  .enter().append('circle')
  .attr('r', 5)
  .attr('cx', d => linearScale(d))
  .style('fill', d => quantileScale(d))
```

**效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/AD49AFB571F27382B24FF02264127D11.jpg =653x53)

以上例子中定义域（排序过的）数组中的值被分为n块等大小的组（各分割点间数值不一定相同，但相隔两分割点内的定义域元素个数均相同），组内，其中n大小为值域中离散值的个数。

因此🌰中的定义域被分为了三个区域：
  - 首五项被映射为‘lightblue’
  - 中间五项被映射为‘orange’
  - 最后五项被映射为‘lightgreen’

区间分割点可以通过`.quantiles()`得到：

```js
quantileScale.quantiles();  // returns [26.66..., 63]
```

如果值域存在四个值，`quantileScale`会计算定义域的**quantiles**。换句话说，前四分之一会被映射到`range[0]`，接下来的三个四分之一会依次映射到`range[1]`, `range[2]`, `range[3]`。

#### scaleThreshold

`scaleThreshold`比例尺会将连续数值映射为值域中的离散值，定义域要求传入一数值数组存在n-1个数字（有小到大排序）作为分割点分割数值，其中n为至于中离散值的个数。

下面例子中我们会在定义域中定义分割点为0， 50， 100，u为输入比例尺参数

  - u < 0 映射为 ‘#ccc’
  - 0 ≤ u < 50 映射为 ‘lightblue’
  - 50 ≤ u < 100 映射为 ‘orange’
  - u ≥ 100 映射为 ‘#ccc’

```js
const thresholdScale = d3.scaleThreshold()
  .domain([0, 50, 100])
  .range(['#ccc', 'lightblue', 'orange', '#ccc']);

thresholdScale(-10);  // returns '#ccc'
thresholdScale(20);   // returns 'lightblue'
thresholdScale(70);   // returns 'orange'
thresholdScale(110);  // returns '#ccc'
```

**🌰**

```js
const thresholdScale = d3.scaleThreshold()
  .domain([0, 50, 100])
  .range(['#ccc', 'lightblue', 'orange', '#ccc']);

const linearScale = d3.linearScale()
  .domain([0 , 100])
  .range([0, 600])

const data = d3.range(-10, 110, 2)

d3.select('svg')
  .selectAll('rect')
  .data(data)
  .enter().append('rect')
  .attr('x', d => linearScale(d))
  .attr('height', 30)
  .attr('width', 9)
  .style('fill', thresholdScale(d))
```

**效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/80AAEE96189C177B6B55AB26F2876BFD.jpg)

### 输入输出均为离散值的比例尺

#### scaleOrdinal

`scaleOrdinal`将定义域中的离散值分别映射到值域中的离散值。定义域中的值要求位能够出现输入值。值域中的值会按顺序对应，如果值域中的值个数少于定义域中的，则会重复从头开始按顺序对应。

```js

const myData = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

const ordinalScale = d3.scaleOrdinal()
  .domain(myData)
  .range(['black', '#ccc', '#ccc']);

ordinalScale('Jan');  // returns 'black';
ordinalScale('Feb');  // returns '#ccc';
ordinalScale('Mar');  // returns '#ccc';
ordinalScale('Apr');  // returns 'black';
```

**栗子**

```js
const myData = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

const linearScale = d3.scaleLinear()
	.domain([0, 11])
	.range([0, 600]);

const ordinalScale = d3.scaleOrdinal()
	.domain(myData)
	.range(['black', '#ccc', '#ccc']);

d3.select('svg')
	.selectAll('text')
	.data(myData)
	.enter()
	.append('text')
	.attr('x', function(d, i) {
		return linearScale(i);
	})
	.text(function(d) {
		return d;
	})
	.style('fill', function(d) {
		return ordinalScale(d);
	});
```

**效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/814C566470B70DD9842153B31DFE3371.jpg =641x36)

如果传入值没有存在于定义域中，比例尺实例会添加（push）到定义域中。

```
ordinalScale('Monday') // return 'black'
```

当然这其实不是个好行为，毕竟我们希望在定义比例尺实例时候就能穷尽所有可能情况。当后面有未定义值出现时，我们更希望做的是容错处理而不是破坏之前的定义，所以可以使用`.unknown()`来容错：

```js
ordinalScale.unknown('Not a month');
ordinalScale('Tuesday'); // returns 'Not a month'
```

**第二个🌰**

D3提供了一些现成的配色方案：

```js
const ordinalScale = d3.scaleOrdinal()
  .domain(myData)
  .range(d3.schemePaired);
```

```js
const ordinalScale = d3.scaleOrdinal()
  .domain(myData)
  .range(d3.schemePaired);

const myData = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

const linearScale = d3.scaleLinear()
	.domain([0, 11])
	.range([0, 600]);

d3.select('svg')
	.selectAll('text')
	.data(myData)
	.enter()
	.append('text')
	.attr('x', function(d, i) {
		return linearScale(i);
	})
	.text(function(d) {
		return d;
	})
	.style('fill', function(d) {
		return ordinalScale(d);
	});
```

**第二个效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/2B72CCCD983037D48DA4BE8C84986679.jpg =652x49)

#### scaleBand

画柱状图📊时，`scaleBand`会帮你确定柱图的宽度和各个柱子间的间距。`scaleBand`的定义域为一个数组，数组中的各个元素对应每个柱子，值域为柱图的最小/大范围。

事实上，`scaleBand`会将值域范围划分为n段（n为定义域数组中元素个数），并计算每个柱子的宽度和起始位置。

```js
const bandScale = d3.scaleBand()
  .domain(['Mon', 'Tue', 'Wed', 'Thu', 'Fri'])
  .range([0, 200]);

bandScale('Mon'); // returns 0
bandScale('Tue'); // returns 40
bandScale('Fri'); // returns 160
```

可以通过`.bandWidth()`方法获取每段区域的宽度值：

```js
bandScale.bandWidth() // return 40
```

有两种方法来设定各段之间的间距：
  - `paddingInner`: 指定各段（两个柱子）之间的间距值（百分比）。
  - `paddingOuter`: 指定各段（两个柱子之间和首/末柱子左/右两边的）间距值（百分比）。

```js
bandScale.paddingInner(0.05);

bandScale.bandWidth();  // returns 38.38...
bandScale('Mon');       // returns 0
bandScale('Tue');       // returns 40.40...
```

**🌰**

```js
const bandScale = d3.scaleBand()
	.domain(['Mon', 'Tue', 'Wed', 'Thu', 'Fri'])
	.range([0, 200])
	.paddingInner(0.05)

const myData = [
	{day : 'Mon', value: 10},
	{day : 'Tue', value: 40},
	{day : 'Wed', value: 30},
	{day : 'Thu', value: 60},
	{day : 'Fri', value: 30}
]

d3.select('svg')
  .selectAll('rect')
	.data(myData)
	.enter()
	.append('rect')
	.attr('x', function(d, i) {
		return bandScale(d.day);
	})
    .attr('y', d => 200 - d.value)
	.attr('height', d =>  d.value)
	.attr('width', bandScale.bandwidth())
```

**效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/58364E51BB965A3B373FF03AE903B378.jpg =256x88)

#### scalePoint

`scalePoint`生成一个比例尺实例，将定义域集合中的离散值按照值域(数组中为一个范围值)中指定的范围等距映射。

```js
const pointScale = d3.scalePoint()
  .domain(['Mon', 'Tue', 'Wed', 'Thu', 'Fri'])
  .range([0, 500]);

pointScale('Mon');  // returns 0
pointScale('Tue');  // returns 125
pointScale('Fri');  // returns 500
```

**例子**

```js
const pointScale = d3.scalePoint()
  .domain(['Mon', 'Tue', 'Wed', 'Thu', 'Fri'])
  .range([0, 500]);

const myData = [
	{day : 'Mon', value: 10},
	{day : 'Tue', value: 40},
	{day : 'Wed', value: 30},
	{day : 'Thu', value: 60},
	{day : 'Fri', value: 30}
];

d3.select('svg')
  .selectAll('circle')
  .data(myData)
  .enter().append('circle')
  .attr('cx', d => pointScale(d))
  .attr('r', 4)
```

**效果图**


![](https://github.com/lx7575000/Translation/blob/master/All%20About%20Scale/resources/8ADC1C01BC6EE4A96AFB7465D1BF70B1.jpg =636x42)

上例中各点的间距可以通过`.step()`方法得到：

```js
pointScale.step() // return 125
```

可以使用`.padding()`方法指定外部填充的比率，例如如果希望两边间距为四分之一， 可以指定间距为0.25。

```js
pointScale.padding(0.25);

pointScale('Mon');  // returns 27.77...
pointScale.step();  // returns 111.11...
```
