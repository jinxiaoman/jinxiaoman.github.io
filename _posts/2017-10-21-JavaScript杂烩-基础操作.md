---
layout:     post
title:      JavaScript杂烩  基础操作
subtitle:   一些常用的基础操作
date:       2017-10-21
author:     jinxm
header-img:  img/post-bg-js-version.jpg
catalog: true
tags:
    - JavaScript
    - 杂烩
---

# 字符串
> 字符串转换是最基础的要求和工作，你可以将任何类型的数据转换为字符串

#### 字符串转换

```
var num = 20;
var myStr = num.toString()  //'20'
var myStr = num.String(num)   //'20'
var myStr = '' +num;          //'20'
```

#### 字符串分割
> 把一个字符串分割为多个字符串

```
var myStr = "I,Love,You,Do,you,love,me";
var substrArray = myStr .split(","); // ["I", "Love", "You", "Do", "you", "love", "me"];
var arrayLimited = myStr .split(",", 3); // ["I", "Love", "You"];
split()的第二个参数，表示返回的字符串数组的最大长度
```

#### 获取长度
```
var myStr = 'I,Love,You'
var myStrLength = myStr.length;
```

#### 查询字符串

第一个函数：indexOf()，它从字符串的开头开始查找，找到返回对应坐标，找不到返回-1。如下：

```
var myStr = "I,Love,you,Do,you,love,me";
var index = myStr.indexOf("you"); // 7 ,基于0开始,找不到返回-1
第二个函数：lastIndexOf()，它从字符串的末尾开始查找，找到返回对应坐标，找不到返回-1。如下：

var myStr = "I,Love,you,Do,you,love,me";
var index = myStr.lastIndexOf("you"); // 14
以上两个函数同样接收第二个可选的参数，表示开始查找的位置。
```

#### 字符串替换

单单查到字符串应该还不会停止，一般题目都还经常会遇到让你查到并替换为你自己的字符串，例如：

```
var myStr = "I,love,you,Do,you,love,me";
var replacedStr = myStr.replace("love","hate");//"I,hate,you,Do,you,love,me"
默认只替换第一次查找到的，想要全局替换，需要置上正则全局标识，如：

var myStr = "I,love,you,Do,you,love,me";
var replacedStr = myStr.replace(/love/g,"hate");//"I,hate,you,Do,you,hate,me"
```

#### 字符串连接

 字符串连接操作可以简单到用一个加法运算符搞定，如：

 ```
 var str1 = "I,love,you!";
 var str2 = "Do,you,love,me?";
 var str = str1 + str2 + "Yes!";//"I,love,you!Do,you,love,me?Yes!"
 同样，JavaScript也自带了相关的函数，如：

 var str1 = "I,love,you!";
 var str2 = "Do,you,love,me?";
 var str = str1.concat(str2);//"I,love,you!Do,you,love,me?"
 其中concat()函数可以有多个参数，传递多个字符串，拼接多个字符串。
 ```

#### 字符串切割和提取
```
第一种，使用slice():
var myStr = "I,love,you,Do,you,love,me";
var subStr = myStr.slice(1,5);//",lov"
第二种，使用substring():

var myStr = "I,love,you,Do,you,love,me";
var subStr = myStr.substring(1,5); //",lov"
第三种，使用substr():


var myStr = "I,love,you,Do,you,love,me";
var subStr = myStr.substr(1,5); //",love"  -1表示从倒数第一开始
与第一种和第二种不同的是，substr()第二个参数代表截取的字符串最大长度，如上结果所示
```

#### 字符串大小写转换

```
常用的转换为大写或者小写字符串函数，如下：

var myStr = "I,love,you,Do,you,love,me";
var lowCaseStr = myStr.toLowerCase();//"i,love,you,do,you,love,me";
var upCaseStr = myStr.toUpperCase();//"I,LOVE,YOU,DO,YOU,LOVE,ME"
```

#### 字符串比较

```
比较两个字符串，比较是规则是按照字母表顺序比较的，如：
var myStr = "chicken";
var myStrTwo = "egg";
var first = myStr.localeCompare(myStrTwo); // -1
first = myStr.localeCompare("chicken"); // 0
first = myStr.localeCompare("apple"); // 1

```