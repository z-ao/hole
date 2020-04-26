# 踩坑记录🤢
## 格式
+ 标题   
+ 详情
+ 技术标签

### webpack动态加载
动态加载可以减小初始包体积。但语法与es6相似，造成困惑。  

```
import _ from 'lodash' // es6引入方式
import('lodash').then(_ => {
    // Do something...
}) //webpack动态加载方式，注意返回Promise

//其他动态加载方式
require.ensure(['lodash'], function(require) { //CommonJS
    // Do something...
});

define(['lodash'], function(_) { //AMD  
    // Do something...
})
```

文档地址: https://www.webpackjs.com/guides/code-splitting/#%E5%8A%A8%E6%80%81%E5%AF%BC%E5%85%A5-dynamic-imports-  


### 第三方浏览器劫持video的控制栏
当页面运行在qq、ucApp这些第三方浏览器，有可能video标签被劫持，替换成App的逻辑。      
但这些与业务出现冲突。更加严峻的是，会被插入广告、给私密视频提供下载逻辑。    

最好的方案是，联系浏览器厂商，把网站加入白名单里，防止被劫持。    
降级的方案是，使用canvas实现video功能。（但对性能和体验会有影响）    
在降级的方案，调整业务逻辑，例如私密视频只播一小段，然后引导用户使用其他客户端。    

碎碎念   
个别手机限制，不能在video是最高层级，不行有任何模块遮挡。   
如果个别原生浏览器不支持controlsList（定义媒体元素上显示的控件），可以试style的伪类是否生效。   

```
video::-webkit-media-controls-enclosure {
    overflow: hidden !important;
    -webkit-appearance: none;
}
video::-webkit-media-controls-panel {
    width: calc(100% + 30px);
    display: none !important;
    -webkit-appearance: none;
}
video::-webkit-media-controls-start-playback-button {
    display:none !important;
    -webkit-appearance: none;
}
video::-webkit-media-controls {
    display:none !important;
    -webkit-appearance: none;
}

video::--webkit-media-controls-play-button {
    display: none !important;
    -webkit-appearance: none;
}
```    


### ios applink不生效
如果**appLink**配置的路径是***baidu.com/app/link***，那么在***baidu.com***域名下使用**appLink**，不会生效。    
因为Ios有限制，必须要跨越才生效，   
**appLink**配置在***baidu.com***域名，那么只会要除***baidu.com***的其他域名生效。eg，***m.baidu.com***、***music.baidu.com***。

### 手机端后退不获取HTML
最近在一些app内的webview，在后退的时候不用发送HTML请求，这导致数据不同步的问题。    
但浏览器提供**pageshow**事件，该事件在onload之后触发。

```
//persisted 为 true 表明从缓存获取
//performance.navigation.type 为 2 表明是后退操作
window.addEventListener('pageshow',function(event){
    if(event.persisted || (window.performance && window.performance.navigation.type === 2)){
        window.location.reload();
    }
},false);
```

### web推流
现在推流技术有，RTMP，Hls，WebRtc

+ RTMP需要依赖Flash，不兼容手机，Tcp流速度比Hls快。第三库 [rtmp-streamer](https://github.com/chxj1992/rtmp-streamer/blob/master/rtmp-streamer.js)
+ Hls不支持web推流
+ WebRtc支持端对端传二进制数据，但需要服务器支持把二进制转对应数据格式(m3u8,flv)


### 移动页跳转到App
接到一个需求，需要移动页跳转到App。这时候需要用到**DeepLink**技术。    
通过调研，发现不是接入个连接那么简单。会遇到几种特殊情况。   

+ **连接在微信被拦截**     
  微信不支持他不信任的页面直接打开App，所以做了拦截。但我们可以跳转到应用宝，   
  然后点击直接跳转到应用商店下载或者引导用户打开浏览器访问，在浏览器跳转App。
+ **用户没有安装App**    
  浏览器没有提供设备有没有安装对应App的接口。但可以使用取巧的方法   

  ```
  方案一
  location.href = deepLink; //如果下载了App，设备自动识别成功，直接跳转。
  //如果没有下载，执行下面方法，跳转到下载App页面
  setTimeout(function(){
           location.href = downloadUrl 
  }, 0)
  弊端：如果下载了app，会同时打开app与shop
  
  方案二
  location.href = deepLink; //页面打开立刻跳转
  //如果没有下载，跳转失败，引导用户点击跳转到shop
  <a href="javascript:;" click="download"></a>
  function download() {
  		location.href = downloadUrl;
  }
  ``` 
  

### 全屏与锁定旋转
浏览器**Full Screen**API能控制标签全屏，但必须要htmlDocument对象调用。

```
//例如想video标签全屏 
document.querySelector('video').requestFullscreen();

//直接使用变量保存方法会失败
var fullScreen = document.querySelector('video').requestFullscreen;
fullScreen(); //typeerror
```
浏览器有**lockOrientation**操作屏幕旋转，但只能在andriod机型以及全屏使用。

### poster andriod不生效
在andriod系统 poster显示黑色    
需要添加 ``x5-video-player-type``

### SVG动画缓存
image标签引入一张svg动画图片播放完后，在其他容器引入同一张svg，动画不会播放。    
因为当第一次引入后，图片会缓存在内存里。第二次再引入时，图片会在内存获取，但获取的图片已经是播放完的。     
但object标签不会缓存资源，所以使用object可以去除内存的缓存。    

### html2canvas
在手机页面使用html2canvas生成图片，但是由于二维码不清晰  
所以识别不到    
后来发现图片是背景图的就造成不清晰，所以用img替代背景图。

### laravel 模版层函数使用
在套模版中 ``{{ }}``可以使用帮助函数，但不能使用原生函数。   
在 ``if``类型的语句才能使用原生函数。

### ios video自动播放或者play函数失效
在ios上设置autoplay或者调用play函数，video不会播放，   
原因是，ios作者不想用户在不清楚状况下，被突然播放视频打扰，所以禁止事件。   
只有其中一种情况就可以生效   

1. 视频设置静音
2. 视频要可视区域里

### vue router高亮
vue router是根据URL路径和嵌套匹配的.

```
{
	path: '/aaa',	
	children: [
		{
			path: '/aaa/bbb', //success
		},
		{
			path: '/ccc/aaa', //fail
		},
	]
},
```
<span style="display: inline-block; background: #eff3f6; padding: 0 4px ">vue</span>
<span style="display: inline-block; background: #eff3f6; padding: 0 4px ">vue router</span>

## mv隐藏文件
但遇到把origin文件夹所以文件移到target文件夹中，执行

```
mv origin/* target
```

但发现origin里面的 ```.```开头隐藏文件没有迁移。后来查询发现。   
需要运行


```
shopt -s dotglob
```
bash在文件名扩展的结果中包括以点（.）开头的文件名

## iphone内容超入滚动区域显示空白
但块状区域内容过多时，在iphone浏览器会显示空白，但区域是能交互的，例如button可以点击。

```
document.querySelector('element').parentElement.style.display = 'none';
document.querySelector('element').parentElement.style.display = 'block';
```

在stackoverflow解决方案   
<https://stackoverflow.com/questions/17582439/webkit-overflow-scrolling-touch-large-content-gets-cut-off-when-specifying-a>

## iphone 上需要点击两次才出发点击事件
最近在一个移动页面使用了element-ui的，select组件，需要点击两次才选中。导致原因如下

>Google开发者文档中有提到：   
>mobile browsers will wait approximately 300ms from the time that you tap the button to fire the click event. The reason for this is that the browser is waiting to see if you are actually performing a double tap.   
>移动浏览器为什么会设置300毫秒的等待时间呢？这与双击缩放的方案有关。平时我们有可能已经注意到了，双击缩放，即用手指在屏幕上快速点击两次，可以看到内容或者图片放大，再次双击，浏览器会将网页缩放至原始比例。   
>浏览器捕获第一次单击后，会先等待一段时间，如果在这段时间区间里用户未进行下一次点击，则浏览器会做单击事件的处理。如果这段时间里用户进行了第二次单击操作，则浏览器会做双击事件处理。这段时间就是上面提到的300毫秒延迟。

解决办法是使用fastclick.js   

```
引入脚步
https://github.com/ftlabs/fastclick/blob/master/lib/fastclick.js

执行方法
window.addEventListener(function(){   
    FastClick.attach( document.body );  
},false );
```