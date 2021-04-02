Name
====

a little library for usting [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) in [aardio](http://www.aardio.com/)

Table of Contents
=================

* [Name](#name)
* [Dependent](#Dependent)
* [Methods](#Methods)
    * [open](#open)
    * [connect](#connect)
    * [waitEvent](#waitEvent)
    * [run](#run)
* [Example](#example)
	* [用脚本获取节点内容](#用脚本获取节点内容)

Dependent
=========

* [HP-Socket-aardio](https://github.com/btx638/HP-Socket-aardio)

Methods
=======

Example
=======

用脚本获取节点内容
------------------

![运行动画](https://raw.githubusercontent.com/btx638/dp/master/aaz/chrome/dp/example/3.gif)

````javascript
import win.ui;
/*DSG{{*/
var winform = win.form(text="aardio form";right=629;bottom=362)
winform.add(
button={cls="button";text="运行";left=528;top=290;right=617;bottom=326;z=1};
lvResult={cls="listview";left=9;top=6;right=621;bottom=278;edge=1;fullRow=1;z=2}
)
/*}}*/

import win.ui.statusbar;
import aaz.chrome.dp;

var cdp, err = aaz.chrome.dp()
if(!cdp){
    winform.msgboxErr(err);
	return ; 
}

var statusbar = win.ui.statusbar(winform);
statusbar.addItem("运行状态：", 70);
statusbar.addItem("", -1);

winform.lvResult.insertColumn("序号",50);
winform.lvResult.insertColumn("标题",-1);

var uiLog = function(str){
	statusbar.setText(str, 2);
}

var uiLogRunning = function(name){
    var str = string.format("正在%s...", name);
	uiLog(str);	
}

var uiLogFail = function(name, err){
    var str = string.format("%s失败，原因：%s", name, err);
	uiLog(str);
}


var uiResult = function(str){
	winform.lvResult.addItem({
		tostring(winform.lvResult.count+1);
		str
	})
}

var uiInit = function(){
	uiLog("任务开始", 2);
	winform.lvResult.clear();
}

var closeChrome = function(){
	return cdp.Browser.close(); 
}

var doFail = function(name, err){
    uiLogFail(name, err);
	closeChrome();
}


var task = function(){
    uiInit();
 	
 	uiLogRunning("打开浏览器");
	var ok, err = cdp.open(true); // 打开浏览器并开启 headless 模式，即不显示界面
    if(!ok){
        doFail("打开浏览器", err);
    	return ; 
    }
	
	uiLogRunning("连接浏览器");
    var ok, err = cdp.connect();
    if(!ok){
        doFail("连接浏览器", err);
    	return ; 
    }
    
	uiLogRunning("订阅 Page 事件");	
	var ok, err = cdp.Page.enable();
    if(!ok){
        doFail("订阅 Page 事件", err);
    	return ; 
    }
	
	// 打开网址
	uiLogRunning("打开网址 https://www.htmlayout.cn/");	
	var ok, err = cdp.Page.navigate(
		url = "https://www.htmlayout.cn/";
	)
    if(!ok){
        doFail("打开网址 https://www.htmlayout.cn/", err);
    	return ; 
    }
	
	uiLogRunning("等待页面加载完成");
	var ok, err = cdp.waitEvent( "Page.loadEventFired" );
    if(!ok){
        doFail("等待页面加载完成", err);
    	return ; 
    }
 	
	uiLogRunning("开启 runtime");
	var ok, err = cdp.Runtime.enable()
	if(!ok){
		doFail("开启 runtime", err);
		return ; 
	}
	
	uiLogRunning("编译脚本");
	var script, err = cdp.Runtime.compileScript(
		sourceURL = "https://www.htmlayout.cn/";
		persistScript = true;
		expression = /**
		 var getTitles = function(){
       		var doms = document.querySelectorAll(".home-box-list .post-list .item-content h2 a");
       		var ret = new Array();
       		for(var i=0;i<doms.length;i++){
           		ret.push(doms[i].innerText);
       		}
       		return ret; 
		 }
		**/
	)
	if(!script){
        doFail("编译脚本", err);
    	return ; 
	}
	
	uiLogRunning("运行脚本");
	var ret, err = cdp.Runtime.runScript(
		scriptId = 	script.scriptId;
		returnByValue = true;
	)
	if(!ret){
		doFail("运行脚本", err);
		return ; 
	}
	
	uiLogRunning("对全局对象的表达式求值");
	var ret, err = cdp.Runtime.evaluate(
		expression = "getTitles()";
		returnByValue = true;
	)
	if(!ret){
		doFail("对全局对象的表达式求值", err);
		return ; 
	}
	
	// 读取结果
	for(i=1;#ret.result.value;1){
		uiResult(ret.result.value[i]);
	}
	
	uiLogRunning("关闭浏览器");
	var ok, err = closeChrome();
	if(!ok){
		doFail("关闭浏览器", err);
		return ; 
	}
	
	uiLog("任务完成");
}

winform.button.oncommand = function(id,event){
	cdp.run(task)
}

winform.show();
win.loopMessage();
return winform;
````

[Back to TOC](#table-of-contents)