# 1.Ajax概述

Ajax: Asynchronous javascript and xml (异步javascript和xml)。

Ajax并不是一种新技术，而是已有技术的集合。

JavaScript是核心载体。

Ajax优势：在不刷新页面的情况下，更新页面数据，提升用户体验Ajax就像一个小秘书，能够实现异步工作。

# 2.发送Ajax请求

## 2.1 ajax核心对象--XMLHttpRequest

历史：1999年诞生，微软在IES中集成了XMLHttpRequest对象，但是并没有收到人们的重视，2005年，Google在 gtalk即时聊天工具中使用了改对象，从此之后Ajax技术开始受到人们的重视.

创建XMLHttpRequest对象要分为IE和非IE两种方式：

IE浏览器（IE7之前，包括ie7）：var  xhr = new ActiveXObject('Msxml2.XMLHTTP');

非IE浏览器（主流浏览器：charom firefox safair 包括IE7之后）：var xhr = new XMLHttpRequest( );

兼容操作：

```
function getXhr () {
 var xmlhttp;
    if (window.XMLHttpRequest) {
        //IE7+ 和 非IE 中都有 XMLHttpRequest对象
        xmlhttp = new XMLHttpRequest();
    } else {
        xmlhttp = new  ActiveXObject('Msxml2.XMLHTTP');
    }
    return xmlhttp;
}
```

## 2.1核心方法

对象有了，就可以发送请求了，发送求需要两个方法：

open（‘请求方式：get/post’，‘请求后端程序地址’，‘异步（true）/同步（false），默认true’）；

send（‘var’）；var：分为两种情况，如果是get请求，则填写null或者不填写即可，如果是post请求，则填写要发送到后端的数据

# 3.接受后端响应的结果

## 3.1核心属性--readystate

Ajax的整个过程有5个状态，对应readystate的5个值 0---4

0---（未初始化） 对象已建立，但是尚未初始化（尚未调用open方法）

1---（载入）已经调用send()方法，正在发送请求

2---（载入完成） send方法调用完成，但是当前的状态及http头未知

3---（交互） 已接收部分数据，因为响应及http头不全，这时通过responseBody和responseText获取部分数据会出现错误

4---（完成） 数据接收完毕，此时可以通过通过responseBody和responseText获取完整的回应数据

## 3.2核心事件---onreadystatechange

readyState的值每次发生变化都会触发该事件0-->1    1-->2    2-->3    3-->4 总共触发4次

## 3.3其他重要的属性

responseText：以字符串形式接受后端程序返回的值

responseXML：以XML格式接收后端程序返回值---基本不用了

jQuery的$.ajax方法响应数据类型有如下几种：xml、html、script、json、jsonp、text

本质上原生ajax响应数据格式只有2种：xml和text，分别对应xhr.responseText和xhr.responseXML

# 4.综合案例

 点击按钮，从student表中随机取出一条学生的信息，显示在网页上

思路分析：

1.需要创建两个页面前端静态页面以及后端php页面

2.在html页面中设置一个按钮，用来触发Ajax请求→在php页面中随机取出一条学生的信息，返回给html页面→html页面接收数据然后显示在页面中

```
<!--
1. 创建一个按钮一个div --按钮用来发送ajax请求，div用来显示结果
2. 按钮上绑定点击事件
3. 事件函数 --- 发送ajax请求
1) 实例化 XMLHttpRequest 对象
2) 调用open方法，准备ajax请求
3) 调用send方法，发送ajax请求
4) 调用onreadystatechange事件，监控readyState的值，
当readyState=4时，使用responseText来接收后端返回的数据
-->
<input type="button" value="btn" id="btn">
<div id="d"></div>

<script type="text/javascript">
//2. 按钮上绑定点击事件
document.getElementById('btn').onclick = function () {
//1) 实例化 XMLHttpRequest 对象
var xhr = getXhr();

//4) 调用onreadystatechange事件，监控readyState的值，
// 当readyState=4时，使用responseText来接收后端返回的数据
xhr.onreadystatechange = function () {
if (xhr.readyState == 4) {
//alert(xhr.responseText);
//将接收到的后端数据显示到div中
document.getElementById('d').innerText = xhr.responseText;
}
}

//2) 调用open方法，准备ajax请求
xhr.open('get', './index.php');
//3) 调用send方法，发送ajax请求
xhr.send(null);
}
</script>
```

后端页面

```
<?php
header('content-type:text/html;charset=utf-8');
//1. 拼接SQL语句
$id = rand(1,7);
$sql = "select * from ali_admin where admin_id=$id";
//echo $sql;

//2. 链接MySQL服务器并执行SQL语句
$conn = mysql_connect('localhost', 'root', 'root');
mysql_select_db('test');
mysql_query('set names utf8');

$resource = mysql_query($sql);
$result = mysql_fetch_assoc($resource);
//print_r($result);

//双引号+大括号可以处理一切字符串拼接问题
echo "我是管理员{$result['admin_nickname']}, 我的人生格言是{$result['admin_sign']}";
?>
```

# 5.GET和POST

## 1.用GET方式实现新用户注册---用户名检测

第一步：get.html页面1）在用户名文本框上绑定失去光标焦点事件（onblur）2）失去焦点的函数A：获取用户名文本框已填写的用户名B：发送Ajax请求，并且将已经填写好的用户名一起发送给后端页面 

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
<script src="../xhr.js"></script>
</head>
<body>
<form action="">
用户名: <input id="txt" type="text" name="username"> <span id="s"></span><br>
密 码: <input type="text" name="password"><br>
<input type="submit" id="btn" value="注册">
</form>
<script>
//1.在用户名文本框上绑定失去光标焦点的事件
document.getElementById('txt').onblur = function () {
//2.获取用户文本框已经填写的内容
// 发送Ajax请求，并将填写好的用户名一起发送
// 2-1.获取输入的内容
var name = this.value;
//2-2.发送请求，实例化XML对象
var xhr = getXhr();
//调用事件，检测状态值
xhr.onreadystatechange = function () {
if(xhr.readyState == 4) {
alert(xhr.responseText);
//接收后端返回的数据，判断是1，还是2
//获取span标签，需要给span标签中添加内容
// var span_obj = document.getElementById('s');
// if(xhr.responseText == 1) {
// //如果等于1，说明可用
// span_obj.innerText = '用户名可用';
// span_obj.style.color = 'green';
// } else {
// //不等于1 书名用户名已经被占用，不可用
// span_obj.innerText = '用户名不可用';
// span_obj.style.color = 'red';
// }
}
}
//调用open方法发送准备发送请求
xhr.open('get','get.php?n=' + name);
//调用send方法，发送请求
xhr.send(null);
}
</script>
</body>
</html>
```

第二步：get.php页面接收前端发送过来的用户名模拟用户被占用的情况将结果返回给前端（1用户名可以用，2用户名被占用）

```
<?php
// echo "后端的返回数据是" . $_GET['n'];
//接收前端发送过来的用户名
$email = $_GET['n'];
//拼接SQL语句
$sql = "select * from ali_admin where admin_email='$email'";
//链接数据库并执行SQL语句
$conn = mysql_connect('localhost','root','root');
mysql_select_db('test');
$resource = mysql_query($sql);
$result = mysql_fetch_assoc($resource);

//将结果返回给前端页面
if(empty($result)) {
//为空时，说明数据表中没有该数据，用户名可用
echo 1;
} else {
//输出2，表示不为空，说明数据表中有该数据，用户名不可用
echo 2;
}
?>
```

第三步：get.html页面1）接收后端返回的数据，判断1还是22）如果是1，提示用户名可以用，并且给span中添加内容如果是2，提示用户名不可用

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
<script src="../xhr.js"></script>
</head>
<body>
<form action="">
用户名: <input id="txt" type="text" name="username"> <span id="s"></span><br>
密 码: <input type="text" name="password"><br>
<input type="submit" id="btn" value="注册">
</form>
<script>
//1.在用户名文本框上绑定失去光标焦点的事件
document.getElementById('txt').onblur = function () {
//2.获取用户文本框已经填写的内容
// 发送Ajax请求，并将填写好的用户名一起发送
// 2-1.获取输入的内容
var name = this.value;
//2-2.发送请求，实例化XML对象
var xhr = getXhr();
//调用事件，检测状态值
xhr.onreadystatechange = function () {
if(xhr.readyState == 4) {
alert(xhr.responseText);
// 接收后端返回的数据，判断是1，还是2
// 获取span标签，需要给span标签中添加内容
var span_obj = document.getElementById('s');
if(xhr.responseText == 1) {
//如果等于1，说明可用
span_obj.innerText = '用户名可用';
span_obj.style.color = 'green';
} else {
//不等于1 书名用户名已经被占用，不可用
span_obj.innerText = '用户名不可用';
span_obj.style.color = 'red';
}
}
}
//调用open方法发送准备发送请求
xhr.open('get','get.php?n=' + name);
//调用send方法，发送请求
xhr.send(null);
}
</script>
</body>
</html>
```

## 2.POST方式实现新用户注册---用户名检测

get.html页面

与get方式的区别：

1.发送请求时，参数1为post，参数2只设置请求的后端文件路径

2.需要将传递到后端的数据拼接成一个字符串

3.需要调用setRequestHeader方法，将数据格式转为 application/x-www-form-urlencoded4.发送请求时，将拼接好的字符串放入send方法中

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
<script src="../xhr.js"></script>
</head>
<body>
<form action="">
用户名:<input type="text" id="txt" name="username"><span id="s"></span><br>
密 码: <input type="text" name="password"><br>
<input type="submit" id="btn" value="按钮">
</form>
<script>
//获取页面中需要操作的元素
document.getElementById('txt').onblur = function () {
//获取应户名文本框输入的内容
var name = this.value;
//实例化对象
var xhr = getXhr();
var span_obj = document.getElementById('s');
//检测状态值
xhr.onreadystatechange = function () {
if(xhr.readyState == 4) {
// alert (xhr.responseText);
if(xhr.responseText == 1) {
span_obj.innerText = '用户名可用';
span_obj.style.color = 'green';
} else {
span_obj.innerText = '用户名不可用';
span_obj.style.color = 'green';
}
}
}
//调用open方法，准备发送请求
xhr.open('post','post.php');

//拼接需要传入的数据,将传递到后端的数据定义成一个字符串
var str = 'name=' + name;
//调用setRequestHeader 方法将数据的格式转为application/x-www-form-urlencoded
//重设请求头
xhr.setRequestHeader('content-type', 'application/x-www-form-urlencoded')
//调用send方法，发送请求
xhr.send(str);
}
</script>
</body>
</html>
```

后端页面post.php与get方式的区别1.接收数据的方式使用post方式

```
<?php
// echo "返回前端的数据是:" . $_POST['name'];
$email = $_POST['name'];

$sql = "select * from ali_admin where admin_email='$email'";
//链接数据库
$conn = mysql_connect('localhost','root','root');
mysql_select_db('test');
mysql_query('set names utf8');
$resource = mysql_query($sql);
$result = mysql_fetch_assoc($resource);
if(empty($result)){
echo 1;
} else {
echo 2;
}
?>
```

## 3.GET缓存

什么是缓存： 浏览器的请求需要从服务器获得许多 css、img、js 等相关的文件，如果每次请求都把相关的资源文件加载一次，对 带宽、服务器资源、用户等待时间 都有严重的损耗。如果浏览器将css、img、js等文件在第一次请求成功后就保存在本机上，以后的每次请求就在本机获得相关的资源文件，那么就可以明显地加快用户的访问速度，同时可以节省各种资源(带宽、服务器资源、用户等待时间)。

案例： ndex.html页面中创建一个按钮，点击该按钮时发送ajax请求，得到后端php程序返回的当前时间戳并显示。
index.html页面：

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
<script src="../xhr.js"></script>
</head>
<body>
<input type="button" value="获取时间戳" id="btn">
<script>
document.getElementById('btn').onclick = function () {
var xhr = getXhr();
xhr.onreadystatechange = function () {
if(xhr.readyState == 4) {
alert(xhr.responseText);
}
}
xhr.open('get','index.php?n=' + Math.random());
xhr.send(null)
}
//为什么后缀时间是不一样的在没有加随机时间的情况下
</script>
</body>
</html>
```

index.php页面：

```
<?php
// header('cache-contronller:no-cache');
// header('Pragam:no_cache');
// header('Expires:-1')
echo time();
?>
```

存在的问题：一直拿到的时间戳是一个值，说明没有访问后台页面，一直拿的都是缓存
在ie中会有缓存问题，谷歌也有，但是谷歌自身会生成一个时间戳，正在解决这个问题

### 解决方法：

1) 前端方案: 在open准备ajax请求时，为请求的地址增加随机后缀。相当于每次请求都是新的地址

2) 后端方案: 后端程序设置不允许缓存的头信息，php程序固定使用如下3句即可。

header('cache-controller:no-cache');

header('Pragam:no-cache');

header('Expires:-1');

# 6.同步和异步

同步：执行顺序是一步一步执行的

异步：甲在完成一系列的工作时，自己主动完成工作，将一些分支工作交给乙完成，甲此时一直在完成自己的工作，并等待乙完成的结果，乙完成结果后将结果返回给甲
open方法的参数三，来控制同步异步true:异步，默认false:同步

## XMLHttpRequest2.0新特性

### 1.timeout和ontimeout

timeout:请求超时设置

xhr.timeout = 3000;设置等待时长为3秒，3秒内返回数据，则一切正常，如果3秒内没有返回数据，则会触发ontimeout事件

xhr.ontimeout = function ( ) {   alert('请求超时')；}需要一起使用

### 2.FormDate表单对象

使用FormData对象获取所有表单的数据，并发送Ajax请求

1.表单设置：

1）form标签需要设置id，方便获取form标签的DOM对象，

2）每个域都需要设置name值

3）提交按钮必须用button，因为submit有跳转的功能4）必须使用post的提交方式

2.获取表单中的数据 并发送Ajax请求

注：要使用Form表单的DOM对象$('main')[0]send方法中要将fd对象作为参数传入

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
<script src="../xhr.js"></script>
</head>
<body>
<form id="main">
用户名:<input type="text" name="username"><br>
密 码:<input type="text" name="password"><br>
<input type="button" id="btn" value="提交">
</form>
<script>
document.getElementById('btn').onclick = function () {
//获取form表单对象DOM对象
var fm = document.getElementById('main');
var fd = new FormData(fm);
//发送Ajax请求
var xhr = getXhr();
xhr.onreadystatechange = function () {
if (xhr.readyState == 4 && xhr.status == 200) {
alert(xhr.responseText);
}
}
xhr.open('post', 'formdate.php');
xhr.send(fd)
}
//返回的数据是一个关联数组
</script>
</body>
</html>
```

# 7.Ajax文件上传进度条

核心：Ajax方式上传文件必须使用FormData对象

1.表单提交的按钮必须是button

2.根据id获取form表单对象----DOM对象

需要在PHP文件中的php.ini文件配置中修改可以上床最大文件的最大值phpStudy--php--php-5.4.45--php.ini
第790行以及661行修改后需要重启Apache
上传进度条：

xhr对象中有一个子对象，在uoload对象中 与一个onprogress事件，该事件中有两个属 性 loaded(已上传文件)  total(文件总大小)

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
<script src="../xhr.js"></script>
</head>
<body>
<div id="outer" style="width:300px; height:30px; border:1px solid red">
<div id="inner" style="width:0px; height:30px; background:red"></div>
</div>
<form id="main">
<input type="file" name="f">
<input type="button" value="上传" id="btn">
</form>
<script>
//设置上传按钮的点击事件
document.getElementById('btn').onclick = function () {
var fm = document.getElementById('main');
//实例化FormDate对象
var fd = new FormData(fm);
var xhr = getXhr();

var percent = 0;
var inner_obj = document.getElementById('inner');
xhr.upload.onprogress = function (evt) {
percent = (evt.loaded / evt.total) . toFixed(2);
inner_obj.style.width = percent * 300 + 'px';
}
xhr.onreadystatechange = function () {
if(xhr.readyState == 4 && xhr.status == 200) {
// alert(xhr.responseText);
if(xhr.responseText == 1) {
alert('文件上传成功');
} else {
alert('文件上传失败');
}
}
}
xhr.open('post','upload.php');
xhr.send(fd);
}
</script>
</body>
</html>
```

# 8.JSON

JavaScript Object Notation 是一种轻量级数据交互格式数据交互：每一种语言的编码都不一样，它们之间互不认识。但是现在的情况是不同的语言开发出的系统也需要进行数据交互，这时候就需要一种大家都认识的语言或者技术来实现

## 1.JSON数据的声明和使用

### 1.声明： 

var json_obj = {"key1":"value1", "key2":"value2", ...};

key: ==双引号包含的字符串==

value: 数据--数值型、字符串、数组、json对象

```
var json3 = [
    {"name":"郭麒麟", "age":20},
    {"name":"于思洋", "age":8},
    {"name":"沈腾", "age":40}
];
alert(json3[1].name); //于思洋
```

### 2.JSON的本质

JSON是JS对象的字符串表示法，他使用文本表示一个JS对象的字符串表示法，它使用文本表示一个JS对象的信息，本质是一个字符串

var obj = {a:"hello", b:"world"}; // js对象

var obj = {"a":"hello", "b":"world"}; // ==json格式的js对象==，也可以叫json对象 (JSON才是真正的JSON对象)

var str = '{"a":"hello", "b":"world"}'; // json，也叫json格式的字符串 

==必须外层单引号，内存双引号==

var str = "{'a':'Hello', 'b':'world'}"; //错误，不能这样写，会影响到其他程序的执行

### 3.PHP数组转JSON格式的字符串（json_encode($arr);）

```
$list = [
    ["name"=>"马云", "money"=>2500],
    ["name"=>"许家印", "money"=>3000],
    ["name"=>"马化腾", "money"=>2300]
];
//二维数组会转为  数组，内部每个单元都是json，整体是个字符串
echo json_encode($list);
//'[
//  {"name":"\u9a6c\u4e91","money":2500},
//  {"name":"\u8bb8\u5bb6\u5370","money":3000},
//  {"name":"\u9a6c\u5316\u817e","money":2300}
// ]'
```

### 4.JSON字符串转为JSON对象

前端的Ajax请求，最后接收的都是字符串responseText，以字符串的形式接收后端返回的数据

json格式的字符串转为json对象: JSON.parse(json_str);

参数: json格式的字符串

```
var str1 = '["zs", "ls", "ww"]';
console.log(JSON.parse(str1)); //数组 ["zs", "ls", "ww"]
```

# 9.JQuery提供的Ajax方法

## 1.$.get

$.get(var1, var2, var3, var4);

参数：

1: 请求的后端程序的地址参数

2: 要发送到后端程序的数据，json对象/js对象（推荐）或者 字符串参数

3: 当readyState==4时的触发函数，该函数中有一个参数，就是后端程序返回的数据参数

4: 设置返回数据的类型: text(默认) json xml

使用jquery提供的ajax方法，就是为了简化开发。

```
$.get('getData.php', {"goods_id":10101, "_":Math.random()}, function(msg){
    alert(msg);
}, 'json');
```

```
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function(){
    if(xhr.readyState == 4){
        msg = xhr.responseText;
        msg = JSON.parse(msg);
        alert(msg);
    }
}
xhr.open('get', 'getData.php?goods_id=10101&_='+Math.random());
xhr.send(null);
```

## 2.$.post

$.post 函数的用法和  get一模一样，只是发送请求方式变为post

$.post(var1, var2, var3 , var4); //最标准的写法

参数1: 请求的后台程序的地址

参数2: 要发送到后台程序的数据，json对象/js对象（推荐） 或者 字符串

参数3: 当readyState=4时的触发函数，该函数中有一个参数，就是后台程序返回的数据

参数4: 设置返回数据的类型: text(默认)  json  xml

## 3.$.ajax方法

$.ajax使用JS对象配置ajax请求

$.ajax(obj)

必须配置项：

url：要请求的后台程序地址

data：要发送到后台程序的数据（建议使用json/js对象，也可以是字符串）

type：请求类型post和get两种（put、delete）

dataType：返回值类型text（默认）、json、xml、jsonp（跨域使用）

success：成功完成ajax触发的时间，回调函数，其参数是后端程序的返回数据

其他配置项：

cache：是否进行缓存（true/false）如果设置type为get，一般设置该项为false（不缓存）

async：同步、异步设置，true(异步，为默认值)，false为同步

timeout：超时设置，当超过****ms后，仍然没有接收到后端返回的数据，在结束本次请求---------进入error方法中

error：请求失败时的回调函数

          error：function（va1，var2，var3）{}

参数一：xhr对象

参数二：错误信息（通常有：null，timeout，error，not  modified，parseerror（数据类型转换错误））

参数三：异常对象（可选）

complete：Ajax完成时的回调函数

beforeSend：发送Ajax请求前执行的函数（可用来进行正则验证）

beforeSend() ---> success/error ---> complete

contentType：头信息设置，使用FormDate对象时设置该值为false，其他情况会自动设置，不需要手动设置

processData：处理数据方式，使用FormDate对象时设置该值为false，其他情况会自动设置，不需要手动设置

contentType和processData只有在使用FormData对象时，设置为false，其余情况均不用设置






































































