title: 【CSS 学习笔记】CSS元素和布局
date: 2016-11-29 20:49:18
tags: ["学习笔记", "CSS"]
categories: CSS
---
## 前言
本文绝大部分摘自 `CSS 权威指南 第三版`
## 基本概念
* **正常流 (Normal Flow)：** 有时会被翻译为 **文档流** 或者 **普通流**，指文档从左至右、从上至下的显示内容，是传统的 HTML 文档布局。如果使元素不在正常流中，可以使用浮动（`float`）或者定位（`absolute`, `fixed`）。
* **块级元素 (Block-level)：** 块级元素在**普通流**中会独占一行，即在其框之前和之后生成“换行”，因此处于**普通流**中的块级元素会按照从上到下的顺序垂直（vertically）排列。常见的 block 元素有 `div`, `p`, `h1-h6`, `ul`, `li`, `canvas`, `table` 等。完整的元素可以参考[这里](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements)。通过使用 `display:block`，可以将元素生成块级框。
* **内联元素 (Inline)：** 或者称为 **行内元素**。在**普通流**中的内联元素之间不会生成“行分割符”，因此处于**普通流**中的内联元素会首先按照从左至右的顺序水平（horizontally）排列，当父容器水平方向上的剩余宽度不足以放下新的内联元素时，会往下换行，在新行的中继续按照水平顺序排列元素。常见的 inline 元素有：`a`, `img`, `button`, `br`, `input`, `label`, `select`, `textarea`。完整的可以参考[这里](https://developer.mozilla.org/en-US/docs/Web/HTML/Inline_elements)通过使用 `display:inline` 可以让元素变成内联元素。
* **非替换元素 (non-replaced)：** 如果元素的内容包含在文档中，就成为非替换元素。 例如段落 `<p>`。
* **替换元素 (replaced)：** 可以理解为嵌入元素，相当于一个占位符，解析时会被其他内容替换。例如 `<img>` 和大部分表单元素 `<input type="radio">`。
* `em`: 1em等于 `font-size` 的设置值

<!-- more -->
## 盒模型 (Box-Model)
![](/images/css_layout/box-model.png)  

在盒子模型中，水平和垂直方向上各有7个属性:
* 水平方向 - `margin-left`, `border-left`, `padding-left`, `width`,  `padding-right`,  `border-right`,  `margin-right`
* 垂直方向 - `margin-top`,  `border-top`,  `padding-top`,  `height`, `padding-bottom`, `border-bottom`, `margin-bottom`

其中 `margin` 称为外边距，在计算元素整体宽高的时候一般不包括它。CSS3 中新增了一个属性 `box-sizing`，可以用来指定使用的盒模型计算方式。下面是 CSS3 中支持的盒模型计算方式（CSS2种只支持默认的）
+ `content-box`（默认值）: `width` 和 `height` 属性只作用到 Content Area 的长宽，在 Content Area 外面绘制内边距和边框，见图 (1)。
    - `Width = width + padding-left + padding-right + border-left + border-right`
    - `Height = height + padding-top + padding-bottom + border-top + border-bottom`
+ `border-box`: `width` 和 `height` 属性设置的值就为元素整体的宽高，内边距和边框在已设定的宽度和高度内进行绘制，见图 (2)。
    - `Width = width`
    - `Height = height`
+ `inherit`: 继承父类的属性

![图 (1) : content-box](/images/css_layout/box-model-1.png)  

![图 (2) : border-box](/images/css_layout/box-model-2.png)  


## 块级元素
<!--### 水平属性
通过上面的盒子模型，我们可以看到水平方向上对应7个属性 `margin-left`, `border-left`, `padding-left`, `width`, `padding-right`, `border-right`, `margin-right`。默认的盒模型计算方式为`content-box`（CSS3 新增了两个盒模型计算方式：`padding-box` 和 `border-box`，具体可以参考[这里](https://leohxj.gitbooks.io/front-end-database/content/html-and-css-basic/box-module.html)）。-->
### auto
在上面提到的几个属性中，只有`margin`, `width`, `height` 可以设为 `auto`，`padding` 和 `border` 必须设定为特定值或者使用默认值。
#### 水平属性
在上面提到的7个水平属性中，只有3个值可以设置为 `auto`：`width`, `margin-left`, `margin-right`。其余属性必须设置为特定的值或者使用默认值。下面是使用 `auto` 的几种情形：
* 没有一个属性设为 `auto`： 如果 `width`, `margin-left`, `margin-right` 这三个属性都设为非 `auto` 的特定值，那么会将 `margin-right` 强制为 `auto`。
* 有且只有一个属性设为 `auto`： 如果三个属性中某个值设为 `auto`，而余下的两个属性设为特定的值，那么设置为 `auto` 的属性值会自动确定所需长度，从而使元素框的总宽度（上面提到的7种属性相加）等于父容器的 `width`。
* 两个外边距都设为 `auto`，`width` 设为特定值： 元素会居中（常用的居中方式），`margin-left` 和 `margin-right` 会设为相等的长度
* `width` 设为 `auto`，外边距有一个或者两个均设为 `auto`： 设为 `auto` 的外边距会变成0，如果两个外边距都设为 `auto`，会都变为0。

#### 垂直属性
* 如果 `margin-top` 和 `margin-bottom` 都设为 `auto`（对于定位元素会有不同），会将它们计算为0。
* `height` 设为 `auto`，一般等于其包含的子元素的总高度。

### 外边距合并
针对垂直外边距（`margin-top` 和 `margin-bottom`），两个相邻的垂直外边距会合并成一个外边距，两个外边距中较小的一个会被较大的一个合并。详细内容可以参考 [这里](http://www.w3school.com.cn/css/css_margin_collapsing.asp) 。

![](/images/css_layout/margin-collapse.gif)  

如果外边距中有负值：
* 如果相邻的垂直外边距都设为负值，会取外边距中绝对值较大的那个外边距。例如一个外边距为：`margin-bottom:-10px`，和它相邻的另外一个为：`margin-top:-20px`，会保留 `margin-top:-20px`。
* 如果一个正外边距和一个负外边距，会从正外边距减去负外边距的绝对值。例如一个外边距为: `margin-bottom:20px`，另外一个为: `margin-top:-10px`，最终的效果相当于 `margin-bottom:10px`。

### 负外边距
外边距可以是负的，即 `margin` 可以设为负值，此时子元素的 `width` 或者 `height` 就有可能大于父元素的 `width`。只有外边距能小于0，内边距、边框和内容的宽高都不能设为负值。

## 内联元素
东西比较多，先附一些文章链接：
* [CSS 中的line-height](https://segmentfault.com/a/1190000003038583)
* [CSS 行高line-height的一些深入理解及应用](http://www.zhangxinxu.com/wordpress/2009/11/css%E8%A1%8C%E9%AB%98line-height%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E5%8F%8A%E5%BA%94%E7%94%A8/)
* [CSS line-height 中文版](http://www.slideshare.net/daemao/line-height-2470819)
* [视觉格式化模型中的各种框](http://blog.doyoe.com/2015/03/09/css/%E8%A7%86%E8%A7%89%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%A8%A1%E5%9E%8B%E4%B8%AD%E7%9A%84%E5%90%84%E7%A7%8D%E6%A1%86/)
* **CSS权威指南**

### 行内框（inline-box）
[http://meyerweb.com/eric/css/inline-format.html](http://meyerweb.com/eric/css/inline-format.html)  
[https://www.w3.org/TR/CSS21/visuren.html#inline-box](https://www.w3.org/TR/CSS21/visuren.html#inline-box)  

行内框通过向内容区（context-area）增加行间距（leading）来描述。对于非替换元素来说，元素行内框的高度刚好等于 `line-height` 的值。对于替换元素来说，元素行内框的高度等于元素的 `height + margin-top + margin-bottom + padding-top + padding-bottom + border-top + border-bottom`。
浏览器会根据行内元素行内框的大小来对元素布局。假设行内元素的内容区高 `20px`，但是 `line-height` 只有 `14px`，那么为该元素分配的高度只有 `14px`，就会出现内容去溢出的情况（覆盖其他的行元素）。

#### 非替换元素
* margin, border, padding 不影响行内框的高度，但是会影响行内框的宽度。
* `width` 和 `height` 属性不会作用于行内非替换元素，即不能设置宽高。

#### 替换元素
* 替换元素的 margin, border. padding 会影响行内框的宽度和高度
* 可以对替换元素设置 `width` 和 `height`。如果不设置宽高，会使用元素本来的宽度和高度。

#### 设置line-height的几种方式
`line-height` 只作用于内联元素或者其他的内联内容。
* `normal` - 默认值，设置合理的行间距（1.2）
* 具体的长度 - `12px` 、`1em` 等等
* 纯数字 - 和当前 `font-size` 的比值
* 百分比 - 和当前 `font-size` 的百分比
* `inherit` - 从父类中继承

#### 注意点
* 内联非替换元素的 `width` 和 `height` 是不起作用的
* padding 和 border 不改变 `line-height`
* `margin-top` 和 `margin-bottom` 不作用于行内非替换元素，比如 `span`

## 修改显示类别
使用 `display:value` 可以修改元素的类别。有效值如下：


| 值                 | 描述                                                             |
|--------------------|------------------------------------------------------------------|
| none               | 此元素不会被显示。                                               |
| block              | 此元素将显示为块级元素，此元素前后会带有换行符。                 |
| inline             | 默认。此元素会被显示为内联元素，元素前后没有换行符。             |
| inline-block       | 行内块元素。（CSS2.1 新增的值）                                  |
| list-item          | 此元素会作为列表显示。                                           |
| run-in             | 此元素会根据上下文作为块级元素或内联元素显示。                   |
| compact            | CSS 中有值 compact，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。 |
| marker             | CSS 中有值 marker，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。  |
| table              | 此元素会作为块级表格来显示（类似 table），表格前后带有换行符。 |
| inline-table       | 此元素会作为内联表格来显示（类似 table），表格前后没有换行符。 |
| table-row-group    | 此元素会作为一个或多个行的分组来显示（类似 tbody）。           |
| table-header-group | 此元素会作为一个或多个行的分组来显示（类似 thead）。           |
| table-footer-group | 此元素会作为一个或多个行的分组来显示（类似 tfoot）。           |
| table-row          | 此元素会作为一个表格行显示（类似 tr）。                        |
| table-column-group | 此元素会作为一个或多个列的分组来显示（类似 colgroup）。        |
| table-column       | 此元素会作为一个单元格列显示（类似 col）                       |
| table-cell         | 此元素会作为一个表格单元格显示（类似 td 和 th）              |
| table-caption      | 此元素会作为一个表格标题显示（类似 caption）                   |
| inherit            | 规定应该从父元素继承 display 属性的值。                          |

* `inline-block`：会使元素表现的像行内非替换元素一样，是行内元素，但是可以设置宽高，margin, border, padding 会影响行内框的高度
* `run-in`：使某些块级元素成为下一个元素的行内元素(chrome不支持)。

## 浮动(float)

[MDN float](https://developer.mozilla.org/en-US/docs/Web/CSS/float)
[w3 float](https://www.w3.org/TR/CSS2/visuren.html#floats)
### 定义
下面是MDN上关于 `float` 的定义
> The float CSS property specifies that an element should **be taken from the normal flow** and placed along the **left or right side of its container**, where **text and inline elements will wrap around it**.

根据定义需要注意的有下面三点：
1. 浮动元素会脱离正常流。
2. 浮动元素会被放置在所在容器的左侧或者右侧。
3. 文字和行内元素会环绕浮动元素，所以会影响布局。

### 其他注意点

* 浮动元素会生成一个块级框，即便元素本身是行内元素，也会生成块级框。所以不需要为浮动元素声明 `display:block`。
* 浮动元素的外边距不会合并。
* 浮动元素之间一般不会重叠（外边距设为负值就可能会重叠），会按照顺序排序，如果当前行剩余的宽度不足以放下新的元素，会另起一行。
* 浮动元素会延伸，从而包含其所有的代浮动元素。

### 重叠
如果浮动元素和正常流中的内容发生重叠（浮动元素的外边距为负值），会按照以下规则显示内容：
* 行内框和一个浮动元素重叠时，其边框、背景和内容都会在该浮动元素 **之上** 显示
* 框框与一个浮动元素重叠时，其边框和背景在该浮动元素 **之下** 显示，内容在浮动元素 **之上** 显示

### 清除浮动

清除浮动就是让元素的左边或者右边或者两边不会有浮动元素出现。清除浮动的一个主要的原因就是增加父容器的高度，当子元素浮动时，会脱离正常流，因此父元素计算高度时不会加上浮动子元素的高度，就会造成父元素的高度小于浮动子元素。当清除浮动之后，父容器就可以正确高度。下面是清除浮动的几种方式，更多方式可以参考 [这里](http://lightcss.com/all-about-clear-float/) ：

* 使用带clear元素的空属性
    ```
    .clear{
        clear:both;
    }
    <div class="clear"></div>
    ```
* 使用 `:after` 伪元素
    ```css
    .clearfix:after {
        clear:both;
        content:'\20';
        display:block;
        height: 0;
        visibility:hidden
    }
    <div class="clearfix"></div>
    ```
* 在父容器里添加 `overflow:auto` 或者 `overflow:hidden`

## 定位
CSS 有三种基本的定位机制: 正常流、浮动和绝对定位。使用 `position` 可以设置不同类型的定位方式。下面是 `position` 属性值的定义:
* `static`：默认值，元素框正常生成，不会被特殊的定位。块级元素生成块级块，行内元素生成一个或者多个行框，置于其父元素中。
* `relative`: 元素框偏移某个距离。元素仍保持其未定位前的形状，它原本所占的空间仍保留。`relative` 的表现和 `static` 十分类似，不同的是相对于定位参考的是它**应该在的位置**(或者说它自身的位置)，通过使用偏移属性 `top`, `bottom`, `left` 和 `right` 属性会使元素**相对于** 它的**起点**进行移动。**其他元素的位置不会受到影响**。
* `absolute`: 元素会脱离正常流，相对于其最近的非 static 定位的祖先元素定位，如果没有满足条件的祖先元素，则会相对于文档的 `body` 元素。元素在正常流中的所占的位置会被清除，就好像该元素不存在一样。`absolute` 元素会生成一个块级框。
* `fixed`: 和 `absolute` 类似，不过其定位的参考元素是视窗，当页面滚动时还是会停留在原先的位置。 `absolute` 会跟随父元素滚动。

### 其他注意点
* 一般称 `relative`, `absolute` 以及 `fixed` 为定位元素 (positioned)
* 除了 `static`，其他三种定位都可以使用偏移属性 `top`, `bottom`, `left`, `right`。
* `absolute` 定位里 `left`,  `right`, `width`，有一个值设为 `auto`，会自动调整其大小，使总长度相加等于父容器宽度。如果有没有auto，会重置 `right`。`top` 和 `bottom` 类似。

### z-index
利用 `z-index` 可以修改元素相互的覆盖顺序。所有数都可以作为 `z-index` 的值，包括负数。需要注意的是
* `z-index` 只能作用于定位元素，`static` 元素会失效
* 子元素会继承父元素的 `z-index`，子元素设置的 `z-index` 是相对于父元素的局部 `z-index`。比如下面的代码：
```css
.p1 {
    position:absolute;
    z-index:2;
}
.p2 {
    position:absolute;
    z-index:1;
}
.p1 .c {
    z-index: 10;
}
.p2 .c{
    z-index:100;
}
```
`.p2 .c` 会在 `.p1 .c` 的下面
