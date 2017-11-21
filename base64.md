# BASE64

> 解释： Base64 是常见的用于传输8bit字节码的编码方式之一，Base64是一种基于64个可打印字符来表示二进制的方法。

## 1. base64  ——— 图片

好处： 节省了一个http请求

坏处： 浏览器不会缓存图片

1. 使用canvas
2. 使用H5 api file reader

~~~javascript
function convertImgToBase64(url, callback, outputFormat){
    var canvas = document.createElement('CANVAS'),
        ctx = canvas.getContext('2d'),
        img = new Image;
    img.crossOrigin = 'Anonymous';
    img.onload = function(){
        canvas.height = img.height;
        canvas.width = img.width;
        ctx.drawImage(img,0,0);
        var dataURL = canvas.toDataURL(outputFormat || 'image/png');
        callback.call(this, dataURL);
        canvas = null; 
    };
    img.src = url;
}
~~~

~~~javascript
convertImgToBase64('http://bit.ly/18g0VNp', function(base64Img){
  	console.log(base64Img);
});
~~~



## 2. base64 — — — blob

~~~javascript
function blobToDataURL(blob, callback) {
    var a = new FileReader();
    a.onload = function (e) { callback(e.target.result); }
    a.readAsDataURL(blob);
}
~~~

~~~javascript
blobToDataURL(blob, function (dataurl) {
	console.log(dataurl);
})
~~~

~~~javascript
function dataURLtoBlob(dataurl) {
  	var arr = dataurl.split(','), 
        mime = arr[0].match(/:(.*?);/)[1],
        // window.atob 用来解码一个已经被base-64编码过的数据
      	bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
  	while (n--) {
    	u8arr[n] = bstr.charCodeAt(n);
  	}
  	return new Blob([u8arr], { type: mime });
}
~~~

~~~javascript
var blob = dataURLtoBlob('data:text/plain;base64,YWFhYWFhYQ==');
~~~

