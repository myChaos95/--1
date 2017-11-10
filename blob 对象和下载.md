# blob 对象 和 下载

## 一. blob 对象

*http://www.cnblogs.com/hhhyaaon/p/5928152.html*

​	*一个blob对象就是一个包含有只读原始数据的类文件对象（不可变）。Blob 对象中的数据并不一定得是 JavaScript 的原生形式。 File接口基于Blob，继承了Blob的功能，并且扩展支持了用户计算机的本地文件*。

​	*一直以来JS并没有好的直接处理二进制的手段，blob的存在允许通过JS操作二进制数据*

​	*Blob 对象可以看做是存放二进制数据的容器，并且可以设置二进制数据的MIME类型*

### 1. 创建Blob

`var blob = new Blob( dataArr: Array<any>, opt: {type: string} );`

* dataArray：数组，包含了要添加到blob对象中的数据，数据可以是多个ArrayBuffer，ArrayBufferView，Blob 或者 DOMString 对象。
* opt： 对象用于设置Blob对象的属性。

~~~javascript
var s = "<div>hello world!</div>";
var blob = new Blob([s],{type: "text/xml"});

var abf = new ArrayBuffer(8);
var blob = new Blob([abf],"text/plain");
~~~

​	`Blob.slice(start: number, end: number, contentType: string);`

* start 开始
* end 结束索引，不包括（end）
* contentType 新 Blob 的 MIME 类型

`canvas.toBlob()`

~~~javascript
let canvas = document.getElementById('canvas');
cavans.toBlob(function(blob){
  console.log(blob);
})
~~~



## 二. 分片上传

​	通过 Blob.slice 方法，可以将大文件分片，轮循向后台提交文件片段

​	上传逻辑如下：

*  获取要上传的 file 对象，根据chunk（每片大小）对文件进行分片
*  通过 post 方法轮循上传每片文件，其中 url 中拼接querystring用于描述当前上传的文件信息，post body 存放本次要上传的片段
*  接口每次返回offset，用于执行下次上传

`详细代码见 http://www.cnblogs.com/hhhyaaon/p/5928152.html `

## 三. 通过url显示图片

img 的 src 属性 和 background 的 url 属性，都可以通过接受地址或者 base64 编码来显示图片，同样，我们也可以把图片转换成Blob对象，生成 *<u>URL</u>* 来显示图片。

## 四. 通过url来下载文件

window.url 对象可以为Blob对象生成一个网络地址，结合a标签的download，实现文件下载

### 1. URL.createObjectURL

会根据传入的参数创建一个指向该参数的URL。

`objectURL = window.URL.createObjectURL(blob || file);`

### 2. URL.revokeObjectURL

每次调用createObjectURL时，都会创建一个新的URL对象，通过本方法可以释放它

`window.URL.revokeObjectURL(objectURL);`

### 3. 实现下载

~~~javascript
function download(fileName,blob){
	if (window.navigator.msSaveOrOpenBlob) {
      navigator.msSaveBlob(blob, fileName);
    } else {
      var link = document.createElement('a');
      link.href = window.URL.createObjectURL(blob);
      link.download = fileName;
      link.click();
      window.URL.revokeObjectURL(link.href);
    }
}
~~~
