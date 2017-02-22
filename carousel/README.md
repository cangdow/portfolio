摘自:http://www.jianshu.com/p/95b8b5b0ef80

这是前阵子写的2款原生JS轮播，一个是渐变轮播预览，一个是滚动轮播预览，现在补充博文总结。

渐变轮播

因为是梳理自己的思路，以下内容不贴代码，具体效果和代码见Github，可能有更好更简洁的写法，希望互相指正。
预览效果 源码地址
兼容性：IE7及以上，火狐和谷歌等主流浏览器

HTML&CSS

最外面1个总容器#ad，里面放1个图片容器#list，1个按钮容器#btns，2个箭头#prev和#next；#ad相对定位，其他相对于#ad绝对定位。
所有图片都是绝对定位，会重叠在一起，默认z-index:0；给选中的图片和按钮分别设置class为selected和on，其中选中图片的z-index:1，才会出现在顶层。
JS

手动轮播：把选中图片和按钮添加相应类名selected和on，把原来选中的按钮和图片取消相应类名。需要用循环，把所有按钮都遍历一遍，并且给每个按钮都定义一个索引index值。其中，需要判断选中当前已选中的，则不操作。
自动轮播。函数nextPic用于实现下一张图片，设置一个定时器，每隔一定时间就执行一次函数nextPic。注意处理移动到最后一张和第一张
注意：鼠标移入按钮时，自动轮播应该暂停；鼠标移出按钮时，自动轮播应该继续，且给一个当前的索引，告诉自动轮播从哪里开始播。
修改代码：将第1步和第2步中代码相似的部分合并写为一个函数show()
将图片和左右箭头鼠标悬停时清除计时器，写成函数pause，将图片和左右箭头鼠标离开时重启计数器，写成函数restart
给向左箭头写一个函数prevPic，用于显示上一张图片
设置左右箭头的开始和暂停
为了实现切换时的透明度渐变效果，需要给图片设置transition属性。只设置selected的话则会出现白底渐变效果，需要给所有图片也设置默认的transition属性。但只兼容IE10及以上，因为IE9不支持transition属性
兼容IE9及以下

因为IE9不支持transition属性，尝试用js实现渐入渐出，思路如下：

写透明度变化函数
function setOpacity(elem,level){
 if(elem.filters){ //IE9及以下有这个属性
     elem.style.filter="alpha(opacity="+level+")";
 }else{
     elem.style.opacity=level/100;
 }
}
透明度变化是动态的，隔一段时间就要变化，首先想到用setInterval
function fade_In(elem){
 setOpacity(elem,0); //初始透明
 var level=0;
 setInterval(function(){
     if(level<=95){
         level+=5;
         setOpacity(elem,level);
         console.log(level);
     }
 }, 300);
}
也可以用setTimout，感觉比较绕（挺有意思的，值得深究）。先考虑用循环，注意循环内的匿名函数问题，可以用括号的方法自执行。

//淡入效果
function fadeIn(elem){
 setOpacity(elem,0); //初始透明
 for(var i=0;i<=20;i++){
     (function(){
         var level=i*5;
         //try 1
         //console.log(level); //测试用，会一次性全部依次输出，如L61
         //setOpacity(elem,level);  //如果这样写，而不用L54的setTimeout的话是无效的，会一次性地全部实现，没有动画效果。

         //try 2
         /*if(level<=100){
             console.log(level);
             setInterval(setOpacity(elem,level), 300);  //每隔一定时间就执行依次函数，但这里level已经是100了。应该是每隔一段时间就变化level。另外写一个。
         }*/

         //正确：
         setTimeout(function(){
             setOpacity(elem,level)
         }, i*300);   //于是用setTimeout：i=0，间隔为0时，level=0；i=1，间隔为300时，level=5，依次……从而实现渐变。
     })(i);  //传入参数i，函数立即执行，会依次执行21次
 }
}
注意渐出的函数，容易写错，如下：

//淡出效果1：错误，实际是淡入效果
function fade_Out(elem){
 for(var i=20;i>=0;i--){
     (function(){
         var level=i*5;  //level从100到0
         setTimeout(function(){
             setOpacity(elem,level);
             //console.log(level); //测试用，这里却是从0到100
         }, i*300);
         //因为i=0时，即间隔时间为0，level=0；i=20，即间隔时间为6000，level=100。虽然for循环是从大到小，但setTimeout执行却是按照间隔时间的顺序的。所以实际上是淡入效果。
     })(i);
 }
}
正确的如下：

//淡出效果2：正确
function fadeOut(elem){
 for(var i=0;i<=20;i++){
     (function(){
         var level=100-i*5; //level从100到0
         setTimeout(function(){
             setOpacity(elem,level)
         }, i*300);
     })(i);
 }
}
在show函数里添加渐入和渐出，第一轮轮播切换时会出现第5张图闪一下再切换到目标图片，后面几轮轮播就不会了———因为CSS中没有设置默认透明度为0，选中透明度为1，切换的时候原来选中的一下子就变到底层了，所以会有第5张闪现。

P.S.由于本例中尝试用setTimeout写渐出时的出错，引发的问题如：循环中的匿名函数问题，setTimeout的多次执行，另外补充一篇博文分析。

滚动轮播

明天继续。
