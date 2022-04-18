### 定义

**CSS**：层叠样式表（**C**ascading **S**tyle **S**heet）是为网页添加样式的代码，甚至不是标记语言，它是一门样式表语言

CSS 布局主要就是基于盒模型的。每个占据页面空间的块都有这样的属性：

- `padding`：即内边距，围绕着内容（比如段落）的空间。
- `border`：即边框，紧接着内边距的线。
- `margin`：即外边距，围绕元素外部的空间，(上右下左)若无左则与右同

![three boxes sat inside one another. From outside to in they are labelled margin, border and padding](https://mdn.mozillademos.org/files/9443/box-model.png)



### 组成

+ **选择器**：Selector
+ **声明**：Declaration
+ **属性**：Properties
+ **属性的值**：Property value

![图解CSS声明](https://mdn.mozillademos.org/files/16483/css-declaration.png)

### 根据位置确定样式

+ 空格：如 **li em:{}** 将应用于 **\<li\>** 里的 **\<em\>** 元素上
+ **+**：相邻选择符，如 **\<h1+p\>** ,将应用紧接于**\<h1>** 之后的 **\<p>**元素上
+ **#**：Id选择器
+ *****：通配选择器

### 优先级

+ 相同优先级的规则，顺序在后的生效
+ 元素选择器不具体的优先级低，元素选择器具体的(如class 类选择器)优先级高
+ 最高优先级：**!important**，强制该属性应用，如（**border: none   !important;**）

### 继承

+ 一些设置在父元素上的CSS属性会被子元素继承如（body{color:}, span{color:}），有些则不能(width:50%)
+ 控制继承: 存在四个特殊的通用属性值，所有CSS属性都接收这些值
  + inherit：使子元素和父元素属性相同，“开启继承”
  + initial：设置属性值与默认浏览器相同
  + unset：将属性重置为自然值，也就是如果属性是自然继承那么就是 `inherit`，否则和 `initial` 一样
  + revert：很少浏览器支持，略

## 选择器

+ 根据一个元素上的某个标签的属性是否存在：如 **a[title] { }**
+ 根据一个有特定值的标签属性是否存在：如 **a[href="https://example.com"] { }**，可匹配多个值**p[class~="special special1 special2"]**

| 选择器          | 示例                            | 描述                                                         |
| :-------------- | :------------------------------ | :----------------------------------------------------------- |
| `[attr]`        | `a[title]`                      | 匹配带有一个名为 *attr* 的属性的元素 —— 方括号里的值。       |
| `[attr=value]`  | `a[href="https://example.com"]` | 匹配带有一个名为 *attr* 的属性的元素，其值**正为** *value*—— 引号中的字符串。 |
| `[attr~=value]` | `p[class~="special"]`           | 匹配带有一个名为 *attr* 的属性的元素 ，其值正为 *value*，或者匹配带有一个 *attr* 属性的元素，其值有一个或者更多，至少有一个和 *value* 匹配。注意，在一列中的好几个值，是用空格隔开的。 |
| `[attr|=value]` | `div[lang|="zh"]`               | 匹配带有一个名为 *attr* 的属性的元素，其值可正为 *value*，或者开始为 *value*，后面紧随着一个连字符。 |

+ 伪类：样式化一个元素的特定状态，如 **a:hover { }** 在指针悬停时选择，**p:first-child{ }**选择第一个p元素;*artical p:first-child{}*，选择artical下的第一个p元素
+ 伪元素：选择一个元素的某个部分而不是元素本身，如 `::first-line`是会选择一个元素中的第一行
  + 可以组合使用`p:first-child::first-line {}`选择第一段的第一行
  + content：与`::before`,`::after`相配合，可以插入内容，并进行样式编辑

```css
.box::before {
    content: "";
    display: block;
    width: 100px;
    height: 100px;
    background-color: rebeccapurple;
    border: 1px solid black;
}
```

+ 运算符：**article > p { }** ，选择了 `<article>` 元素的初代子元素
+ **~**：通用兄弟选择器`h1 ~p`, 适配h1下一级中所有的p兄弟元素
+ *****：全局选择器可以是代码更易读
  + `article :first-child{}`代表\<article>元素选择器下的一个兄弟选择器
  + `article:first-child{}`代表 作为其他元素的第一子元素\<article>
  + `article *:first-child{}`代表\<article>元素选择器下的**任何**第一子元素

+ **.**：类选择器，
  + 可以添加前缀以达到特定的选择特定元素的功能，如`span.special`, `p.special`
  + 在元素有多个类时，可以通过多类组合来筛选出需要的对象，如`.notebox.warning`
+ **#**：Id选择器，可以组合筛选，如`h1#heading`;也可单独使用，如`#One`
+ 子字符串匹配选择器：

| 选择器          | 示例                | 描述                                                         |
| :-------------- | :------------------ | :----------------------------------------------------------- |
| `[attr^=value]` | `li[class^="box-"]` | 匹配带有一个名为 *attr* 的属性的元素，其值开头为 *value* 子字符串。 |
| `[attr$=value]` | `li[class$="-box"]` | 匹配带有一个名为 *attr* 的属性的元素，其值结尾为 *value* 子字符串 |
| `[attr*=value]` | `li[class*="box"]`  | 匹配带有一个名为 *attr* 的属性的元素，其值的字符串中的任何地方，至少出现了一次 *value* 子字符串。 |

+ **i**：让选择器变得大小写不敏感，如**li[class^="a" i] {}**

### 盒模型

+ block:块级，会占用一行，且会换行
+ inline：内联级，互相之间不会换行
+ flex/inline-flex:弹性布局
+ inline-block：折中方案，是块级，会推开其他元素，同时也可在同一行显示

#### 盒模型样式

+ 圆角边框：border-radius: 10px，为以角为起点10px为长度的所形成的圆角，  border-top-right-radius: 1em 10%;右上角水平半径为1em，垂直半径为10%
+ 几种颜色显示方式：
  + rgba(0,0,0,.5);
  + #000;
  + black;

+ 文本排版方向，writting-mode:
  + horizontal-tb:从上至下，文本为横向（默认文本排版方式）

#### 盒模型尺寸控制

+ overflow：
  + 默认值为visible,可见的
  + hidden：隐藏超出限制大小的部分
  + scroll：添加滚动条，overflow-y：仅显示Y轴方向滚动条；overflow-x：仅显示X轴方向滚动条
  + overflow: scroll hidden，（x,y）
  + auto：在溢出时显示滚动条

#### CSS的值与单位

+ ##### 绝对长度单位

  以下都是**绝对**长度单位——它们与其他任何东西都没有关系，通常被认为总是相同的大小。

  | 单位 | 名称         | 等价换算            |
  | :--- | :----------- | :------------------ |
  | `cm` | 厘米         | 1cm = 96px/2.54     |
  | `mm` | 毫米         | 1mm = 1/10th of 1cm |
  | `Q`  | 四分之一毫米 | 1Q = 1/40th of 1cm  |
  | `in` | 英寸         | 1in = 2.54cm = 96px |
  | `pc` | 十二点活字   | 1pc = 1/16th of 1in |
  | `pt` | 点           | 1pt = 1/72th of 1in |
  | `px` | 像素         | 1px = 1/96th of 1in |

+ ##### 相对长度单位

  相对长度单位相对于其他一些东西，比如父元素的字体大小，或者视图端口的大小。使用相对单位的好处是，经过一些仔细的规划，您可以使文本或其他元素的大小与页面上的其他内容相对应。下表列出了web开发中一些最有用的单位。

  | 单位   | 相对于                                                       |
  | :----- | :----------------------------------------------------------- |
  | `em`   | 在 font-size 中使用是相对于父元素的字体大小，在其他属性中使用是相对于自身的字体大小，如 width |
  | `ex`   | 字符“x”的高度                                                |
  | `ch`   | 数字“0”的宽度                                                |
  | `rem`  | 根元素的字体大小                                             |
  | `lh`   | 元素的line-height                                            |
  | `vw`   | 视窗宽度的1%                                                 |
  | `vh`   | 视窗高度的1%                                                 |
  | `vmin` | 视窗较小尺寸的1%                                             |
  | `vmax` | 视图大尺寸的1%                                               |

+ ##### 透明度单位

    opacity: 0(完全透明)和1(完全不透明)之间的数字。

+ ##### CSS函数

  calc()：例如  width: calc(20% + 100px);  20%是根据父容器.wrapper的宽度来计算的。
  
+ ##### 视口单位

    + vh,vw为相对于当前视窗的高宽比例，如20vh：为当前视窗宽度的20%。

    + 调整图像大小：

      + 对于固定大小的box，如果图像的尺寸大于box的尺寸，首先要设置图片可占用的比例（如height:100%;width:100%；），然后可以设置**object-fit**属性：

        + 为cover时只会显示大小与box尺寸大小相同的图像，多余部分不显示；
        + 为contain时会缩小比例全图显示；

        + 为fill时会拉伸比例填充box。

    + 

    