> 2018年2月9日14:30:27

[toc]

## 工具说明
* Fiddler或者Chrome浏览器控制台

## 基础知识
* JS基础
* Http基础

## 实现步骤
**以天津银行举例：**

[天津银行个人网上银行登录](https://www.bankoftianjin.com/pweb/prelogin.do?LoginType=R&_locale=zh_CN)

![](http://p1hy9syru.bkt.clouddn.com/18-2-9/30090775.jpg)

### 天津银行登录分析
F12打开浏览器调试工具，点击source，找到登录的form表单
，如图：
![](http://p1hy9syru.bkt.clouddn.com/18-2-9/3184899.jpg)

Form表单代码如下：
```
 <form name="form2" action="#" method="post"  id="con2">
				<input id="_locale" name="_locale" type="hidden" value="zh_CN" />
				<input type="hidden" name="LoginType" value="R" />
		  		<input type="hidden" name="PreLoginMode" value="Login"/>
				<input type="hidden" name="TransName" value="LoginAll"/>
				<input type="hidden" id="HrefType" name="HrefType" value=""/>
					<div id="divLoginWindow" >
					    <div id="loginBox" class="login_login">
						    <div class="login_input">
			        			<div class="login_i_name"></div>
			        			<!-- value="用户昵称/账(卡)号/证件号 " -->
			       				<input type="text" name="LoginId"  id="LoginIdOrAc"   value="登录名/证件号" 
			       				style="width:150px;" onclick="this.value='';" 
			       				onblur="resetText(this,1);error(' ',this);" onkeydown="changeToNextTag();"/>
			      			</div>
						    <div class="login_input">
						        <div class="login_i_password"></div>
						        <span id="PPP1" class="login_input1"><script type="text/javascript">writePassObject("powerpass1",{"width":150,"height":24,"maxLength":20,"accepts":"[:graph:]+"},"Password");</script></span>
								<span id="MMM1"><script type="text/javascript">writeUtilObject("powerutil1",{"width":1,"height":1},"MachineCode");</script></span>
								
						        <!-- <a href="#">下载密码控件</a> -->
						    </div> 
						    
						    <div class="login_input">
						 <!--    error('请输入校验码，校验码不区分大小写',this); -->
						    	 <div class="login_i_code"></div>
						    	 <input type="text"  style="width:73px;font-size: 12px;"  name="_vTokenName" value="不区分大小写" id="_vToken1" onfocus="this.value='';" onblur="resetText(this,2);error(' ',this);" onkeyup="value=value.replace(/[\W]/g,'') " onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^\d]/g,''))" onkeydown="javascript:if(event.keyCode==13){form2.btnLogin2.focus();}"/>
						    	 <div style="_margin-left: 5px;"><img id="form2_tokenImg" name="_tokenImg" src="GenTokenImg.do" width="61" height="32"  onclick="reloadTokenImg(this.id);" alt="点击刷新" /></div>
				   				<!--  <div class="login_code"><label style="width:80px;"></label></div> -->
				   				<div style="_margin-left: 5px;"><b  
				   	  style="display: inline-block;width:80px;underline; margin-top:1px;height:24px; line-height:24px; vertical-align:middle;font-size: 12px;">点击图片刷新</b></div>
				   	 		</div>
						    
								
					        <div id="spTishi1" style="display:none;height:23px;text-align:center;margin-left:0px;margin-top:-8px;_margin-left:0px;">
					        	<p id="form2_EEE" class="font-size:13px;color:#ffdf27;padding-left:22px;line-height:23px;"></p>
					        </div>
					        <div id="spTishi2" class="login_error" style="display:none;">
					        	<p id="form5_EEE"></p>
					        </div>
					        <input type="button" id="btnLogin2" name="btnLogin2" onclick="doIt(null,document.getElementById('con2'));" class="login_logbtn" />
					        <!-- <a class="login_logbtn" href="#" id="btnLogin2" name="btnLogin2" onclick="doIt(null,document.getElementById('con2'));"></a> -->
					        <div class="login_links">
								<div class="login_l_left">
									<a href="#" onclick="register();">注册</a>
									<a href="#" onclick="ElecReceiptQryPre();">回单验证</a>
								</div>
								<div class="login_l_right">
								   <a href="#" onclick="ResetChannelUserName()">忘记登录名</a>&nbsp;
								   <a href="#" onclick="javascript:resetChannelPwd()">忘记登录密码</a>
								</div>
							</div>
					        
					  	</div>
					</div>			    
				</form>

```
可以看到，要发送到服务端的数据有以下几个：

```
//语言
<input id="_locale" name="_locale" type="hidden" value="zh_CN" />

//登录类型
<input type="hidden" name="LoginType" value="R" />

//不知道干嘛的
<input type="hidden" name="PreLoginMode" value="Login"/>

//交易名
<input type="hidden" name="TransName" value="LoginAll"/>

<input type="hidden" id="HrefType" name="HrefType" value=""/>

//登录名
<input type="text" name="LoginId"  id="LoginIdOrAc"   value="登录名/证件号" />

//验证码
<input type="text"  name="_vTokenName" value="不区分大小写" id="_vToken1" />
 
 //此为密码控件 获取的就是密码的值
<span id="PPP1" class="login_input1"><script type="text/javascript">writePassObject("powerpass1",{"width":150,"height":24,"maxLength":20,"accepts":"[:graph:]+"},"Password");</script></span>

//此为机器码控件  获取的是当前机器唯一的机器码
<span id="MMM1"><script type="text/javascript">writeUtilObject("powerutil1",{"width":1,"height":1},"MachineCode");</script></span>

//登录按钮
<input type="button" id="btnLogin2" name="btnLogin2" onclick="doIt(null,document.getElementById('con2'));" class="login_logbtn" />
```

看一下登录按钮的逻辑，其onclick()绑定了doIt()方法，看一下这个函数的逻辑，如下：
```
function  doIt(clickObj,vForm)
{
	
	var vErrorId = vForm.name+"_EEE";
	var errdiv=window.document.getElementById(vErrorId);
	if(document.getElementById('_vToken')!=null && document.getElementById('_vToken').className=="inputtxt"){
		document.getElementById('_vToken').value="";
	}
	if(document.getElementById('_vToken1')!=null && document.getElementById('_vToken1').className=="inputtxt"){
		document.getElementById('_vToken1').value="";
	}
	
		var password = getIBSPassword("powerpass1", ts , "form5_EEE", "密码至少需要六位")
		errdiv.style.color="#ffdf27";
		adJustDiv(window.document.getElementById("form5_EEE"))
		if(password==null)
			return false;
		vForm.Password.value=password;
	
	
			var mfm = getMFMInput("powerutil1", ts , "form5_EEE", "获取机器码失败")
			errdiv.style.color="#ffdf27";
			adJustDiv(errdiv)
			if(mfm==null)
				return false;
		vForm.MachineCode.value=mfm;
	 

	showErrDiv1(false);
	myCounter.init(vErrorId);
	myCounter.run();
	post2SRV('login.do',vForm,clickObj,null,  function(flag, answer){
		   clearTimeout(myCounter.timer);
		   if(!flag){
			   postData2SRVWithCallback("Timestamp.do", "", function(flag, answer){	if(flag){   ts = answer;}}	)
			   var vTokenImgObj = document.getElementsByName("_tokenImg");
			   for(var i=0;i<vTokenImgObj.length;i++){
				   var vImgId = vTokenImgObj[i].id;

				   if(vImgId.substring(0,vImgId.indexOf("_")) ==vForm.name ){
					   reloadTokenImg(vTokenImgObj[i].id);
					   break;
				   }
				  
			   }
			  
			showpeerrforlogin(flag,answer,true,vForm);
		   }
	});
}

```

这个函数的作用有两个：
1. 对输入的数据进行合法性判断
2. 将数据发送到login.do接口

大概分析了一下需要往服务端发送这些数据，那么我们登录一下，抓包分析是否正确。

### 抓包分析
打开Fiddler

到天津银行登录页面，输入账号名和密码、验证码后点登录。可以看到Fiddler已经抓到了数据，上面我们已经分析了天津银行将数据发送的接口为login.do，所以在Fiddler中直接找login.do的接口即可，如下图：

![](http://p1hy9syru.bkt.clouddn.com/18-2-9/42911388.jpg)

发送的数据如下：
```
POST https://www.bankoftianjin.com/pweb/login.do HTTP/1.1
Host: www.bankoftianjin.com
Connection: keep-alive
Content-Length: 711
Origin: https://www.bankoftianjin.com
PE-ENCODING: UTF-8
PE-AJAX: true
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.134 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Accept: */*
Referer: https://www.bankoftianjin.com/pweb/prelogin.do?LoginType=R&_locale=zh_CN
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8
Cookie: tccbHint1=N; tccbHint=N; BIGipServerpool_xwy_8443=2197881099.64288.0000; JSESSIONID=Wn12h9FZp2F2Qmd339cDmQjHv39VHn29GjMWfX19fnBQsl6hNDSx!35681400; TS0129b339=015b9dc47f67ea9cc3a144bc2c70805d1b3c6931b27172395e782df1859fc57f743a60215dd24592a1787dc7148bb7529cf20c344bad860a200e4dcca9c7bf643569ce1b62

_locale=zh_CN&LoginType=R&PreLoginMode=Login&TransName=LoginAll&HrefType=&LoginId=powerebank_31&Password=MDAwMDAxNDABAgAAAWgAAACkAACJ4NZN19oDSgX5qhBbrXad2y9SqZno%2FsQQeaynEAfhPqMpwW0rKKRDpeG57kghMRlFQYa%2BfNRZJOdifNq1X5uEqTmgSGpxxxQ5Zi8YobCUq%2FrzDfskYFP5EnPqbLVkraOYDUdN07gguplvc55ReX7VVoPxZGwjk%2FXtHgbLOcwwYjAwMDAwMDIzA8IojCiRBANdB%2FmeaZY02JsEaV9JOVQ%3D&MachineCode=MDAwMDAxNDABAgAAAWgAAACkAABPYtSJ33%2Bp%2BqtjPhblSIB7ChwHjA%2FUtkgf1ZTBpmScFNrBcnuTlKE1mGh1%2BsIhJXVv%2BF529Lly89rgsWeVI4YO2azS1NvxzTPLg%2Fs7oqyFTpAX5N0TE7%2FE9jO55CqWZNw3gYrKpTHRvYes9Uwm2sJCRdQHl5rL%2FoNPljbnolOebTAwMDAwMDY0Z3hu88V0wZK1SHNNEWEA%2FeKMPpSS%2Bhn8reYN%2FqhBsbA7QYFOaE9POlhOqSC7WToI%2B2zBl1oE3Gq1vGSQp9qvHg%3D%3D&_vTokenName=asgq
```
注意：这里有两个Http头信息很重要，就是PE-ENCODING: UTF-8 和 PE-AJAX: true

下面是表单的数据，很重要，分析一下：
```
_locale=zh_CN&LoginType=R&PreLoginMode=Login&TransName=LoginAll&HrefType=&LoginId=powerebank_31&Password=MDAwMDAxNDABAgAAAWgAAACkAACJ4NZN19oDSgX5qhBbrXad2y9SqZno%2FsQQeaynEAfhPqMpwW0rKKRDpeG57kghMRlFQYa%2BfNRZJOdifNq1X5uEqTmgSGpxxxQ5Zi8YobCUq%2FrzDfskYFP5EnPqbLVkraOYDUdN07gguplvc55ReX7VVoPxZGwjk%2FXtHgbLOcwwYjAwMDAwMDIzA8IojCiRBANdB%2FmeaZY02JsEaV9JOVQ%3D&MachineCode=MDAwMDAxNDABAgAAAWgAAACkAABPYtSJ33%2Bp%2BqtjPhblSIB7ChwHjA%2FUtkgf1ZTBpmScFNrBcnuTlKE1mGh1%2BsIhJXVv%2BF529Lly89rgsWeVI4YO2azS1NvxzTPLg%2Fs7oqyFTpAX5N0TE7%2FE9jO55CqWZNw3gYrKpTHRvYes9Uwm2sJCRdQHl5rL%2FoNPljbnolOebTAwMDAwMDY0Z3hu88V0wZK1SHNNEWEA%2FeKMPpSS%2Bhn8reYN%2FqhBsbA7QYFOaE9POlhOqSC7WToI%2B2zBl1oE3Gq1vGSQp9qvHg%3D%3D&_vTokenName=asgq
```
1. _localle=zh_CN 照抄就行
2. LoginType=R 代表个人网银登录
3. PreLoginMode=Login 照抄
4. TransName=LoginAll 照抄
5. LoginId=powerebank_31 用户名
6. Password=MDAwMDAxNDABAgAAAW.... 密码控件加密后的密码
7. MachineCode=MDAwMDAxNDABAgAAAWgA...... 机器码
8. _vTokenName=asgq 验证码

知道传输什么数据了，我们仿造一个试试

### 模仿数据发送

编写个Html文件，代码如下：
```
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<title>天津银行登录测试</title>
		<script language="javascript" src="https://www.bankoftianjin.com/pweb/js/writeObject.js"></script>
		<script src="http://libs.baidu.com/jquery/1.7.2/jquery.min.js"></script>
	</head>

	<body>
		账号：<input id="LoginId" /><br /> 密码：
		<span id="PPP1" class="login_input1"><script type="text/javascript">writePassObject("powerpass1",{"width":150,"height":24,"maxLength":20,"accepts":"[:graph:]+"},"Password");</script></span> <br /> 验证码：
		<input type="text" name="_vTokenName" placeholder="不区分大小写" id="_vToken1" /><br/> 验证码图片：
		<img id="form2_tokenImg" name="_tokenImg" src="https://www.bankoftianjin.com/pweb/GenTokenImg.do" width="61" height="32" alt="点击刷新" /></div>

		<!--机器码隐藏-->
		<span id="MMM1"><script type="text/javascript">writeUtilObject("powerutil1",{"width":1,"height":1},"MachineCode");</script></span><br />
		<input type="submit" value="登录" onclick="login()" />

		<!--输出参数信息 -->
		<div id="accountInfo"></div>

		<!--执行结果输出-->
		<div id="result">
		</div>

		<!--错误信息输出-->
		<div id="errorinfo" style="color: red;"></div>
	</body>

	<script>
		//获取当前时间戳
		var ts = Date.parse(new Date()) + '';
		/*个人用户登录的逻辑*/
		function login() {
			//获取账号名
			var loginIdVal = document.getElementById("LoginId").value;
			//获取密码
			var passwordVal = getIBSPassword("powerpass1", ts, "errorinfo", "密码输入错误");
			//获取机器码
			var mfmVal = getMFMInput("powerutil1", ts, "errorinfo", "获取机器码失败");
			//获取验证码
			var tokenVal = document.getElementById("_vToken1").value;

			document.getElementById("accountInfo").innerHTML = "账号：" + loginIdVal + "<br/>密码：" + passwordVal + "<br/>机器码:" + mfmVal + "<br/>验证码：" + tokenVal;

			//发送数据
			$.ajax({
				type: 'POST',
				url: "https://www.bankoftianjin.com/pweb/login.do",
				headers: {
					"PE-AJAX": true,
					"PE-ENCODING": "UTF-8"
				},
				//参数信息
				data: {
					LoginId: loginIdVal,
					Password: passwordVal,
					MachineCode: mfmVal,
					LoginType: "R",
					TransName: "LoginAll",
					HrefType: "",
					PreLoginMode: 'Login',
					_locale: "zh_CN",
					_vTokenName: tokenVal,
				},
				dataType: 'text',
				success: function(data) {
					//截取服务端返回的信息
					var loginStatusObj = getJSONObjFromData(data);
					//打印在控制台
					console.log(loginStatusObj);
					if(loginStatusObj != null) {
						//输出在dom中
						document.getElementById("result").innerHTML="<br/>服务端返回的信息："+loginStatusObj._exceptionMessage;
						return false;
					}
					document.getElementById("result").innerHTML="<br/>服务端返回的信息："+data;
				},
				error: function(result) {
					
				}
			});
		}

		/**
		 * 从一串文本中截取JSON格式的字符串，返回JSON对象
		 * @param {Object} str 
		 */
		function getJSONObjFromData(str) {
			if(/<PEAjaxError>\[(.*)\]<\/PEAjaxError>/.test(str)) {
				return JSON.parse(RegExp.$1);
			} else {
				return null;
			}
		}
	</script>

</html>
```


右键Chrome快捷方式，点击属性，在目标上添加上 --disable-web-security，禁用浏览器同源策略的限制，如下图：
![](http://p1hy9syru.bkt.clouddn.com/18-2-9/2498382.jpg)


然后用chrome访问上述的Html文件。

截图如下：
![](http://p1hy9syru.bkt.clouddn.com/18-2-9/47328141.jpg)



![](http://p1hy9syru.bkt.clouddn.com/18-2-9/57694511.jpg)

可以看到，我们成功的拿到了服务端返回的数据。





