## Some Skills in vscode

+ 生成标签直接输标签名回车即可：div --> \<div>\</div>
+ 生成多个相同标签输标签名之后输个 * 后面加上个数：a*3
+ 父子级关系的标签用 > 连接：ui>li\*3, tr\*3>td\*4
+ 兄弟关系的用 + 连接：h1+div\*3+p+ul>li\*3
+ 带有类名或 id 名的：div.header--> \<div class="header">\</div>; div#header--> \<div id="header">\</div>
+ div.number$\*3

```html
<div class="number1"></div>
<div class="number2"></div>
<div class="number3"></div>
```

+ div\*3{number$}

```html
<div>number1</div>
<div>number2</div>
<div>number3</div>
```



+ 标签内部有内容的内容用 {} 括起来：div{helloWorld} --> \<div>helloWorld\</div>

### 释义

**HTML**： Hyper Text Markup Language，超文本标记语言

### 标记符

+ **\<p>**：**p**aragraph 文本段落
+ **\<img>**：**i**magine 图片，**src**为图片地址，**alt**为替换文字属性，是图像的描述内容，用于当图像不能被用户看见时显示

```html
<img src="https://" alt="错误时显示的信息">
```

+ **\<h>**：**h**eading 标题，共有1-6级
+ **\<ul>**：**u**nordered **L**ist 无序列表 以点显示，如此列表
+ **\<ol>**：**o**rder **L**ist 有序列表 会以数字123显示
+ **\<li>**：**l**ist **i**tem 列表项目

```html
<ul>
  <li>技术人员</li>
  <li>思考者</li>
  <li>建造者</li>
</ul>
```

+ **\<a>**：**a**nchor 链接，**href**：超文本引用（ **h**ypertext **ref**erence）

```html
<a href="https://">点击跳转到百度</a>
```

+ **\<>**：

### 特殊标记符

+ **\<strong>**：强调突出的效果，文本加粗

```html
<p>我的猫咪脾气<strong>暴躁</strong></p>
```

### 属性用法**\<style>**

+ **test-index**：文本缩进，可为负数，若为负数则向前突出，形成“悬挂缩进” 的效果
```html
  <style>
  	p {text-indent:36px;}
  </style>
```

+ 当你使用 `var` 时，可以根据需要多次声明相同名称的变量，但是 `let` 不能
