---
title: "微信小程序图片压缩及base64化上传"
---

## 微信小程序图片压缩

微信小程序原生提供了图片原图上传和压缩上传的办法，示例如下：

```html
<view class="section">
    <button size="mini" bindtap="uploadImage">Upload Image</button>
</view>
```

```js
Page({
  data: {
    imgUrl: '',
  },
  uploadImage: function (e) {
    var _this = this;
    wx.chooseImage({
      count: 1, // 可选择图片的数量，默认为9，当前为1
      sizeType: ['original', 'compressed'], // 可以指定是原图上传还是压缩图上传，默认二者都有，假如删掉'original'则只有压缩上传。
      sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
      success: function (res) {
        // 返回选定照片的本地文件路径列表，tempFilePath可以作为img标签的src属性显示图片
        var tempFilePaths = res.tempFilePaths;
        _this.setData({
          imgUrl: tempFilePaths[0]
        }) //将生成的图片url保存下来，后面继续处理
      }
    })
  },
})
```

## 微信小程序图片base64化

在微信小程序中将图片base64化需要借助微信原生的 [wx.canvasGetImageData](https://developers.weixin.qq.com/miniprogram/dev/api/canvas/get-image-data.html) 得到图片的像素信息，再通过开源库 [UPNG](https://github.com/photopea/UPNG.js/) 将像素信息编码，最后通过`wx.arrayBufferToBase64`转化为base64数据。主要过程如下：

```html
<!-- canvas实例，写在wxml文件中 -->
<canvas style="width: 300px; height: 200px;" canvas-id="myCanvas"></canvas>
```

```js
//转换处理函数
var getBase64Image = function (canvasId, imgUrl, callback, imgWidth, imgHeight) {
  const ctx = wx.createCanvasContext(canvasId);
  ctx.drawImage(imgUrl, 0, 0, imgWidth || 300, imgHeight || 200);
  ctx.draw(false, () => {
    // API 1.9.0 获取图像数据
    wx.canvasGetImageData({
      canvasId: canvasId,
      x: 0,
      y: 0,
      width: imgWidth || 300,
      height: imgHeight || 200,
      success(res) {
          var result = res;
        // png编码
        var pngData = upng.encode([result.data.buffer], result.width, result.height);
        // base64编码
        var base64 = wx.arrayBufferToBase64(pngData);
        var base64Data = 'data:image/png;base64,' + base64;
        callback(base64Data);
      }
    })
  })
};
```

```js
//调用该处理函数的示例，this.data.imgUrl为上文中得到的上传文件tempFilePath
getBase64Image('myCanvas', this.data.imgUrl, function(base64Data){
    //  在此处处理得到的base64数据
});
```

但是在iOS下通过`wx.canvasGetImageData`得到的像素信息相比原图是上下颠倒的，我们可以直接通过判断当前平台来决定是否将颠倒的数据修复，代码如下：

```js
let platform = wx.getSystemInfoSync().platform
if (platform == 'ios') {
    // 兼容处理：ios获取的图片上下颠倒
    result = reversedata(res)
};

var reversedata = function(res) {
  var w = res.width;
  var h = res.height;
  let con = 0;
  for (var i = 0; i < h / 2; i++) {
    for (var j = 0; j < w * 4; j++) {
      con = res.data[(i * w * 4 + j) + ""];
      res.data[(i * w * 4 + j) + ""] = res.data[((h - i - 1) * w * 4 + j) + ""];
      res.data[((h - i - 1) * w * 4 + j) + ""] = con;
    }
  }
  return res;
};
```

不过这样还是有新的问题，就是万一什么时候微信官方将这个bug修复了，我们在iOS下的这个处理就会又好心办坏事，把原本正常的图片给搞颠倒了，所以最好的办法还是直接根据导出图片信息是否颠倒来判断，而不是仅仅根据平台判断。

办法是我们自己先建一个隐藏的canvas，然后在这个canvas上绘制两个已知像素信息的点，再把这个canvas的信息通过`wx.canvasGetImageData`导出，与已知的像素信息比较，从而得出当前平台导出的图片是否会有问题，代码如下：

```html
  <canvas style="width:2px;height:2px;visibility:hidden;" canvas-id="judgeCanvas"></canvas>
```

```js
var checkOrientation = function(canvasId) {
  // 取得canvas上下文，然后在这个2px*2px的canvas左上角绘制一个1px*1px的红点，右下角绘制一个1px*1px的黄点。
  const ctx = wx.createCanvasContext(canvasId);  //在这个场景下canvasId="judgeCanvas"
  ctx.setFillStyle("red");
  ctx.fillRect(0,0,1,1);
  ctx.setFillStyle("yellow");
  ctx.fillRect(1,1,1,1);
  ctx.draw(false, () => {
    wx.canvasGetImageData({
      //调用wx.canvasGetImageData获得返回像素点数据，并与已知像素点数据比较是否相同。
      canvasId: canvasId,
      x: 0,
      y: 0,
      width: 2,
      height: 2,
      success(res) {
        var expectedFirstPoint = [255, 0, 0, 255]; //一个红点的RGB值和透明度
        var expectedLastPoint = [255, 255, 0, 255]; //一个黄点的RGB值和透明度
        var w = res.width;
        var h = res.height;
        var data = res.data;
        var totalCount = w*h*4;
        var realFirstPoint = data.slice(0,4);
        var realLastPoint = data.slice(totalCount - 4, totalCount);
        var isOrientationRight = areTwoArraysIdentical(expectedFirstPoint, realFirstPoint) && areTwoArraysIdentical(expectedLastPoint, realLastPoint);
        try {
          wx.setStorageSync('isOrientationRight', isOrientationRight); //将当前设备导出图片是否会颠倒的信息储存在storage中，方便以后判断
        } catch(ex) {
          console.error(ex.message);
        };
        console.log('isOrientationRight: ' + isOrientationRight);
      }
    })
  });
};

//比较两个数组内部信息是否相同
var areTwoArraysIdentical = function(array1, array2) {
  var totalCount = array1.length, isIdentical = true;
  var i;
  for (i = 0; i < totalCount; i++) {
    if(!isIdentical) {
      return isIdentical;  //只要一个不同，直接跳出
    }
    isIdentical = array1[i] === array2[i];
  }
  return isIdentical;
};
```

只需要在项目启动后调用checkOrientation一次就可以了，用储存在storage中的isOrientationRight字段替换掉之前仅仅判断平台是否为iOS的代码。

```js
let platform = wx.getSystemInfoSync().platform

if (platform == 'ios') {
    // 兼容处理：ios获取的图片上下颠倒
    result = reversedata(res)
};
```

改写为：

```js
var isOrientationRight = wx.getStorageSync('isOrientationRight');
//根据之前获得的当前设备canvas导出图片是否正常来确定是否反转上传图片
if (isOrientationRight) {
  result = res;
} else {
  result = reversedata(res);
}
```

将以上代码合并，我们就能得到一个完善的微信小程序图片压缩并base64化上传的方案。

详细代码和demo请见[wx-image-uploader](https://github.com/xuweilu/wx-image-uploader)
