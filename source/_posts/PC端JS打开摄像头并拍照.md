---
title: PC端JS打开摄像头并拍照
tags:
  - JS
abbrlink: 73f78475
date: 2018-06-22 09:50:06
---
# PC端JS打开摄像头并拍照


## 主要步骤及要点
1. 打开摄像头主要用到getUserMedia方法，然后将获取到的媒体流置入video标签

2. 截取图片主要用到canvas绘图，使用drawImage方法将video的内容绘至canvas中

3. 将截取的内容上传至服务器，将canvas中的内容转为base64格式上传，后端（PHP）通过file_put_contents将其转为图片

## Html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>JS调用摄像头并拍照</title>
    <style>
        video {  
            border: 1px solid #ccc;  
            display: block;  
            margin: 0 0 20px 0;  
            float:left;  
        }  
        #canvas {  
            margin-top: 20px;  
            border: 1px solid #ccc;  
            display: block;  
        }  
    </style>
</head>
<body>
    <video id="video" width="500" height="400" autoplay></video>
    <canvas id="canvas"></canvas>
    <button id="snap">拍照</button>
    <button id="close">关闭</button>
    <button id="upload">上传</button>
</body>
</html>
```

## javascript 部分


### 1. 打开摄像头

**getUserMedia有新旧版本：**

#### - 旧版本位于`navigator`下面   
```
// 获取媒体方法 
navigator.getMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMeddia || navigator.msGetUserMedia;

if(navigator.getMedia) {
    navigator.getMedia({
        video: true,
        audio: true
    }, function(stream) {
        // 用来获取停止摄像头的方法所在的对象
        // 这个写法其实是兼容了<在旧版本中可以直接通过调用`MediaStream.stop()` 来关闭摄像头，不过在新版之中已废弃。需要使用`MediaStream.getTracks()[index].stop()` 来关闭相应的Track>
        mediaStreamTrack = typeof stream.stop === 'function' ? stream : stream.getTracks()[1]; 

        videoEle.src = (window.URL || window.webkitURL).createObjectURL(stream);
        videoEle.play();
    }, function(err) {
        console.log(err);
    });
}

```
> getUserMedia参数说明
1. 第一个参数中指示需要使用视频（video）或音频（audio）
2. 第二个参数调用成功后的回调,其中带一个参数（MediaStream）。在旧版本中可以直接通过调用`MediaStream.stop()` 来关闭摄像头，不过在新版之中已废弃。需要使用`MediaStream.getTracks()[index].stop()` 来关闭相应的Track   
比如：新版中关闭video则根据第一个参数index=0,使用MediaStream.getTracks()[0].stop()

3. 第三个参数指示调用失败后的回调


#### - 新版本位于navigator.mediaDevices 对象下
```
if(navigator.mediaDevices && navigator.mediaDevices.getUserMeida){
    navigator.mediaDevices.getUserMeida({
        video: true,
        audio: true
    }).then(function(stream) {
        mediaStreamTrack = typeof stream.stop === 'function' ? stream : stream.getTracks()[1];
        videoEle.src = (window.URL || window.webkitURL).createObjectURL(stream);
        videoEle.play();
    }).catch(function(err) {
        console.log(err);
    })
}

```

与旧版类似，不过该方法返回了一个Promise对象，可以使用then和catch表示成功与失败的回调

**另外**，需要注意的是，MediaStream.getTracks() 返回的Tracks数组是按第一个参数倒序排列的

比如现在定义了
```
{
    video: true,
    audio: true
}
```
想关闭摄像头，就需要调用MediaStream.getTracks()[1].stop();

同理，0对应于audio的track


### 2. 摄像头操作

#### 0. 首先初始化画布
```
 var context = canvas.getContext("2d"); 
 var canvesEle = document.getElementById("canves");
 var videoEle = document.getElementById("video");
```

#### 1. 拍照
document.getElementById("snap").addEventListener('click', function(e){
    context.drawImage(videoEle, 0, 0, 500, 400);
})

#### 2. 关闭摄像头
```
// 关闭摄像头
close.addEventListener('click', function() {
    mediaStreamTrack && mediaStreamTrack.stop();
}, false);
```

#### 3. 图像上传获取
```
canvas.toDataURL('image/png')
```

### 完整js
```
<script type="text/javascript" src="jquery.js"></script>
<script type="text/javascript">
    function $(elem) {
        return document.querySelector(elem);
    }

    // 获取媒体方法（旧方法）
    navigator.getMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMeddia || navigator.msGetUserMedia;

    var canvas = $('canvas'),
        context = canvas.getContext('2d'),
        video = $('video'),
        snap = $('#snap'),
        close = $('#close'),
        upload = $('#upload'),
        uploaded = $('#uploaded'),
        mediaStreamTrack;

    // 获取媒体方法（新方法）
    // 使用新方法打开摄像头
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({
            video: true,
            audio: true
        }).then(function(stream) {
            console.log(stream);

            mediaStreamTrack = typeof stream.stop === 'function' ? stream : stream.getTracks()[1];

            video.src = (window.URL || window.webkitURL).createObjectURL(stream);
            video.play();
        }).catch(function(err) {
            console.log(err);
        })
    }
    // 使用旧方法打开摄像头
    else if (navigator.getMedia) {
        navigator.getMedia({
            video: true
        }, function(stream) {
            mediaStreamTrack = stream.getTracks()[0];

            video.src = (window.URL || window.webkitURL).createObjectURL(stream);
            video.play();
        }, function(err) {
            console.log(err);
        });
    }

    // 截取图像
    snap.addEventListener('click', function() {
        context.drawImage(video, 0, 0, 500, 400);
    }, false);

    // 关闭摄像头
    close.addEventListener('click', function() {
        mediaStreamTrack && mediaStreamTrack.stop();
    }, false);

    // 上传截取的图像
    upload.addEventListener('click', function() {
        jQuery.post('/uploadSnap.php', {
            snapData: canvas.toDataURL('image/png')
        }).done(function(rs) {
            rs = JSON.parse(rs);

            console.log(rs);

            uploaded.src = rs.path;
        }).fail(function(err) {
            console.log(err);
        });
    }, false);

</script>
```
 
[getUserMedia支持情况-不容乐观：点击查看](https://caniuse.com/#search=getUserMedia)
