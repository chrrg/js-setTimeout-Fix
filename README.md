# js-setTimeout-Fix
JavaScript 跨域调用时setTimeout和setInterval失效的解决方案

# 原因
出现此问题的原因：浏览器安全因素。
仅跨域调用时会出现，不跨域不会出现此情况。
例如：
index.html
```js
function callback(fn){
  setTimeout(function(){
    //执行到这里的时候domain已经设置为了跨域
    fn();//调用来自跨域的函数
  },100);
}
```

iframe.html:
```js
top.callback(function(){
  console.log("这里会执行！");
  setTiemout(function(){//由于跨域了，setTiemout失效
    console.log("这里不会执行！");
  },100);
  console.log("这里会执行！");
});
window.domain="xx.cn";
```


# 解决方案：
## setTimeoutFix.js
```js
(function(){
	var _setTimeout=setTimeout;
	var _clearTimeout=clearTimeout;
	var _setInterval=setInterval;
	var _clearInterval=clearInterval;
	var list=[];
	setTimeout=function(fn,time=0){
		var n=list.length;
		requestAnimationFrame(function(){
			if(list[n]===false)return;
			list[n]=_setTimeout(fn,time);
		});
		return n+1;
	};
	clearTimeout=function(id){
		_clearTimeout(list[id-1]);
		list[id-1]=false;
	};
	setInterval=function(fn,time=0){
		var n=list.length;
		requestAnimationFrame(function(){
			if(list[n]===false)return;
			list[n]=_setInterval(fn,time);
		});
		return n+1;
	};
	clearInterval=function(id){
		_clearInterval(list[id-1]);
		list[id-1]=false;
	};
})();
```
## setTimeoutFix.min.js
```js
(function(){var a=setTimeout;var b=clearTimeout;var c=setInterval;var d=clearInterval;var l=[];setTimeout=function(e,f){var n=l.length;requestAnimationFrame(function(){if(l[n]===!1)return;l[n]=a(e,f);});return n+1;};clearTimeout=function(g){b(l[g-1]);l[g-1]=!1;};setInterval=function(e,f){var n=l.length;requestAnimationFrame(function(){if(l[n]===!1)return;l[n]=c(e,f);});return n+1;};clearInterval=function(g){d(l[g-1]);l[g-1]=!1;};})();
```

# 原理
用js重写setTimeout、clearTmeout、setInterval、clearInterval代理原生的函数。
使用requestAnimationFrame嵌套即可。
