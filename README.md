<center><a target="_blank" href="https://cloudb.pub"><i class="fa-sharp fa-solid fa-pen-to-square"></i>   我的主页</a>    |    <a target="_blank" href="https://github.com/xyz349925756"><i class="fa-brands fa-github"></i>   Github</a>   |    <a href="https://gitee.com/xyz349925756" target="_blank"><i class="fa-brands fa-gitlab"></i>    Gitee</a>    |   <a target="_blank" href="https://xyz349925756.gitee.io/"><i class="fa-brands fa-internet-explorer"></i>   国内镜像网站</a></center > 

<hr>
<nav>
    <div id='themeChange' style="cursor: pointer;"><p><i class="fa-solid fa-jet-fighter-up"></i>更换Theme</p></div>
    <p><i class="fa-brands fa-themeco"></i>  Them Color : 
    <font style="color: green">Vue</font>
    <font style="color: dark">Dark</font>
    <font style="color: blue">Buble</font>
    <font style="color: purple">Pure</font>
    <font style="color: #00ffff">Dolphin</font>
    </p>
</nav>


---

关于这个网站：这个网站是使用GitHub Page 搭建的，上面出现 404 的页面是因为我取消了分享，因为这些技术在中国未来20年甚至更长时间是不会有大变动。就现在分享的这些学会的人在中国IT都是中上水准。

<!DOCTYPE html><html>
<head>
<style>   
    .box{
        overflow: hidden;
        border-radius: 5px;
    }    
.border-top {
  position: absolute;
  top: 0;
  right: 0;
  width: 100%;
  height: 3px;
  background: linear-gradient(to right,darkorange,darkcyan);
  animation: animate-border-top 2s linear infinite;
}
@keyframes animate-border-top {
  0% {
      transform: translateX(-100%);
  }
  100% {
      transform: translateX(100%);
  }
}
.border-right{
  position: absolute;
  top: 0;
  right: 0;
  width: 3px;
  height: 100%;
  background: linear-gradient(to bottom,darkorange,darkcyan);
  animation: animate-border-right 2s linear infinite;
  animation-delay: 1s;
}
@keyframes animate-border-right{
  0% {
      transform: translateY(-100%);
  }
  100% {
      transform: translateY(100%);
  }
}
.border-bottom {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 100%;
  height: 3px;
  background: linear-gradient(to left,darkorange,darkcyan);
  animation: animate-border-bottom 2s linear infinite;
}
@keyframes animate-border-bottom {
  0% {
      transform: translateX(100%);
  }
  100% {
      transform: translateX(-100%);
  }
}
.border-left{
  position: absolute;
  top: 0;
  left: 0;
  width: 3px;
  height: 100%;
  background: linear-gradient(to top,darkorange,darkcyan);
  animation: animate-border-left 2s linear infinite;
  animation-delay: 1s;
}
@keyframes animate-border-left{
  0% {
      transform: translateY(100%);
  }
  100% {
      transform: translateY(-100%);
  }
}
</style>
</head>
<body>
<div class="box">
   <h1>Border Animation</h1>
   <span class="border-top"></span>
   <span class="border-right"></span>
   <span class="border-bottom"></span>
   <span class="border-left"></span>
</div>
</body>
</html>

