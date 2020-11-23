## 核心API

1. 获取一个元素在页面上的真实位置(非当前窗口视角,也就是说如果往下滚动了,那么滚动部分的高度也要算到位置高度里)

方法:
(1)jquery.offset
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>offset demo</title>
  <style>
  p {
    margin-left: 10px;
    color: blue;
    width: 200px;
    cursor: pointer;
  }
  span {
    color: red;
    cursor: pointer;
  }
  div.abs {
    width: 50px;
    height: 50px;
    position: absolute;
    left: 220px;
    top: 35px;
    background-color: green;
    cursor: pointer;
  }
  </style>
  <script src="https://code.jquery.com/jquery-3.5.0.js"></script>
</head>
<body>
 
<div id="result">Click an element.</div>
<p>
  This is the best way to <span>find</span> an offset.
</p>
<div class="abs">
</div>
 
<script>
$("*", document.body).click(function( event ) {
  var offset = $(this).offset();
  event.stopPropagation();
  $("#result").text(this.tagName + "coords ( " + offset.left + ", " + offset.top + " )" );
});
</script>
</body>
</html>
```

(2)递归
```
会导致元素位置偏移的属性有哪些?
margin
float
flex
position 非 static 时, left, top
transform 里的 translate scale
```
(3)getBoundingClientRect

2. reduce

3. compose

4. apply, bind 的实现
