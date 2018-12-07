---
title: 小程序用canvas绘制拼接图片
date: 2018-12-07 13:22:51
---
因为小程序的一些限制，目前不支持整个直接分享到朋友圈，所以需要根据不同的用户id在后端生成二维码，拿到这个二维码图片后，把它跟设计好的海报拼接起来，保存到本地，这样就可以把保存好的这个张海报图片分享到朋友圈。
接下来讲讲拼接图片的方法。

#### 首先需要获取微信小程序手机相册相关授权
因为保存海报需要将网络图片保存到本地相册，所以需要通过小程序相对应的授权：是否授权保存到相册。我这里保存海报有一个按钮，如果获取了相册授权，则保存海报按钮显示；如果拒绝授权，则保存海报按钮隐藏，此时应该引导用户手动打开相册。
wpy代码如下：
``` bash
  button.btn(wx:if="{{!isShowDownload}}" open-type="openSetting" @tap="handleOpenSettingPhotosAlbum") 打开微信相册授权
  button.btn(wx:if="{{isShowDownload}}" @tap="handleDrawImg") 保存海报
```
js代码如下：
``` bash
data = {
  isShowDownload: false,
  imgBg: '../../images/ad_bg.jpg' // 海报背景图片
}

onLoad() {
  // 小程序在加载时就检查相册授权
  this.handleWritePhotosAlbum()
}

// 获取相册授权
handleWritePhotosAlbum () {
  var that = this
  wx.getSetting({
    success(res) {
      if (!res.authSetting['scope.writePhotosAlbum']) {
        console.log('检测暂未授权小程序访问相册')
        wx.authorize({
          scope: 'scope.writePhotosAlbum',
          success() {
            console.log('获取相册授权')
            that.isShowDownload = true // 显示保存海报按钮
            that.$apply()
          },
          fail() {
            console.log('获取相册授权失败')
            commonMixin.wHandleWarning('提示', '请授权小程序访问您的相册，否则无法保存海报到您的手机相册~')
            that.isShowDownload = false // 拒绝授权，不显示保存海报按钮
            that.$apply()
          }
        })
      } else {
        console.log('检测到您已授权小程序访问相册')
        that.isShowDownload = true // 之前已授权
        that.$apply()
      }
    }
  })
}

// 引导用户手动打开微信相册授权
handleOpenSettingPhotosAlbum () {
  var that = this
  wx.openSetting({
    success: (res) => {
      if (res.authSetting['scope.writePhotosAlbum']) {
        console.log('你打开了设置界面，并已手动开启相册授权')
        that.isShowDownload = true
        that.$apply()
        wx.authorize({
          scope: 'scope.writePhotosAlbum',
          success() {
            console.log('通过设置授权了相册')
            that.isShowDownload = true
            that.$apply()
          }
        })
      } else {
        console.log('你打开了设置界面，但并未手动开启相册授权')
        that.isShowDownload = false
        that.$apply()
      }
    }
  })
}
```

#### 获取了相关授权后，会显示保存海报按钮，此时点击保存海报按钮，触发canvas绘制图片的方法
先说一下思路
1. 需要先在页面上给一个canvas区域，设置ID，方便后面相对应的操作。这个canvas的尺寸，跟期望保存的海报尺寸一样就好，比如我的海报尺寸是750x1350，就设置canvas为750x1350；

scss代码如下：（之所以给一个绝对定位，是因为在手机保存海报的时候遇到一个问题，保存好的海报会覆盖在这个页面所有内容之上，所以给个定位，让它占位，但不覆盖当前页面需要显示的内容。）
```
.my-cavans{
  position: absolute;
  width: 100%;
  height: 100%;
  z-index: -99;
  opacity: 0;
  left: -9999px;
  canvas{
    width: 750px !important;
    height: 1350px !important;
  }
}
```
wpy代码如下：
```
  view.my-cavans
    canvas(canvas-id="notes")
```
js代码如下：
```
  const ctx = wx.createCanvasContext('notes', this)
```

2. 使用HTML5的drawImage函数：
+ drawImage(image, x, y, width, height) // 按指定大小绘制
+ image:所要绘制的图像。这必须是表示 标记或者屏幕外图像的 Image 对象，或者是 Canvas 元素
+ x和y：图片在文档中的坐标位置
+ width和height：图片的宽高

js代码如下：
```
  ctx.drawImage(this.imgBg, 0, 0, 750, 1350)
```
如果想要绘制多张图，即拼接，那就多使用几次drawImage，比如：
```
ctx.drawImage(img1, 0, 0, 750, 1350)
ctx.drawImage(img2, 0, 0, 200, 200)
```
好了，背景图片就这样已经绘制并拼接了。

3. 保存绘制的图片到手机相册：
这里用到了小程序的一个方法：wx.canvasToTempFilePath(Object object, Object this)，把当前画布指定区域的内容导出生成指定大小的图片。在 draw() 回调里调用该方法才能保证图片导出成功。
这个方法的参数不作赘述，可以在小程序的官方文档里搜一下。
使用了这个方法后，会获得一个生成文件的临时路径。
拿到临时路径后，调用小程序的另一个方法：wx.saveImageToPhotosAlbum(OBJECT)，保存图片到系统相册。
到此为止，绘制拼接图片就完成了。有一个需要注意的地方就是，绘制图片是按顺序来的，最先绘制的图片在最下面，以此类推。

完整的js代码如下：
```
handleDrawImg () {
  console.log('点击了绘图')
  commonMixin.wHandleErrorCont('正在保存海报，请稍后…')
  wx.getImageInfo({
    src: this.agentQRCode,
    success: (res) => {
      const ctx = wx.createCanvasContext('notes', this)
      ctx.drawImage(this.imgBg, 0, 0, 750, 1350)
      ctx.drawImage(res.path, 250, 1050, 250, 250)
      ctx.draw(true, setTimeout(function () {
        wx.canvasToTempFilePath({
          width: 750,
          height: 1350,
          destWidth: 750,
          destHeight: 1350,
          canvasId: 'notes',
          fileType: 'jpg',
          success: function (res) {
            wx.saveImageToPhotosAlbum({
              filePath: res.tempFilePath,
              success (res) {
                console.log('成功')
                commonMixin.wHandleErrorCont('海报已保存至您的相册~')
                console.log(res)
              },
              fail () {
                console.log('不成功')
                commonMixin.wHandleWarning('提示', '保存海报失败，请在设置中打开微信相册访问权限~')
                console.log(res)
              }
            })
          },
          fail: function (res) {
            console.log(res)
          }
        }, this)
      }, 1500))
    }
  })
}
```
到这里就全部完成了，后面因为客户要求拼接的二维码必须要有圆角，而canvas绘制的图片都是按原图进行绘制的，画圆角并拼接比较麻烦，所以我用了一个比较投机取巧的办法，就是按照二维码的尺寸另外做一个带有圆角但空心的图片，覆盖在二维码上，这样二维码看起来就是圆角的拉~
```
  ctx.drawImage(this.imgBg, 0, 0, 750, 1350) // 这是背景图片
  ctx.drawImage(res.path, 250, 1050, 250, 250) // 这是二维码图片
  ctx.drawImage(this.imgBorder, 250, 1050, 250, 250) // 这是覆盖在二维码上的圆角边框图片
```




