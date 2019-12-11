##音频流的处理
###1、html接受数据流
####1、文件框获取音频流并播放
```
<input  type="file"/>
<script>
  function analyticBuffer(buffer){
    //将ArrayBuffer异步转换为一个AudioBuffer类型
    audioCtx.decodeAudioData(buffer, (myArrayBuffer) => {
          let source  = audioCtx.createBufferSource();
            source.buffer = myArrayBuffer;
            source.connect(audioCtx.destination);
            source.start();
    });
   }


    //1.通过input=file 获取的音频文件
    let fileInput = document.querySelector('input'),
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    fileInput.onchange = function(ev){
        let file = ev.target.files[0],
            fr = new FileReader();
        fr.readAsArrayBuffer(file);
        fr.onload = function(data){
            //result是一个arraybuffer类型二进制数据
            let result = data.target.result;
            //解析数据
            var myArrayBuffer = analyticBuffer(result);
        source = audioCtx.createBufferSource();
    	//myArrayBuffer是一个AudioBuffer
    	source.buffer = myArrayBuffer;
   	source.loop = true; //循环播放
    	source.connect(audioCtx.destination);
    	source.start(); //开始播放音频源
	};
    };
</script>
```
####2、获取音频流方式
```
    //1.通过input=file 获取的音频文件
    let fileInput = document.querySelector('input'),
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    fileInput.onchange = function(ev){
        let file = ev.target.files[0],
            fr = new FileReader();
        fr.readAsArrayBuffer(file);
        fr.onload = function(data){
            //result是一个arraybuffer类型二进制数据
            let result = data.target.result;
            //解析数据
            analyticBuffer(result);
        };
    };
    //2.通过XHR获取音频数据(注意需要返回arraybuffer类型)
    let request = new XMLHttpRequest();
    request.open('GET', 'xxx.mp3', true);
    //指定文件返回数据类型
    request.responseType = 'arraybuffer';
    //请求成功时执行
    request.onload = function() {
        //这是一个arraybuffer
        var buffer = request.response;
        //解析数据
        analyticBuffer(buffer )
    }
    request.send();

    //解析二进制数据
    function analyticBuffer(buffer){
        //将ArrayBuffer异步转换为一个AudioBuffer类型
        audioCtx.decodeAudioData(buffer, (myArrayBuffer) => {
            let source  = audioCtx.createBufferSource();
            source.buffer = myArrayBuffer;
            source.connect(audioCtx.destination);
            source.start();
        });
    }
    //3.自己创造一个AudioBuffer
    //采样率sample/s
    let sampleRate = audioCtx.sampleRate,
        //帧数，音频时间 = frameCount / sampleRate
        frameCount = audioCtx.sampleRate * 2.0,
        //创建一个两通道的音频数据,这是一个没有声音的音频数据
        myArrayBuffer = audioCtx.createBuffer(2, frameCount , sampleRate);
    //随机填充白噪音
    //两个通道循环2次
    for (var channel = 0; channel < 2; channel++) {
        //获取每个通道的array数据
        var nowBuffering = myArrayBuffer.getChannelData(channel);
        for (let i = 0; i < frameCount; i++) {
            //对每一帧填充数据
            nowBuffering[i] = Math.random() * 2 - 1;
        }
    }
```
####3、AudioBuffer的属性和方法

AudioBuffer的方法在我们直播的时候需要用到，在后面的AudioNode(音频处理模块)中也会出现AudioBuffer数据，我们需要它是获取和传输数据
```
    let myArrayBuffer = audioCtx.createBuffer(2, 4096, sampleRate);
    myArrayBuffer.sampleRate //采样数
    myArrayBuffer.length //采样帧率 也就是4096
    myArrayBuffer.duration //时长
    myArrayBuffer.numberOfChannels //通道数
    //返回x通道的Float32Array类型的数据，x表示是哪个通道0或1
    myArrayBuffer.getChannelData(x) 
    //将myArrayBuffer第x通道的数据复制到anotherArray中，y表示数据复制开始的偏移量
    let anotherArray = new Float32Array;
    myArrayBuffer.copyFromChannel(anotherArray,x,y);
    //将anotherArray数据复制到myArrayBuffer的X通道中，y偏移量
    let anotherArray = new Float32Array;
    myArrayBuffer.copyToChannel(anotherArray,x,y);
    //关于copyToChannel，copyFromChannel，getChannelData在下一章看见例子就明白了
```

###2、input文件转化成流并播放
```
<input  type="file"/>
<script>
let audioCtx = new (window.AudioContext || window.webkitAudioContext)(),
        source = audioCtx.createBufferSource();
    //myArrayBuffer是一个AudioBuffer
    source.buffer = myArrayBuffer;
    source.loop = true; //循环播放
    source.connect(audioCtx.destination);
    source.start(); //开始播放音频源


    //1.通过input=file 获取的音频文件
    let fileInput = document.querySelector('input'),
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    fileInput.onchange = function(ev){
        let file = ev.target.files[0],
            fr = new FileReader();
        fr.readAsArrayBuffer(file);
        fr.onload = function(data){
            //result是一个arraybuffer类型二进制数据
            let result = data.target.result;
            //解析数据
            analyticBuffer(result);
        };
    };
</script>
```
##AudioNode
AudioNode：是一个处理音频的通用模块, 比如一个音频源 (e.g. 一个 HTML <audio> or <video> 元素), 一个音频地址或者一个中间处理模块 (e.g. 一个过滤器如 BiquadFilterNode, 或一个音量控制器如 GainNode).一个AudioNode 既有输入也有输出。输入与输出都有一定数量的通道。只有一个输出而没有输入的 AudioNode 叫做音频源(MDN的解释)。
```
let audioCtx = new window.AudioContext,
    //频率及时间域分析器，声音可视化就用这个
    analyser = audioCtx.createAnalyser(),
    //音量变更模块
    gainNode = audioCtx.createGain(),
    //波形控制器模块
    distortion = audioCtx.createWaveShaper(),
    //低频滤波器模块
    biquadFilter = audioCtx.createBiquadFilter(),
    //创建源
    source = audioCtx.createMediaStreamSource(stream);
    //通过connect方法从音频源连接到AudioNode处理模块，再连接到输出设备，
    //当我们修改前面的模块时，就会影响到后面AudioNode，以及我们最终听见的声音
    source.connect(analyser);
    analyser.connect(distortion);
    distortion.connect(biquadFilter);
    biquadFilter.connect(convolver);
    convolver.connect(gainNode);
    gainNode.connect(audioCtx.destination);
```
###1、createAnalyser
```
let analyser = audioCtx.createAnalyser();
    //频域的FFT大小，默认是2048
    analyser.fftSize;
    //fftSize的一半
    analyser.frequencyBinCount;
    //快速傅立叶变化的最大范围的双精度浮点数
    analyser.maxDecibels;
    //最后一个分析帧的平均常数
    analyser.smoothingTimeConstant;
    //将当前频域数据拷贝进Float32Array数组
    analyser.getFloatFrequencyData()
    //将当前频域数据拷贝进Uint8Array数组
    analyser.getByteFrequencyData()
    将当前波形，或者时域数据拷贝进Float32Array数组
    analyser.getFloatTimeDomainData()
    //将当前波形，或者时域数据拷贝进 Uint8Array数组
    analyser.getByteTimeDomainData()
```
###2、createGain
```
let gainNode = audioCtx.createGain();
//修改value的大小，改变输出大小，默认是1,0表示静音
gainNode.gain.value = 1
```
###3、createScriptProcessor
缓冲区音频处理模块，这个是我们做直播的核心模块，没有这个模块就做不到音频流的转发，音频数据的延迟在除开网络的影响下，这个模块也占一部分，当然要看自己的配置。AudioBuffer介绍
```
/*
    第一个参数表示每一帧缓存的数据大小，可以是256, 512, 1024, 2048, 4096, 8192, 16384,
      值越小一帧的数据就越小，声音就越短，onaudioprocess 触发就越频繁。
      4096的数据大小大概是0.085s，就是说每过0.085s就触发一次onaudioprocess，
      如果我要把这一帧的数据发送到其他地方，那这个延迟最少就是0.085s，
      当然还有考虑发送过去的电脑处理能力，一般1024以上的数字，如果有变态需求的256也是可以考虑的
    第二，三个参数表示输入帧，和输出帧的通道数。这里表示2通道的输入和输出，当然我也可以采集1，4,5等通道
*/
let recorder = audioCtx.createScriptProcessor(4096, 2, 2);
/*
  缓存区触发事件，连接了createScriptProcessor这个AudioNode就需要在onaudioprocess中，
  把输入帧的数据，连接到输出帧，扬声器才有声音
*/
recorder.onaudioprocess = function(e){
    let inputBuffer = e.inputBuffer, //输入帧数据，AudioBuffer类型
        outputBuffer = e.outputBuffer; //输出帧数据， AudioBuffer类型
     //第一种方式
     //将inputBuffer第0个通道的数据，复制到outputBuffer的第0个通道，偏移0个字节
     outputBuffer.copyToChannel(inputBuffer.getChannelData(0), 0, 0);
     //将inputBuffer第1个通道的数据，复制到outputBuffer的第1个通道，偏移0个字节
     outputBuffer.copyToChannel(inputBuffer.getChannelData(1), 1, 0);
     //第二中方式用循环
     for (var channel = 0; channel < outputBuffer.numberOfChannels; channel++) {
          let inputData = inputBuffer.getChannelData(channel),
              outputData = outputBuffer.getChannelData(channel);
           for (var sample = 0; sample < inputBuffer.length; sample++) {
               outputData[sample] = inputData[sample];      
           }
        }
}
```
###4、完整示例
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title></title>
    <link rel="stylesheet" href="">
</head>
<body>
    <canvas width="500px" height="500px">
        
    </canvas>
    <input type="file" name="" value="" placeholder="">
    <button type="button" class="add">音量+</button>
    <button type="button" class="lost">音量-</button>
</body>
<script type="text/javascript" charset="utf-8">
    let fileInput = document.querySelector('input'),
        add = document.querySelector('.add'), //音量+
        lost = document.querySelector('.lost'), //音量-
        audioCtx = new window.AudioContext, //创建环境
        analyser = audioCtx.createAnalyser(), //analyser分析器
        gainNode = audioCtx.createGain(), //控制音量大小
        recorder = audioCtx.createScriptProcessor(4096, 2, 2), //缓冲区音频处理模块
        canvas = document.querySelector('canvas'),
        canvasCtx = canvas.getContext('2d');
    fileInput.onchange = function(ev){
        let file = ev.target.files[0],
        fr = new FileReader();
        fr.readAsArrayBuffer(file);
        fr.onload = function(data){
            let result = data.target.result;
            //解码ArrayBuffer
            audioCtx.decodeAudioData(result, getBuffer);
        };
    };

    //修改音量大小
    add.onclick = function(){
        gainNode.gain.value += 0.1;
    };
    lost.onclick = function(){
        gainNode.gain.value -= 0.1;
    }

    function getBuffer(audioBuffer){
        //创建对象，用过AudioBuffer对象来播放音频数据
        let source  = audioCtx.createBufferSource();
        source.buffer = audioBuffer;
        //将source与analyser分析器连接
        source.connect(analyser);
        //将analyser与gainNode分析器连接
        analyser.connect(gainNode);
        //音量控制器与输出设备链接
        gainNode.connect(recorder);
        recorder.connect(audioCtx.destination);
        //播放
        source.start(0); 
        draw(analyser);
        //音频采集
        recorder.onaudioprocess = function (e) {
            /*输入流，必须要链接到输出流，audioCtx.destination才能有输出*/
              let inputBuffer = e.inputBuffer, outputBuffer = e.outputBuffer;
                outputBuffer.copyToChannel(inputBuffer.getChannelData(0), 0, 0);
                outputBuffer.copyToChannel(inputBuffer.getChannelData(1), 1, 0);
        };
    }
    let WIDTH = 500, HEIGHT = 500;
    //绘制波形图
    function draw() {
        requestAnimationFrame(draw);
        //保存频率数据
      let dataArray = new Uint8Array(analyser.fftSize),
          bufferLength = analyser.fftSize;
      //获取频域的输出信息 
      analyser.getByteTimeDomainData(dataArray);
      canvasCtx.fillStyle = 'rgb(200, 200, 200)';
      canvasCtx.fillRect(0, 0, 500, 500);

      canvasCtx.lineWidth = 2;
      canvasCtx.strokeStyle = 'rgb(0, 0, 0)';

      canvasCtx.beginPath();

      var sliceWidth = WIDTH * 1.0 / bufferLength;
      var x = 0;

      for(var i = 0; i < bufferLength; i++) {
   
        var v = dataArray[i] / 128.0;
        var y = v * HEIGHT/2;

        if(i === 0) {
          canvasCtx.moveTo(x, y);
        } else {
          canvasCtx.lineTo(x, y);
        }

        x += sliceWidth;
      }

      canvasCtx.lineTo(canvas.width, canvas.height/2);
      canvasCtx.stroke();
    };
</script>
</html>
```
消除回声
https://blog.csdn.net/u011249920/article/details/51374816