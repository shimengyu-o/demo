# 关于封装框架总结
## 常见的框架
  * 1、常见的框架
    学习前端的都知道，前端有很多的丰富的框架。例如：通用框架：jQuery、bootstarp、zepto...;ui框架：bootstar、layerui、amazeui...MVC框架:angular.js、react.js、veu.js...我们不仅仅要知道怎么用，还需要知道为什么这样用。‘知其然，知其所以然’。
  * 2、以jQuery整体分析：
        `(function(global,factory){
        /*
        *
        *
        */
        })(window,function(global,factory){
          //工厂模式
        function jQuery(){
        return  new jQuery.fn.init(selector,context);
        }
        jQuery.fn = jQuery.prototype ={
        constructor: jQuery
        }
        //为什么jQuery在使用的时候 可以使用  $ 和jQuery
        //结合 jQuery构造函数代码，我们得知jQuery对外暴露的方法是init方法
        //所以 整个jQuery的函数入口是init()
        window.jQuery = window.$ = jQuery;
        })
        })`
  * 3、jQuery封装内容
      **事件对象、选择器对象、属性对象、样式、内容...**
## 事件框架
### 事件流
  * 1、页面触发事件的三种方式
      行内执行
      行内调用
      根据id或类定义函数
      *行内事件冒泡机制*
      无论是DOM标准还是IE,直接写在html里的监听处理函数是事件冒泡传递时调用的,由最里层一直往上传递
  * 2、事件冒泡和捕获的区别
  * 3、IE和标准浏览器的绑定事件
      标准: addeventListener
      IE:attachEvent
  * 4、对象形式封装绑定事件
      (a)  On跨浏览器封装
      Event.on(‘div1’,’click’,fn);
        `on:function(id,type,fn){
          var ele = this.$id(id);
          if(ele.addEventListener){
            ele.addEventListener(type,fn,false)
          }else{
            ele.attchEvent('on'+type,fn);
          }
        }`
      b)	单独事件封装
      Event.click();
        `click:function(id,fn){
          this.on(id,'click',fn);
        }`
      Mouseover
      Mouseout
      Hover 模仿jquery
      c) 面向对象 绑定到xx域下
      d) 通过自定义extend方法实现
  * 4、event对象
      (1) IE和标准浏览器的区别
      兼容写法： `var e = e || window.event`
      (2)框架中获取事件对象
      (3)获取目标元素
      IE: e.srcElement
      标准：e.target
  * 5、阻止默认行为
      **兼容问题**
      标准：e.preventDefault();
      IE: window.event.returnValue = false;
  * 6、阻止冒泡
    **兼容问题**
    标准： event.stopPropagation();
    IE： event.cancelBubble = true;
  * 7、事件委托
    可以实现向未来元素绑定事件
        `delegate:function(p,target,type,fn){
          var targetEle = null;
          this.on(p,type,function(e){
            //目标元素
            targetEle = $$.getTarget(e);
            //给目标元素添加事件
            targetEle['on'+type] = function(e){
              // 通过call动态改变当前目标元素为触发事件的对象
                fn.call(this);
                $$.preventBubble(e);
            };
          targetEle[type]();
          })
        }
        });`
## 选择器的封装
### 基本选择器
    * id选择器
      `$id:function(id){
    return document.getElementById(id);
  }`
    * tag
      `$tagName:function(tName,context){
      //如果存在上下文对象 则使用上下文对象  如果不存在 则使用document对象

      var ctx = context?context:document;
      return ctx.getElementsByTagName(tName);
    },`
    *getElementsByTagName  结果是伪数组。
    跟数组一样有length属性，而且能通过下标获取相应位置的值，但不具备数组常有的功能，比如push，sort等*
    * class
          `$class:function(cname,context){
          var rs = [];
          var tagAll = this.$tagName('*',context);
          for(var i = 0 ; i < tagAll.length ; i++){
            var classStr = tagAll[i].getAttribute('class');
            if(!classStr) continue;
            var classArr = this.trim(classStr).split(' ');
            for(var j = 0 ; j < classArr.length ;j ++){
              //如果遍历到的标签 存在cname的样式 则添加
                if(classArr[j] === cname){
              rs.push(tagAll[i]);
              //  div div1 red green  r1 r2 r3 r4...
              break;
                }
              }
            }
            return rs;
          },`
### 多组选择器
    * 简单的多组
      例如：(‘.div1,.div2,#div3’)
### 层次选择器
    * 简单的层次
      例如：比如：(‘.div1  #div2  p’);
### 多组+层次
    例如：(‘.div1 #div2,div3 p #p1’)
### 复杂选择器
    sizzle.js
    jquery的实现
### h5选择器
    Document.querySelector(‘’);
    Document.querySelectorAll(‘’);
### css样式框架
          `css:function(selector,key,value){
          var dom = this.domAllStrict(selector);
          if(value){  //设值
            for(var i = 0 ; i < dom.length ; i++){
              dom[i].style[key] = value;
            }
          }else{  //如果是获取值 跟jq一样 只返回第一个
            //dom[0].style.color ==> 只能取行内样式
            if(dom[0].currentStyle){
              return dom[0].currentStyle[key];
            }else{
              return window.getComputedStyle(dom[0],null)[key];
              }
              }
            }`
## Bom 的封装
  * 标签的宽高
    + 元素的高度与宽度
      Width+padding+border --->offset
      Width+padding --->client
      content --->scroll
  * 浏览器的宽高
    * 可视区域宽高
      + document.body.clientHeight (document.documentElement )  (ie和firfox都能用)
      + window.innerHeight  (firefox能用)
      + 推荐用innerHeight  (忽略滚动条的宽度)
      + 兼容写法： window.innerWidth||this.clientWidth
   * 屏幕宽高
      + window.screen.height
   * 文档宽高
      + document.body.scrollHeight
   * 文档偏移量
      + var scrollTop = window.pageYOffset|| document.documentElement.scrollTop || document.body.scrollTop;
  * event 对象
      + event
        - 兼容获取  var event = e||window.event;
        - pageX/pageY 光标相对于网页的坐标
        - screenX/screenY  相对于屏幕的坐标
        - clientX/clientY 相对于浏览器左上角的坐标
      + pageX/pageY
        - ie不兼容
        - pageY = clientY + div.scrollTop
## 属性封装
  * Attr
  * addClass
  * removeClass
  * hasClass

## 内容的封装
  * html
        `html:function(selector,html){
        var dom = this.domAllStrict(selector);
        if(html){   //设值
          for(var i = 0 ; i < dom.length ;i++){
            dom[i].innerHTML = html;
          }
        }else{//取值
          return dom[0].innerHTML;
        }
        }`
  * text  步骤同html
## ajax封装
        `  ajax:function(json,callBack){
        var xhr = new XMLHttpRequest();
        var method =json.method;//get还是post
        var url =json.url;
        var param = null;  //参数
        xhr.open(method,url);
        if(method.toLowerCase() === 'post'){
        for(var i in json.param){
            param += '$'+i+'='+json.param[i];
        }
        param = param?param.substr(1):null;
          }
          xhr.send(param);
          xhr.onreadystatechange = function(){
        // 404   目标没找到
        // 500    服务器代码错误
        // 200    返回成功
        // 3xx    中转 ， 重定向！
        if(xhr.readyState == 4 && xhr.status == 200){
            var data;
            if(json.type.toLowerCase() === 'xml'){
                data = xhr.responseXML;
            }else{
                data = xhr.responseText;
            }
            if(callBack){
              callBack(data);
            }
          }
        }
        },`
## 动画封装
