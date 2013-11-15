bigPic
======

电商网站图片放大镜效果

图片放大查看效果在购物网站基本是必有的一个功能，功能就是鼠标放到图片上，图片这个区域放大显示。
如下图：

详细源码：
http://pan.baidu.com/share/link?shareid=65755&uk=52813371

原理比较简单：
设置一个倍数k，获取框（灰色小区域）是原框（图片）大小的1/k；放大的图片是原框（图片）大小的k倍；
设置一个放大图片展示框，这里展示框大小和原框大小相同，设置好超出区域不可见。

当获取框在原框中选取内容时，产生一个原图k倍大小的图（这个图层z-index很低），获取框移动时，K倍的放大图也同时移动（放大图片位置根据获取框位置改变），但是由于放大图片的展示框大小限定，而且超出部分不可见，所以这个展示框只截取了放大图片的一部分，这时就产生了一个放大图片的效果。

其中的难点就是计算各种框移动时的位置。

源码中写的一个构造函数：
function Magnifier(option){
     this.cont=option.cont; //装载原框的div
     this.bor=option.bor;  //获取框
     this.img=option.img;  //放大的图像
     this.mag=option.mag;  //放大框
     this.scale=option.scale||4; //比例值,默认值为4
     this.imgHttp=option.imgHttp||$("#img")[0].src; //如果设置新图片地址，则使用和小图一样的地址
 
     this.init(); //初始化
}

首先初始化：
init:function(){
//给放大的图像初始化属性
this.img.css({
'position' : 'absolute',
   'width' : (this.cont.innerWidth() * this.scale) + 'px',    
   //原始图像的宽*比例值 
   'height' : (this.cont.innerHeight() * this.scale) + 'px'    
   //原始图像的高*比例值
});
//给放大框初始化属性
this.mag.css({
'display' : 'none',
'width' : this.cont.innerWidth() + 'px',   
//m.cont为原始图像,与原始图像等宽
'height' : this.cont.innerHeight() + 'px',
'position' : 'absolute',
'left' : this.cont.offset().left + this.cont.outerWidth() + 10 + 'px',  //放大框的位置为原始图像的右方远10px
'top' : this.cont.offset().top + 'px'
});
var borderWid = this.cont.outerWidth() - this.cont.innerWidth();
this.bor.css({
  'display' : 'none',        //开始设置为不可见
  'width' : this.cont.innerWidth() / this.scale - borderWid + 'px',   
  //原始图片的宽/比例值 - border的宽度
  'height' : this.cont.innerHeight() / this.scale - borderWid + 'px',  
  //原始图片的高/比例值 - border的宽度
  'opacity' : 0.5     //设置透明度
});
//以上绿色部分为对样式初始化
this.img[0].src = this.imgHttp;   //设置放大图像的src值

 var _this=this;
  this.cont.bind('mousemove',function(e){
  _this.move(e);
  })
  this.cont.bind('mouseout',function(){
  _this.end();
  })
},
//绑定两个事件，在图片上面移动的事件和离开图片的事件

在图片上获取时的事件，放大图片和获取框同事变化：
move:function(e){
     this.bor.show().css({
          'top' : Math.min(Math.max(e.pageY - this.cont.offset().top - parseInt(this.bor.innerHeight()) / 2,0),this.cont.innerHeight() - this.bor.outerHeight()) + 'px',
          'left': Math.min(Math.max(e.pageX - this.cont.offset().left - parseInt(this.bor.innerWidth()) / 2,0),this.cont.innerWidth() - this.bor.outerWidth()) + 'px'
          //left=鼠标x - this.offsetLeft - 浏览框宽/2,Math.max和Math.min让浏览框不会超出图像
         //上面绿色部分是限定移动框范围的一个很好的写法，很值得学习
     })
     this.mag.show();
     this.img.show().css({
          'top' : - (parseInt(this.bor[0].style.top) * this.scale) + 'px',
          'left' : - (parseInt(this.bor[0].style.left) * this.scale) + 'px'
     })
},

离开框时的事件：
end:function(e){
     this.bor.hide();
     this.mag.hide();
}


这个效果是参考：
http://www.blueidea.com/tech/web/2009/7087.asp 
这篇文字里面有更加详细的解释，上面的也是我改写的一个jquery的构造函数形式。
