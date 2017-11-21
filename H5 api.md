# H5 api

## 1. getUserMedia 

(调用的百度语音接口，目前仍不支持跨域)

### 1. 录音，播放，下载

>张鑫旭博客
>
>http://www.zhangxinxu.com/wordpress/2017/01/html5-speech-recognition-synthesis-api/

*https://yq.aliyun.com/ask/23821*

*http://javascript.ruanyifeng.com/htmlapi/webrtc.html*

~~~html
		<span style="white-space:pre"></span>
		<audio controls autoplay></audio>  
        <input type="button" value="开始录音" onclick="startRecording()"/>  
        <input type="button" value="获取录音" onclick="obtainRecord()"/>  
        <input type="button" value="停止录音" onclick="stopRecord()"/>  
        <input type="button" value="播放录音" onclick="playRecord()"/>  
~~~

~~~javascript
(function(window) {
    //兼容  
    window.URL = window.URL || window.webkitURL;
    navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia;

    var HZRecorder = function(stream, config) {
        config = config || {};
        config.sampleBits = config.sampleBits || 8; //采样数位 8, 16  
        config.sampleRate = config.sampleRate || (44100 / 6); //采样率(1/6 44100)  

        //创建一个音频环境对象  
        audioContext = window.AudioContext || window.webkitAudioContext;
        var context = new audioContext();

        //将声音输入这个对像  
        var audioInput = context.createMediaStreamSource(stream);

        //设置音量节点  
        var volume = context.createGain();
        audioInput.connect(volume);

        //创建缓存，用来缓存声音  
        var bufferSize = 4096;

        // 创建声音的缓存节点，createScriptProcessor方法的  
        // 第二个和第三个参数指的是输入和输出都是双声道。  
        var recorder = context.createScriptProcessor(bufferSize, 2, 2);

        var audioData = {
            size: 0 //录音文件长度  
            ,
            buffer: [] //录音缓存  
            ,
            inputSampleRate: context.sampleRate //输入采样率  
            ,
            inputSampleBits: 16 //输入采样数位 8, 16  
            ,
            outputSampleRate: config.sampleRate //输出采样率  
            ,
            oututSampleBits: config.sampleBits //输出采样数位 8, 16  
            ,
            input: function(data) {
                this.buffer.push(new Float32Array(data));
                this.size += data.length;
            },
            compress: function() { //合并压缩  
                //合并  
                var data = new Float32Array(this.size);
                var offset = 0;
                for (var i = 0; i < this.buffer.length; i++) {
                    data.set(this.buffer[i], offset);
                    offset += this.buffer[i].length;
                }
                //压缩  
                var compression = parseInt(this.inputSampleRate / this.outputSampleRate);
                var length = data.length / compression;
                var result = new Float32Array(length);
                var index = 0,
                j = 0;
                while (index < length) {
                    result[index] = data[j];
                    j += compression;
                    index++;
                }
                return result;
            },
            encodeWAV: function() {
                var sampleRate = Math.min(this.inputSampleRate, this.outputSampleRate);
                var sampleBits = Math.min(this.inputSampleBits, this.oututSampleBits);
                var bytes = this.compress();
                var dataLength = bytes.length * (sampleBits / 8);
                var buffer = new ArrayBuffer(44 + dataLength);
                var data = new DataView(buffer);

                var channelCount = 1; //单声道  
                var offset = 0;

                var writeString = function(str) {
                    for (var i = 0; i < str.length; i++) {
                        data.setUint8(offset + i, str.charCodeAt(i));
                    }
                };

                // 资源交换文件标识符   
                writeString('RIFF');
                offset += 4;
                // 下个地址开始到文件尾总字节数,即文件大小-8   
                data.setUint32(offset, 36 + dataLength, true);
                offset += 4;
                // WAV文件标志  
                writeString('WAVE');
                offset += 4;
                // 波形格式标志   
                writeString('fmt ');
                offset += 4;
                // 过滤字节,一般为 0x10 = 16   
                data.setUint32(offset, 16, true);
                offset += 4;
                // 格式类别 (PCM形式采样数据)   
                data.setUint16(offset, 1, true);
                offset += 2;
                // 通道数   
                data.setUint16(offset, channelCount, true);
                offset += 2;
                // 采样率,每秒样本数,表示每个通道的播放速度   
                data.setUint32(offset, sampleRate, true);
                offset += 4;
                // 波形数据传输率 (每秒平均字节数) 单声道×每秒数据位数×每样本数据位/8   
                data.setUint32(offset, channelCount * sampleRate * (sampleBits / 8), true);
                offset += 4;
                // 快数据调整数 采样一次占用字节数 单声道×每样本的数据位数/8   
                data.setUint16(offset, channelCount * (sampleBits / 8), true);
                offset += 2;
                // 每样本数据位数   
                data.setUint16(offset, sampleBits, true);
                offset += 2;
                // 数据标识符   
                writeString('data');
                offset += 4;
                // 采样数据总数,即数据总大小-44   
                data.setUint32(offset, dataLength, true);
                offset += 4;
                // 写入采样数据   
                if (sampleBits === 8) {
                    for (var i = 0; i < bytes.length; i++, offset++) {
                        var s = Math.max( - 1, Math.min(1, bytes[i]));
                        var val = s < 0 ? s * 0x8000: s * 0x7FFF;
                        val = parseInt(255 / (65535 / (val + 32768)));
                        data.setInt8(offset, val, true);
                    }
                } else {
                    for (var i = 0; i < bytes.length; i++, offset += 2) {
                        var s = Math.max( - 1, Math.min(1, bytes[i]));
                        data.setInt16(offset, s < 0 ? s * 0x8000: s * 0x7FFF, true);
                    }
                }

                return new Blob([data], {
                    type: 'audio/wav'
                });
            }
        };

        //开始录音  
        this.start = function() {
            audioInput.connect(recorder);
            recorder.connect(context.destination);
        };

        //停止  
        this.stop = function() {
            recorder.disconnect();
        };

        //获取音频文件  
        this.getBlob = function() {
            this.stop();
            return audioData.encodeWAV();
        };

        //回放  
        this.play = function(audio) {
            audio.src = window.URL.createObjectURL(this.getBlob());
        };

        //上传  
        this.upload = function(url, callback) {
            var fd = new FormData();
            fd.append('audioData', this.getBlob());
            var xhr = new XMLHttpRequest();
            if (callback) {
                xhr.upload.addEventListener('progress',
                function(e) {
                    callback('uploading', e);
                },
                false);
                xhr.addEventListener('load',
                function(e) {
                    callback('ok', e);
                },
                false);
                xhr.addEventListener('error',
                function(e) {
                    callback('error', e);
                },
                false);
                xhr.addEventListener('abort',
                function(e) {
                    callback('cancel', e);
                },
                false);
            }
            xhr.open('POST', url);
            xhr.send(fd);
        };

        //音频采集  
        recorder.onaudioprocess = function(e) {
            audioData.input(e.inputBuffer.getChannelData(0));
            //record(e.inputBuffer.getChannelData(0));  
        };

    };
    //抛出异常  
    HZRecorder.throwError = function(message) {
        throw new
        function() {
            this.toString = function() {
                return message;
            };
        };
    };
    //是否支持录音  
    HZRecorder.canRecording = (navigator.getUserMedia != null);
    //获取录音机  
    HZRecorder.get = function(callback, config) {
        if (callback) {
            if (navigator.getUserMedia) {
                navigator.getUserMedia({
                    audio: true
                } //只启用音频  
                ,
                function(stream) {
                    var rec = new HZRecorder(stream, config);
                    callback(rec);
                },
                function(error) {
                    switch (error.code || error.name) {
                    case 'PERMISSION_DENIED':
                    case 'PermissionDeniedError':
                        HZRecorder.throwError('用户拒绝提供信息。');
                        break;
                    case 'NOT_SUPPORTED_ERROR':
                    case 'NotSupportedError':
                        HZRecorder.throwError('<a href="http://www.it165.net/edu/ewl/" target="_blank" class="keylink">浏览器</a>不支持硬件设备。');
                        break;
                    case 'MANDATORY_UNSATISFIED_ERROR':
                    case 'MandatoryUnsatisfiedError':
                        HZRecorder.throwError('无法发现指定的硬件设备。');
                        break;
                    default:
                        HZRecorder.throwError('无法打开麦克风。异常信息:' + (error.code || error.name));
                        break;
                    }
                });
            } else {
                HZRecorder.throwErr('当前<a href="http://www.it165.net/edu/ewl/" target="_blank" class="keylink">浏览器</a>不支持录音功能。');
                return;
            }
        }
    };
    window.HZRecorder = HZRecorder;

})(window);
~~~

~~~javascript
var recorder;  
var audio = document.querySelector('audio');  

function startRecording() {  
  HZRecorder.get(function (rec) {  
    recorder = rec;  
    recorder.start();  
  });  
}

function obtainRecord(){  
  var record = recorder.getBlob();  
  debugger;  
};

function stopRecord(){  
  recorder.stop();  
};

function playRecord(){  
  recorder.play(audio);  
};
~~~

### 2. 在线语音合成

~~~javascript
function playUrl(url,options,autoplay = true,download = false){
	let audio = document.createElement('audio');
  	let json = Object.keys(options);
  	let queryString = '';
  	// 获取options key-value
    json.forEach(function(data,index){
      queryString += data + '=' + options[data] + '&';
    })
    url = url + '?' + queryString;
  	audio.src = url;
    audio.autoplay = autoplay;
    if(download){
      let a = document.createElement('a');
      a.href = url;
      a.download = '';
      a.click();
    }
	return audio;
}
~~~

### 3. 在线语音识别

~~~javascript
发送给后台，让后台请求API
~~~



## 2. FileReader

*https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader*

> **FileReader** 对象允许Web应用程序异步读取存储在用户计算机上的文件（或原始数据缓冲区）的内容，使用 [`File`](https://developer.mozilla.org/zh-CN/docs/Web/API/File) 或 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 对象指定要读取的文件或数据。

其中File对象可以是来自用户在一个[``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input)元素上选择文件后返回的[`FileList`](https://developer.mozilla.org/zh-CN/docs/Web/API/FileList)对象,也可以来自拖放操作生成的 [`DataTransfer`](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer)对象,还可以是来自在一个[`HTMLCanvasElement`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement)上执行`mozGetAsFile()方法后返回结果`.

***

`let reader = new FileReader();`

| 方法概述                                     |
| ---------------------------------------- |
|                                          |
| void [abort](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader#abort())(); |
|                                          |
| void [readAsArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader#readAsArrayBuffer())(in Blob blob); |
|                                          |
| void [readAsBinaryString](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader#readAsBinaryString())(in Blob blob); |
|                                          |
| void   [readAsDataURL](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader#readAsDataURL())(in Blob blob); |
|                                          |
| void   [readAsText](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader#readAsText())(in Blob blob, [optional] in DOMString encoding); |

| 状态常量    | 值    | 描述          |
| :------ | ---- | ----------- |
|         |      |             |
| EMPTY   | 0    | 还没有加载任何数据   |
|         |      |             |
| LOADING | 1    | 数据正在被加载     |
|         |      |             |
| DONE    | 2    | 已经完成全部的读取请求 |

| 属性         | 类型             | 描述                  |
| ---------- | -------------- | ------------------- |
|            |                |                     |
| error      | DOMError       | 在读取文件时发生的错误，只读      |
|            |                |                     |
| readyState | unsigned short | 表明FileReader对象的当前状态 |
|            |                |                     |
| result     | jsval          | 读取到的文件内容，只读         |

## 3. URL (试验功能)

> **URL.createObjectURL() **静态方法会创建一个 [`DOMString`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString)，其中包含一个表示参数中给出的对象的URL。这个 URL 的生命周期和创建它的窗口中的 [`document`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document) 绑定。这个新的URL 对象表示指定的 [`File`](https://developer.mozilla.org/zh-CN/docs/Web/API/File) 对象或 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 对象。

**URL.revokeObjectURL()** 释放创建的 DOMString