## 1.功能介绍


打开‘商品手账’小程序，进入小程序首页。

因为找不到一瓶可乐，所以我找来了一只非常专业的猫，扮演一瓶330ml的可口可乐演示给大家看。

摁住它，对准它的条码，扫一下，‘嘀’~地一声，说明扫码成功了，这时页面就自动跳到商品详情页。

如果别人曾经在这个商品下留言的话，你就可以看得到留言，当然你也可以点击右下角的蓝色按钮就发表你的留言了；  
 

  ![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901083227937-1916971089.jpg)


因为一个商品对应一个条码，扫描同一个商品条码就可以进入同一个留言列表；  
留言越多的商品，在排行榜上就排在越前面；

接下来详细介绍一下每个页面


##  2.小程序首页

  ![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901080700846-2105637324.png)

 首页分三个部分：

 &nbsp;&nbsp;&nbsp;&nbsp;1.用户的头像和昵称显示；
 &nbsp;&nbsp;&nbsp;&nbsp;2.轮播图-可以放一些平时要展示的图片；
 &nbsp;&nbsp;&nbsp;&nbsp;3.扫码按钮-点击即可打开微信扫码；

当用户点击扫码按钮时，我们就调用小程序的扫码接口去扫描商品条码，当用户扫描好条码后，我们就得到了商品条码（barcode）；  
这时，我们就可以跳转到商品详情页面了，顺便把条码传过去，这样商品详情页才能知道用户扫的是什么商品：

```
    wx.navigateTo({
          url: "/page/component/proDtl/proDtl?barcode=" + barcode,  //把商品条码传给商品详情页
    })
```


## 3.商品详情页

   ![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901084754661-691288380.png)

进入详情页后，我们的第一件事情：在生命周期onLoad中获取首页传过来商品条码，然后根据条码请求当前商品的留言列表，如果这个商品还没有人扫过的话，就可能没有留言，那我们只要显示“暂无留言”即可

```
  onLoad(options){
       var barcode = options.barcode ? options.barcode:'';
       this.getProductInfo(barcode)  //根据条码请求当前商品的留言列表
  },
```

在getProductInfo（）方法中，我们会向后台请求商品留言列表的和商品的名称；
接着我们就把请求到的数据渲染到页面上:

![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901091528358-1151246726.png)


如果用户觉得请求到的商品名称是不对的，还可以点击名称进行编辑：

![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901091621411-985523100.png)


最后，页面底部还有一个提交留言的小按钮：

![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901091543704-147020248.png)


如果你要发表留言，你可以用你的食指点击它：

![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901091603347-1350552944.png)


点击按钮后，小程序就会跳到提交留言页面，顺便把商品条码信息传过去：

```
  turnToSubmit(){
    wx.navigateTo({
      url: "/page/component/addNode/addNode?barcode=" + this.data.barcode,
    })
  },
```

## 4.添加留言页面

![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901093342813-751243144.jpg)

 如图，这时候我们就可以开始写我们的留言了。

 写完留言之后，你可以标注一下你的留言类型：
 如果你觉得你写的是一首诗，你可以选择类型为‘诗一首’；   
 如果你觉得你写的是一封信，等待别人扫码阅读，你可以选择类型为‘鱼书’；
 如果你扫描的是一本书的条码，你可以选择类型为‘书摘’；

 右边是上传图片功能，
 首先，我们点击'添加图片'小图标，这时就会使用小程序选择图片的接口打开相册或者直接拍照，
 得到图片之后，因为现在的手机图片拍照像素都比较高，导致图片比较大，上传会很慢，占服务器空间，请求也会很慢...
 所以为了优化用户体验，我决定压缩一下图片再上传。

```
 wx.chooseImage({
      count: 1, // 默认9  
      sizeType: ['original'], // 可以指定是原图还是压缩图，默认二者都有
      sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
      success: function (res) {  //图片选择成功之后
        
               var tempFilePaths = res.tempFilePaths;
               self.compressedImg(res)  //调用compressedImg方法，先把图片压缩一下

        }
    })
```
 
图片压缩是使用了小程序提供的canvas画布，把用户上传的图片搞到画布上（....），如果根据画布上的图片高和宽判断图片是否过大，如果过大，就直接把画布按比例缩小；

```
      while (canvasWidth > 220 || canvasHeight > 220) {  //如果宽度或者高度大于220，我就认为图片要进一步压缩，你可以根据需求改成其他的数字
          //比例取整
          canvasWidth = Math.trunc(res.width / ratio)
          canvasHeight = Math.trunc(res.height / ratio)
          ratio++;
       }
```
>图片的压缩参考自：[微信小程序：利用canvas缩小图片][https://blog.csdn.net/akony/article/details/78815544]


图片压缩完了之后，我们就开始上传这张图片：

```
   ......
   wx.uploadFile({  //上传图片
          url: 'https://mp.orancat.com/proImgUpload',
          filePath: photo.tempFilePath,  //压缩后的图片
          name: 'file',
          header: {
            'content-type': 'multipart/form-data'
          },
          success: function (res) {
    .......          

```
 
图片上传成功之后，后台会返回上传图片的地址给我们，我们把图片渲染到页面上，用户就会知道图片上传成功了；

最后点击'提交'按钮，就会把以下内容发送给后台，后台就会自动将留言保存到数据库；

```
   var data={
      authorName: userName, //用户昵称
      token: userId,  //用户ID
      content: this.data.textContent, //留言文本内容
      imgUrl: userImg,  //用户的头像
      code: this.data.barcode,  //商品的条码
      typeIndex: this.data.typeIndex, //留言的类型
      nodeImgUrl: this.data.nodeImgUrl //用户上传的图片的地址
    }
```

留言提交成功之后，页面会自动切回商品详情页面，这时，你就可以看到自己刚刚的留言了；


## 5.排行榜页面

![](https://images2018.cnblogs.com/blog/1178432/201809/1178432-20180901084851229-1528282815.png)

有留言的商品都会出现在排行榜页面，并且按照留言的数量进行排列，点击某个商品就会直接进入商品详情页面；
这页面主要做的事情就是请求排行榜的数据，然后再页面上渲染出来；

## 其他实现的功能

### 1.分页

在商品详情页，有可能出现这中情况，比如说假设A商品有120条留言，如果一进A商品详情页就要加载120条留言的话，可能要花一点时间，也可能页面加载半天都没有出来；这样的话用户体验就会非常不好。所以相对理想的方式应该是，假设12条留言为一页，那么A商品的留言总共有10页，当我们进入A商品的详情页面时，先加载第一页（前12条留言），当我们往上滑动页面到底部时就自动加载下一页的内容，一页一页按顺序加载；

我们使用小程序提供的OnReachBottom触底事件实现分页加载，当用户滑动留言列表到底部时触发加载下一页：

```
  onReachBottom: function () { //滑动到底部时触发
       this.setData({
           bottomLoading: true  // 显示loading提示
       })
       this.getRankList()  //请求下一页数据
  },
```

同理，排行榜页面也使用了分页加载；

### 2.通过wx.login获取用户唯一凭证openId
   ![](https://images2018.cnblogs.com/blog/1178432/201808/1178432-20180831135258317-1713789822.png)

由于用户的昵称，头像什么的都可能随时会改变，当openID不会变，所以使用openId作为用户唯一凭证；  
虽然我获取了用户的Id，但暂时还没有使用到；  
如果以后要弄个用户个人主页或者留言回复等等可能就要用到openId；

## 6.下载与本地电脑运行

在本文底部的github地址下载源码，用微信开发者工具打开项目，填上你的appId即可；

记得在开发者工具点击‘’详情‘’设置不校验域名：

 ![](https://images2018.cnblogs.com/blog/1178432/201808/1178432-20180831163206438-1734095750.png)

然后，由于一般电脑没有摄像头不能扫码，所以当你需要扫码时，你可以把下面这张条码图片保存在本地电脑上，点击扫码按钮时打开这张图片即可：

![clipboard.png](/img/bVbgkxB)

当然，你也可以自己找其他的条码；

另外，需要注意的是，当你在本地电脑调试时，由于我们都是使用同一个后台接口，所以你发的留言都会同步到我的小程序上，所以尽量不要发送太明显的测试留言，例如：

&nbsp;&nbsp;![](https://images2018.cnblogs.com/blog/1178432/201808/1178432-20180831164458356-1196936848.png)

可以发一些强颜欢笑，积极向上，人畜无害的留言，例如：

&nbsp;&nbsp;![](https://images2018.cnblogs.com/blog/1178432/201808/1178432-20180831164646085-261794296.png)


## 7.扫码体验

你也可以直接扫描这个的小程序码体验一下：  

![](https://images2018.cnblogs.com/blog/1178432/201808/1178432-20180831165024411-236298724.png)


>源码下载地址:[https://github.com/AUSERGEE/proSay]
