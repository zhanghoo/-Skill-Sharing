# 使用HTML5里页面可见性接口判断用户是否正在观看你的页面
## HTML5里页面可见性接口中，可使用visibilitychange页面事件来判断当前页面可见性的状态，并针对性的执行某些任务。同时包括新的document.hidden属性。
### 可解决诸如此类的问题: 判断用户是否正在浏览某个指定标签页的方法。用户是否去看别的网站了？他们切换回来了么？手机中按了home推出也浏览器或微信时关闭音频播放等

## document.hidden
这个新出现的document.hidden属性，它显示页面是否为用户当前观看的页面，值为ture或false。

## document.visibilityState
visibilityState的值要么是visible (表明页面为浏览器当前激活tab，而且窗口不是最小化状态)，要么是hidden (页面不是当前激活tab页面，或者窗口最小化了。)，或者prerender (页面在重新生成，对用户不可见。).

## visibilitychange事件
监听页面可见性变化非常容易：
``` html
// 各种浏览器兼容
var hidden, state, visibilityChange; 
if (typeof document.hidden !== "undefined") {
	hidden = "hidden";
	visibilityChange = "visibilitychange";
	state = "visibilityState";
} else if (typeof document.mozHidden !== "undefined") {
	hidden = "mozHidden";
	visibilityChange = "mozvisibilitychange";
	state = "mozVisibilityState";
} else if (typeof document.msHidden !== "undefined") {
	hidden = "msHidden";
	visibilityChange = "msvisibilitychange";
	state = "msVisibilityState";
} else if (typeof document.webkitHidden !== "undefined") {
	hidden = "webkitHidden";
	visibilityChange = "webkitvisibilitychange";
	state = "webkitVisibilityState";
}

// 添加监听器，在title里显示状态变化
document.addEventListener(visibilityChange, function() {
	document.title = document[state];
}, false);

// 初始化
document.title = document[state];
```
使用该事件场景可以是:使用visibilitychange事件呢？如果你的页面上有嵌入视频正在播放，当用户切换到其它标签页时，你的标签页上的视频应自动暂停播放，当用户切换回来时继续接着播放。如果你的页面有自动刷新动作，当用户切换到其它标签页时，你就应该停止刷新，而当用户切换回来时继续之前的动作。
