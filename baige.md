# js别踩白格
## 思想:
共24个小格,6行4列,随机产生小黑格在每一行中,方块自由下落循环交替产生两个不同div.当黑块落地不点击或者点击错误判定失败;

* 难点:6行4列小块的循环产生(2)落地的判定

## js代码:
1.整体方向:定义一个White的函数对象,里面存储着几个基本属性及方法

(1)count,记录分数

(2)Arr[],记录黑格的存储

(3)loc,记录是否落地

(4)root,根元素

(5)timer,时间计时器

(6)init()入口方法

2.在White原型上定义方法:

(1)getBlack()随机产生小黑格,代码:
```javascript
    getBlack:function(){
      var arr=[];
      for(var i=0;i<6;i++){    
        var ran=Math.floor(Math.random()*
        4)+i*4;
        arr.push(ran);
      }
      return arr;
    }
```
(2)重点部分:drawBoard()刻画方格,代码:
```javascript
    drawBoard:function(){
      var self=this; //防止闭包的错误引用,将this先存储到selfzhong
      var rArr=self.getBlack();
      self.Arr=rArr.concat(self.Arr)  //将小黑快存储在Arr数组中  
      var board=document.createElement("div");
      board.setAttribute("id",this.order()); //order()函数一会儿介绍
      board.style.position="absolute";
      board.style.top="-600px";  //创建一个div,表示即将下落的div
      for(var i=0;i<24;i++){
        var d=document.createElement("div");
        d.setAttribute("id","ele"+i);
        if(rArr.indexOf(i)>-1){
          d.setAttribute("class","squaceBlack");
        } //给小黑格加一个标志
        else{
          d.setAttribute("class","squace");
        }
        d.addEventListener("click",function(event){self.judge(event.target)},false); //需要使用匿名函数,若不使用匿名函数,judge函数中的this指向的d
        board.appendChild(d);
      } //利用for循环创建小块
    }
```
(3)jundge() 判断点击是否正确 //self.loc很容易漏掉
```javascript
    jundge:function(target){
      var self=this;
      var num=target.id.substr(3); //取得id的数字
      if(num!=self.Arr.pop()){
        clearTimeout(self.timer);
        alert("你的得分："+self.count+"分"); //当点击错误时判定失败
        return;
      }
      else{
        self.loc+=100; //当点击正确时,让loc的空间加100
        target.style.backgroundColor="silver";
        self.count+=1; //分数加一
        var score=document.getElementById("score").getElementsByTagName("i")[0];
        score.innerText=self.count;
        if(self.count%15==0){
          //产生更快的时间
          clearTimeout(self.timer);
          var newtimer=50-self.count/15*5;
          self.timer=setInterval(function(){self.fall(self)},newtimer);
        }
      }
    }
```
(4)难点部分:fall() 自由下落,难点部分,了解此部分前,先看下order函数
```javascript
    fall:function(){
      var self=this;
      var before=document.getElementById("before");
      var bNum=parseInt(before.style.top);
      if(bNum==595) //此处是595,若正好600,会产生空隙
      {
        self.root.removeChild(before);
        self.drawBoard(); //一个div执行完立即产生另一个div
      }
      bNum+=5;
      before.style.top=bNum+"px";
      var after=document.getElementById("after");
      var aNum=parseInt(after.style.top);
      if(aNum==595){
        self.root.removeChild(after);
        self.drawBoard();
      }
      aNum+=5;
      after.style.top=aNum+"px";
      self.loc-=5;
      if(self.loc==0) //另一个失败条件,没有空间
      {
        clearTimeout(self.timer);
        alert("你的得分："+self.count+"分");
  			return;
      }
    }
```
(5) order() 交替产生两个div的关键
```javascript
    order:function(){
      if(this.ord=="after"){
        this.ord="before";
      }
      else{
        this.ord="after";
      }
      return this.ord;
    }
```
只有两个id,当一个id消失后,立刻产生另一个id,此函数是关键函数

(6)init() 入口函数,自由下落计时器,代码:
```javascript
    init:function(){
        var self=this;
        self.drawBoard(); //先产生一个div
        self.timer=setInterval(function(){self.fall()},50);  //需要使用匿名函数,使fall函数的对象为White对象,方便fall函数调用White中的其他函数
      }
```
## 总结:
这个小游戏也参考了一些其他人写的别踩白格,又结合了自己的思想,才写出来,在我看来几个比较关键的点:
(1)异步函数中调用其他函数时的对象转换易出错
例:
```javascript
    self.timer=setInterval(function(){self.fall()},50);
```
这部分代码中借用了匿名函数function,才使得调用fall函数的对象为self,否则若:
```javascript
    self.timer=setInterval(self.fall(),50);
```
此时调用fall函数的队形就为setInterval对象,即为window对象.

又如:
```javascript
    d.addEventListener("click",function(event){self.judge(event.target)},false);
```
click异步函数同样通过匿名函数调用了judge,与上一个调用fall函数原理相同,此处不做过多解释了

(2)loc的计数,借鉴了别人的才写出来,之前想着设置一个落地标志,但发现设置落地标志就写死了,没办法动态改变,loc先设置为600,点击成功加100,每次下落loc减5,同时top加5,当loc为0时失败

(3)对象上的Arr属性要随时存储,当产生一组小黑格,就要及时加入到Arr中,就像一个大袋子与一个限量的小袋子一样,当小袋子装满后要及时放入大袋子中,这样在取大袋子中的东西时才能保证不漏掉,代码:
```javascript
    var rArr=self.getBlack();
    self.Arr=rArr.concat(self.Arr);
```

*此处尤为需要注意:在检测点击的是否为小黑格的判断代码中:*
```javascript
       if(rArr.indexOf(i)>-1){
         d.setAttribute("class","squaceBlack");
       }
```
*一定要使用创建的rArr数组进行检测,若使用:*
```javascript
     if(self.Arr.indexOf(i)>-1){
      d.setAttribute("class","squaceBlack");
}
```
*self.Arr进行检测一定会产生多个小黑格,因为数组存储超过了6个,会产生重复的黑格(这个地方我找bug找了一天,各种数组检测输出,其实感觉自己有点傻了,产生多余的小黑块一定是在命名时出错的,结果从整个代码中找的.....)*

(4)其他的就是一些基本的方法了,交替产生两个div不容易想到,但还是比较容易理解的
