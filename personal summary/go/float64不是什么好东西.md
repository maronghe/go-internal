###### **Float64不是什么好东西**

#### 背景：

​	近期在做一些数据计算相关的操作，需求是保留小数点后2位，经过一个计算公式以后，会得到2位数的计算结果。根据这个问题，引入了一些问题。

Case 1 ：

​	既然是小数点后有值，就能用整形int去表示值。所以很明显是float类型，在golang语言中，float64是较为常见的类型，但它并不是什么好东西，当前后端要传递float64时，前端的数据类型不能将float64的数据完全存储的下，只能通过转换成字符串的形式传递给前端做展示，当前端给后端传递‘float64’类型的值时，也需要将其包装成字符串类型，后端同学再转换成float64，无意间增加传参的复杂性。

Case 2： 

```go
var x float64 = 2.26 
fmt.Println(x) // 2.26
y := x * 100 
fmt.Println(y) // 225.99999999999997 ??? 为啥不是226呢
fmt.Println(int(y)) // 225 ??? 为啥不是226呢

```

就得好好查查这个问题

#### 解决方案：

Case 1 Solution：

​	对于Web开发时，可以参考B站的七米qimi的方案进行解决，具体视频可以访问后面的链接，其具体思路是通过Golang中的struct tag 进行处理，来兼容前端不能存储float64的情况。[Go实战技巧--你不知道的JSON操作技巧](https://www.bilibili.com/video/BV1AV411S74X)



Case 2 Solution：

​	根据[Golang的官方论坛](https://groups.google.com/g/golang-nuts)中的一个[int16 and float64 Multiplication](https://groups.google.com/g/golang-nuts/c/yAUfWt1Jz_8)的邮件咨询了这个问题给Google的开发者，[Ian Lance Taylor](https://github.com/ianlancetaylor)回答了float64的工作原理，想了解的小伙伴可以参考 [What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)这篇文章。

​	但话说回来，目前的需求很简单，希望的是保留小数点后两位，然后乘以整形的数得到整数即可。那么就可以使用以下的float32的解决方案。思路是将float64强转成float32，再乘以整形的数字即可。到这里你可能有疑问， float64强转float32不会造成数据丢失么？答案是：在某种情况上，是会丢失，因为float64是64位也就是8个字节，float32是32位四个字节。但是，对于仅保留小数点后两位情况的case，float32是能力是完全可以cover的住的。举个例子，若你接口的请求量很低，干嘛要用集群部署呢处？反而是不是增加了系统的复杂性？浪费资源？所以，**做一件事要考虑的东西，一定要结合需求。**

```go
var x float32 = 2.26 
fmt.Println(x) // 2.26
y := x * 100 
fmt.Println(y) // 226
fmt.Println(int(y)) // 226 
```



