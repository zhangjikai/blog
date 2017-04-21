title: 【HTML5】图片预加载
date: 2015-06-22 19:28:07
tags: HTML5
categories: HTML5
---
在HTML5中，我们可以使用drawImage方法在canvas上进行画图操作，其基本代码如下：
```js
var canvas = document.getElementById("canvas");
var context = canvas.getContext("2d");
var image = new Image();
image.src = "images/01.jpg";
context.drawImage(image, 0, 0);  
```
不过我们会发现这样写是无法显示出图片的，因为图片并没有加载完全，我们就调用了drawImage方法，我们可以使用img的onload方法，使图片加载完全后
在执行drawImage操作，代码如下
```js
var image = new Image();
image.src = "images/01.jpg";
image.onload = function() {
    context.drawImage(image, 0, 0);
}
```
或者使用`<img>`标签先加载图片  
`<img src="images/01.jpg" style="display: none" id="image">`  
然后使用getElementById来获得图片对象  
`var image = document.getElementById('image');`  
<!-- more -->
但是在图片较多的情况下，使用上面两种方式都不是太优雅，我们可以使用下面的方法，等待所有图片加载完全后，再执行其他操作
```javascript
    document.addEventListener("DOMContentLoaded", loadImages, true);

    var images = new Array(3), imageNums = 0;

    function loadImages() {
        for (var i = 0; i < images.length; i++) {
            images[i] = new Image();
            images[i].addEventListener("load", trackProcess, true);
            images[i].src = "images/01.jpg";
        }
    }

    function trackProcess() {
        imageNums++;
        if (imageNums = images.length) {
            drawImages();
        }
    }

    function drawImages() {
        var canvas = document.getElementById("canvas");
        var context = canvas.getContext("2d");
        for (var i = 0; i < images.length; i++) {
            context.drawImage(images[i], 200 * i, 0);
        }
    }
```
参考文章: [Preloading Images](http://www.kirupa.com/html5/preloading_images.htm)
