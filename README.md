# Experience

移动端开发总结

## 概要

<!-- 本文档针对移动前端开发，包括 `Hybrid` 里面的web页面，非 `Native` 应用。 -->
本文档针对移动前端开发，包含`Hybrid`里面的Web页面（不包含`Native`应用）

<!-- ## 适用

所有经验适用于：`iOS6.0+`, `Android4.0+` -->
<!-- * [性能优化](#user-content-performance) -->
## 目录

* [兼容性](#user-content-compatibility)
	* [CSS伪类:active](#user-content-active)
	* [清除iOS输入框内阴影](#user-content-input-shadow)
	* [修正iOS输入框禁用文本色](#user-content-input-disabled)
	* [边框圆角致背景溢出](#user-content-background-overflow)
	* [一个失败的圆（圆角）](#user-content-border-radius-percentage)
	* [不要使用伪元素动画](#user-content-pseudo-element-animation)
	* [:checked与兄弟选择符一起使用的bug](#user-content-checked-sibling-bug)
	* [为什么flex布局不生效](#user-content-flex)
	* [为什么小于12px字号不生效](#user-content-12px)
	* [chrome中body使用rem失效](#user-content-chrome-rem-bug)
	* [不要对html设置百分比大小的字号](#user-content-html-percent-font-size)
	*	[单页应用微信签名](#user-content-weixin-sign)
	* [支付宝WebView单页应用问题](#user-content-alipay-web-view)	
* [Html经验](#user-content-experience)
	* [禁止保存或拷贝图像](#user-content-touch-callout)
	* [取消touch高亮](#user-content-tap-highlight-color)
	* [禁止选中内容](#user-content-user-select)
	* [快速回弹滚动](#user-content-overflow-scrolling)
	* [设置添加到主屏幕的Web App标题](#user-content-shortcut-title)
	* [设置添加到主屏幕的Web App图标](#user-content-shortcut-icon)
	* [添加到主屏幕时隐藏地址栏和状态栏（即全屏）](#user-content-hide-bar)
	* [添加到主屏幕时设置系统顶栏颜色](#user-content-status-bar-style)
	* [电话号码识别](#user-content-tel)
	* [邮箱地址识别](#user-content-email)
	* [关闭iOS键盘首字母自动大写](#user-content-autocapitalize)
	* [关闭iOS输入自动修正](#user-content-autocorrect)
	* [禁止文本缩放](#user-content-text-size-adjust)
	* [Img的跨域问题](#user-content-img-block)
	* [IOS Audio标签问题](#user-content-ios-audio-tag)
	* [IOS Audio 调用play无效](#user-content-ios-audio-invalid-play)
* [Vue经验](#user-content-vue-experience)
	* [EventBus当前对象问题](#user-content-vue-event-bus-context)
	* [Vuex使用问题](#user-content-vue-vuex-probrem)
	* [列表渲染问题](#user-content-vue-list-render-probrem)
* [组件划分原则](#user-content-design-component-rule)

<a name="compatibility"></a>
## 兼容性

<a name="active"></a>
### CSS伪类:active

如果你想使用元素的伪类来实现 `按下激活` 状态，那么你需要知道以下问题：

* iOS上的几乎任何浏览器，定义元素的伪类 `:active` 都是无效；
* Android上，`Android Browser` 和 `Chrome` 都支持伪类 `:active` ，其它第三方浏览器有部分不支持；
* 定义了 `:active` 并且当前浏览器环境支持，当手指在滚动或者无意间的划过时，`:active` 状态都会被激活；

> 为了规避上述所有的问题，如果需要 `按下激活` 状态，推荐使用 `js` 新增一个 `className`

<a name="input-shadow"></a>
### 清除输入框内阴影

iOS上的几乎任何浏览器输入框（input, textarea）默认有内部阴影，但无法使用 `box-shadow` 来清除，如果不需要阴影，可以这样关闭：

```css
input,
textarea {
	/* 方法1: 去掉边框 */
	border: 0;

	/* 方法2: 边框色透明 */
	border-color: transparent;

	/* 方法3: 重置输入框默认外观 */
	-webkit-appearance: none;
	appearance: none;
}
```

<a name="input-disabled"></a>
### 修正iOS输入框禁用文本色

在 `iOS` 上，如果将输入框 `disabled`，此时输入框内的文字颜色将比 `color` 所定义的要浅，并且无法通过给输入框的伪类 `:disabled` 定义 `color` 来修正。

想解决这个问题，可以作如下设置，定义输入框的文本填充色：

```css
input:disabled {
	-webkit-text-fill-color: #000;
}
```

需要注意的是，在 `Mac` 上的 `Safari` 也有同样的问题。


<a name="background-overflow"></a>
### 边框圆角致背景溢出

在红米和OPPO等手机某些版本的 `Android Webview` 中，如果一个元素定义了 `border` + `border-radius`，这时如果该元素有背景，那么背景将会溢出圆角之外。

之所以会出现这个问题：其主要原因是因为CSS对背景裁剪（background-clip）有不同的处理方式，通常它可以是 `border-box | padding-box | content-box` 这3种方式。

浏览器的默认裁减方式是 `border-box`，即溢出 `border` 之外的背景都将被裁减。

对于上述无法裁减边框之外背景的手机，将值定义为 `padding-box | content-box` 都能fix这问题，不过更推荐使用 `padding-box`。因为使用 `content-box`，如果定义了 `padding` 不为 `0`，背景将无法铺满元素。


<a name="border-radius-percentage"></a>
### 一个失败的圆（圆角）

在移动平台上开发时，用CSS画一个圆很简单，只需要一句代码：

```css
.circle {
	border-radius: 50%;
}
```
不过，在 `Android Browser2.*` 上，这个定义将会失效，而显示为默认的矩形。

因为 `Android Browser2.*` 不支持以 `百分比` 作为 `border-radius` 的值，所以如果你需要兼容 `Android Browser2.*`，那么你可以这样：

```css
.circle {
	width: 10rem;
	height: 10rem;
	border-radius: 5rem;
}
```
如果你觉得这样定义不够灵活，想懒一点，那么其实可以给 `border-radius` 预设一个比较大的值，比如 `100rem`，用以避免当元素的尺寸变了，圆角半径也得跟着变，除非元素的尺寸超出了你预设的阀值。


<a name="pseudo-element-animation"></a>
### 不要使用伪元素动画

有的时候你可能会为了减少页面上的DOM数，而使用伪元素。但如果你想给伪元素增加 `animation` 或者 `transition` 动画，这时候会碰上支持性问题。

如果你的项目需要支持以下系统版本，那么建议直接使用真实元素：

* `iOS Safari6.1及以下`
* `Android Browser4.1.*及以下`

<a name="checked-sibling-bug"></a>
### :checked与兄弟选择符一起使用的bug

在 `Android Browser4.2.*及以下`（可能版本稍有出入，如果你有这样一段代码：

```css
input:checked ~ .test {
  background-color: #f00;
}
```

那么将无任何效果，如果你想使得上述代码生效，有2种方式：

第一种，使用 `input` 和 `+` 进行激活：

```css
html + input {}
input:checked ~ .test {
  background-color: #f00;
}
```

只要存在 `input`和 `+` 选择符配合使用的选择器（空规则集也行）即可使得上述代码激活生效。

第二种，直接换成 `+`：

```css
input:checked + .test {
  background-color: #f00;
}
```

<a name="flex"></a>
### 为什么flex布局不生效

* 使用[块级元素](http://blog.doyoe.com/2015/03/09/css/%E8%A7%86%E8%A7%89%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%A8%A1%E5%9E%8B%E4%B8%AD%E7%9A%84%E5%90%84%E7%A7%8D%E6%A1%86/#block-level-element)作为 `flex items（flex子项）`；

> `Android Browser4.3及以下`，`iOS Safari6.1及以下` 的 `flex子项` 需要使用块级元素，在这些版本之上还可以使用行内块元素

> 在这些版本中，如果你发现flex子项之间出现了间隙，或者在未定义换行的情况下子项自身抑或子项之间换行了，或者出现了其它不正常的情况，那么仔细看一下flex子项可能是使用了行内级元素；

* 当横向布局时，给 `flex子项` 子项定义 `width` 为非 `auto` 的值

> `Android Browser4.3及以下`，`iOS Safari6.1及以下` 的 `flex子项` 如果没有显式的定义 `width` 为非 `auto` 的值，那么子项分配父元素剩余空间时将会不符合标准预期；

* 当纵向布局时，给 `flex子项` 子项定义 `height` 为非 `auto` 的值

> `Android Browser4.3及以下`，`iOS Safari6.1及以下` 的 `flex子项` 如果没有显式的定义 `height` 为非 `auto` 的值，那么子项分配父元素剩余空间时将会不符合标准预期；

<a name="12px"></a>
### 为什么小于12px字号不生效

如果你是从`pc`开发转到移动平台的，或者应该记得在`pc`端，`Chrome`及后来加入Webkit阵营的`Opera`都不支持页面字号小于`12px`，当然你可以通过更改浏览器设置来改变这一情况，然后这并没有什么卵用，不是么？

不幸的是，在移动端这个限制也依然存在，在`Android Chrome`上（包括部分版本上的`Android Browser`），仍然不支持小于12px的字号（测试至Android5.0.2, Chrome46），除此之外，其他浏览器包括iOS上众浏览器都能够很好的支持超小字体。

所以，如果希望你的程序足够安全，尽量不要定义小于12px的字号，或者换一种方式来实现。

> 题外话：假设你的项目使用了`rem`，那么不要使用`10`作为换算因子，原因也如上

<a name="chrome-rem-bug"></a>
### chrome中body使用rem失效

我知道很多人已经开始使用 `rem` 作为项目中的单位了。但是遗憾的是，在 `Chrome` 和 `Opera` 上，如果我们给 `body` 元素应用了 `rem`，那么这个取值将会计算错误。

假设我们有如下代码：

```css
html {
  font-size: 62.5%;
}
body {
  font-size: 1.4rem;
}
```

因为大部分浏览器的默认字号都是 `16px`，所以 `html` 的字号计算出来应该是 `16px * 62.5% = 10px`。此时，我们预期 `body` 的 `font-size` 为 `14px`。然而实际情况与我们想象的不太一样，最终 `body` 的计算值并不是 `14px`，它忽略了 `html` 的定义，而是直接使用了浏览器的默认字号作为参照。于是最终计算值为：`16px * 1.4rem = 22.4px`。测至 `chrome 45.0` 和 `Opera 33.0` 仍然存在这个问题，不过 `chrome 49.0` 和 `Opera 37.0` 看起来已经被修复了。

为了有效的绕过这个问题，并且实现相同的效果，我们可以将代码修改如下：

```css
html {
  font-size: 62.5%;
}
body {
  font-size: 1.4em;
}
```

由于 `body` 是 `html` 的直接子元素，所以此时对 `body` 使用 `em` 与 `rem` 的效果是相同的。

<a name="html-percent-font-size"></a>
### 不要对html设置百分比字号

很严肃的和大家说，如果你在使用 `rem` 这项技术，那么尽可能不要对html设置百分比大小的字号。比如这样的：

```css
html {
  font-size: 62.5%;
}
```

由于大部分浏览器的默认字号是 `16px`，所以能计算出 `html` 的字号实际为 `10px`。我们在 [为什么小于12px字号不生效](#user-content-12px) 中说过，部分浏览器会将小于 `12px` 的字变成 `12px` 来显示。那么此时，在这些浏览器下，如果我做了这样的定义：

```css
.demo {
	width: 10rem;
}
```

你预期得到 `10px * 10rem = 100px`，但实际上得到的确是 `12px * 10rem = 120px`。这是非常大的错误，我们应当尽量避免。

与此同时，虽然大部分浏览器的默认字号是 `16px`，但仍然有使用其它默认值的浏览器，比如我依稀记得 `firefox` 使用了 `15px`。**而且最重要的是，用户是可以改变浏览器默认字号的**，所以你认为的可能并不是你认为的。

所以不要对html设置百分比字号，尤其是不要对它使用计算值比 `12px` 小的字号。我推荐大家这样做：

```css
html {
  font-size: 100px;
}
```

以 `100px` 作为因子，计算也非常方便。如果你想要设置一个元素的宽度是 `20px`，那么只需要：

```css
.demo {
	width: .2rem;
}
```

<a name="weixin-sign"></a>
###	单页应用微信签名

目前测试：微信JsSDK签名在`Android5.x`以上没发现任何问题，但在`IOS`上存在签名失败的问题，原因在于微信浏览器在IOS端使用的是苹果内核的浏览器，而在Android上使用的是谷歌开源的分支加上自身改造的浏览器内核。
*	详情的适配方案 https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1483682025_enmey


<a name="alipay-web-view"></a>
###	支付宝WebView单页应用问题

目前测试：因支付宝对WebView正在改造,且可能未完善好，支付宝对单页应用调用原生历史管理、页面标题接口存在缺陷与不稳定性，造成关联Api可能失效问题。建议前端使用用支付宝提供的[JSAPI](https://myjsapi.alipay.com/jsapi/index.html) ,[Alipay JSSDK](https://myjsapi.alipay.com/alipayjsapi/index.html),其中Alipay JSSDK 未暴露onBack等接口，但是可以从js文件中找得出。

<a name="experience"></a>
## Html经验

<a name="touch-callout"></a>
### 禁止保存或拷贝图像

通常当你在手机或者pad上长按图像 `img` ，会弹出选项 `存储图像` 或者 `拷贝图像`，如果你不想让用户这么操作，那么你可以通过以下方法来禁止：

```css
img {
	-webkit-touch-callout: none;
}
```

> 需要注意的是，该方法只在 `iOS` 上有效。

<a name="tap-highlight-color"></a>
### 取消touch高亮

在移动设备上，所有设置了伪类 `:active` 的元素，默认都会在激活状态时，显示高亮框，如果不想要这个高亮，那么你可以通过以下方法来禁止：

```css
.xxx {
	-webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}
```

<a name="user-select"></a>
### 禁止选中内容

如果你不想用户可以选中页面中的内容，那么你可以禁掉：

```css
html {
	-webkit-user-select: none;
}
```

<a name="overflow-scrolling"></a>
### 快速回弹滚动

1. 早期的时候，移动端的浏览器都不支持非body元素的滚动条，所以一般都借助 iScroll;
2. Android 3.0/iOS解决了非body元素的滚动问题，但滚动条不可见，同时iOS上只能通过2个手指进行滚动；
3. Android 4.0解决了滚动条不可见及增加了快速回弹滚动效果，不过随后这个特性又被移除；
4. iOS从5.0开始解决了滚动条不可见及增加了快速回弹滚动效果

在iOS上如果你想让一个元素拥有像 Native 的滚动效果，你可以这样做：

```css
.xxx {
	overflow: auto; /* auto | scroll */
	-webkit-overflow-scrolling: touch; /* 该规则可能引起iOS UIWebView崩溃 */
}
```

<a name="shortcut-title"></a>
### 设置添加到主屏幕的Web App标题

`iOS Safari` 允许用户将一个网页添加到主屏幕然后像 `App` 一样来操作它。我们知道每个 `App` 下方都会有一个名字，`iOS Safari` 提供了一个私有的 `meta` 来定义这个名字，代码如下：

```html
<meta name="apple-mobile-web-app-title" content="Web App名称" />
```

`Android Chrome31.0`，`Android Browser5.0` 也开始支持添加到主屏幕了，但并没有提供相应的定义标题的方式，所以如果你想统一 `iOS` 和 `Android` 平台定义 Web app 名称的方式，可以使用 `title` 标签来定义，代码如下：

```html
<title>Web App名称</title>
```

但如果你想要网页标题和App名字不一样的话，那就只有iOS才行。

<a name="shortcut-icon"></a>
### 设置添加到主屏幕的Web App图标

当我们将一个网页添加到主屏幕时，除了会需要设置标题之外，肯定还需要能够自定义这个App的图标，代码如下：

```html
<link rel="apple-touch-icon" href="app.png" />
```

不过该方案，在拟物设计的 `iOS6及以前` 会自动为图标添加一层高光效果，`iOS7` 已使用了扁平化设计，所以如果使用该方案，在不同版本下得到的效果会不一致。

当然，你也可以使用原图作为App的图标，用以保持各平台表现一致，代码如下：

```html
<link rel="apple-touch-icon-precomposed" href="app.png" />
```

如果你想给不同的设备定不同的图标，可以通过 `sizes` 属性来定义，形如：

```html
<link rel="apple-touch-icon" sizes="76x76" href="ipad.png@1x" />
<link rel="apple-touch-icon" sizes="120x120" href="iphone-retina@2x.png" />
<link rel="apple-touch-icon" sizes="152x152" href="ipad-retina@2x.png" />
<link rel="apple-touch-icon" sizes="180x180" href="iphone-retina@3x.png" />
```

规则如下：

* 如果没有跟相应设备推荐尺寸一致的图标，会优先选择比推荐尺寸大并且最接近推荐尺寸的图标。
* 如果没有比推荐尺寸大的图标，会优先选择最接近推荐尺寸的图标。
* 如果有多个图标符合推荐尺寸，会优先选择包含关键字precomposed的图标。

实际情况下，大部分智能手机都接近或者已经达到视网膜屏质量，所以如果想省事的话，可以分别为 `iPhone` 和 `iPad` 定义一种高质量的 `icon` 即可。

该方案在 `iOS` 和 `Android5.0+` 上都通用。

<a name="hide-bar"></a>
### 添加到主屏幕时隐藏地址栏和状态栏（即全屏）

当我们将一个网页添加到主屏幕时，会更希望它能有像 `App` 一样的表现，没有地址栏和状态栏全屏显示，代码如下：

```html
<meta name="apple-mobile-web-app-capable" content="yes" />
```

该方案在 `iOS` 和 `Android5.0+` 上都通用。

<a name="status-bar-style"></a>
### 添加到主屏幕时设置系统顶栏颜色

当我们将一个网页添加到主屏幕时，还可以对 `系统显示手机信号、时间、电池的顶部状态栏` 颜色进行设置，前提是开启了：

```html
<meta name="apple-mobile-web-app-capable" content="yes" />
```

有了这个前提，你可以通过下面的方式来进行定义：

```html
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
```

content只有3个固定值可选：default | black | black-translucent

* 如果设置为 `default`，状态栏将为正常的，即白色，网页从状态栏以下开始显示；
* 如果设置为 `black`，状态栏将为黑色，网页从状态栏以下开始显示；
* 如果设置为 `black-translucent`，状态栏将为灰色半透明，网页将充满整个屏幕，状态栏会盖在网页之上；

该设置只在 `iOS` 上有效。

<a name="tel"></a>
### 电话号码识别

在 `iOS Safari` （其他浏览器和Android均不会）上会对那些看起来像是电话号码的数字处理为电话链接，比如：

* 7位数字，形如：1234567
* 带括号及加号的数字，形如：(+86)123456789
* 双连接线的数字，形如：00-00-00111
* 11位数字，形如：13800138000

可能还有其他类型的数字也会被识别，但在具体的业务场景中，有些时候这是不必须的，所以你可以关闭电话自动识别，然后在需要拨号的地方，开启电话呼出和短信功能。

1. 关闭电话号码识别：

```html
<meta name="format-detection" content="telephone=no" />
```

2. 开启拨打电话功能：

```html
<a href="tel:123456">123456</a>
```

3. 开启发送短信功能：

```html
<a href="sms:123456">123456</a>
```

<a name="email"></a>
### 邮箱地址识别

在 `Android` （iOS不会）上，浏览器会自动识别看起来像邮箱地址的字符串，不论有你没有加上邮箱链接，当你在这个字符串上长按，会弹出发邮件的提示。

1. 关闭邮箱地址识别：

```html
<meta name="format-detection" content="email=no" />
```

2. 开启邮件发送：

```html
<a href="mailto:dooyoe@gmail.com">dooyoe@gmail.com</a>
```

> 如果想同时关闭电话和邮箱识别，可以把它们写到一条 meta 内，代码如下：

```html
<meta name="format-detection" content="telephone=no,email=no" />
```

<a name="autocapitalize"></a>
### 关闭iOS键盘首字母自动大写

在iOS中，默认情况下键盘是开启首字母大写的功能的，如果业务不想出现首字母大写，可以这样：

```html
<input type="text" autocapitalize="off" />
```

<a name="autocorrect"></a>
### 关闭iOS输入自动修正

在iOS中，默认输入法会开启自动修正输入内容的功能，如果不需要的话，可以这样：

```html
<input type="text" autocorrect="off" />
```

<a name="text-size-adjust"></a>
### 禁止文本缩放

当移动设备横竖屏切换时，文本的大小会重新计算，进行相应的缩放，当我们不需要这种情况时，可以选择禁止：

```css
html {
	-webkit-text-size-adjust: 100%;
}
```

> 需要注意的是，PC端的该属性已经被移除，该属性在移动端要生效，必须设置 `meta viewport'

```html
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,minimal-ui">
```
<a name="img-block"></a>
###	Img的跨域问题
 通过实例新建的`image`对象存在协议、域名不同的问题会产生警告导出出现与之设计不符的问题（比如图片加载不出等），建议修改资源的协议与当前页面的协议一致。

<a name="ios-audio-tag"></a>
###	IOS音频播放问题
`IOS` Audio标签默认情况下的	`preload`、`autoplay`无效，当第一次的触发播放的时候，音频流才开始加载，且`初始化一个新的音频流时会有几秒的延时`，所以有几率出现触发播放了却没有播放，因此建议适当延时播放

<a name="ios-audio-invalid-play"></a>
### IOS Audio 调用`play`无效
可尝试添加以下代码
``` javascript
  const audioPlayer = this.$refs.audioPlayer
  audioPlayer.play()
  audioPlayer.pause()
  document.addEventListener('WeixinJSBridgeReady', () => {
    audioPlayer.play()
    audioPlayer.pause()
  }, false)
```

<a name="vue-experience"></a>
##	Vue经验

<a name="vue-event-bus-context"></a>
### EventBus当前对象问题

使用global event bus 时，接收者`$on`必须处于`created`之后的状态,且注意当前实例`this`对象需保持同一个里面

<a name="vue-vuex-probrem"></a>
### Vuex使用问题

1.	使用`Vuex`时，在页面结束的生命周期要清理脏数据
2.	 动态注册`Vuex`时，每个模块的`state，getters、actions，mutations`要构造函数生成,尽量使用`namespace`做域名限制，避免同名、多次注册产生数据冗余、覆盖
3.	当要修改Vuex的`State`时， 只能通过`Commit`的方式进行提交修改, 因为`mutation `都是同步事务，例如：

```js
computed:{
        …mapGetters({ defaultPageSize:mGetters.GET_DEFAULT_PAGE_SIZE})
},
methods:{
       getAdc:function(){
            // 非法操作 
            this. defaultPageSize++;
             let  defaultPageSizeA = this. defaultPageSize;
            defaultPageSizeA ++;
            // 正确操作
            let {defaultPageSize} = this;
	    defaultPageSize++;
	}
}
```

<a name="vue-list-render-probrem"></a>
### 列表渲染问题

1.	使用列表渲染时，一定要加上一个`key`值，降低因刷新数据而消耗不必要的渲染问题，例：

```html
    <div v-for="(item,index) in 5" :key="index">
      <div>{{item}}</div>
    </div>
```

2.	由于 `JavaScript` 的限制，`Vue `不能检测以下变动的数组：
	*	当你利用索引直接设置一个项时，例如：`vm.items[indexOfItem] = newValue`
	*	当你修改数组的长度时，例如：`vm.items.length = newLength`
	*	为了解决第一类问题，以下两种方式都可以实现和 `vm.items[indexOfItem] = newValue` 相同的效果，同时也将触发状态更新：

	```js
	// Vue.set
	Vue.set(example1.items, indexOfItem, newValue)

	// Array.prototype.splice
	example1.items.splice(indexOfItem, 1, newValue)

	为了解决第二类问题，你可以使用 splice：
	example1.items.splice(newLength)
	```

<a name="design-component-rule"></a>
##	组件划分原则
*	展示组件
	1.	关心应用的外观。
	2.	可能包含展示组件或容器组件，除此之外常常还会包含属于组件自身的DOM结点与样式信息
	3.	对应用的其余部分（store）没有依赖
	4.	不回指定数据如何加载或改变。
	5.	只通过props、emit获取数据与行为。
	6.	极少包含在自身的状态，如果有，一定是界面状态而非数据。

*	容器组件
	1.	关心应用如何工作。
	2.	可能包含展示组件或容器组件，但通常不回包含Dom结点（除包裹用的根节点）
	3.	为展示组件或其他容器组件提供数据与行为（回调函数）
	4.	往往是用状态的，扮演数据源的角色

*	好处
	1.	展示和容器更好的分离，更好的理解应用程序和UI
	2.	通过职责将组建明确地的划分，应用的界面与逻辑都会变得更加清晰。
	3.	让我们更好的复用组件。展示组件具有更好的复用性，他们可以通过包裹的不同的数据源成为不同容器的组件。

	如：（容器型）头像组件的设计![头像组件](./asset/AvatarComponent.png "头像组件")