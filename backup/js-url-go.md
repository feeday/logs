通过判断识别终端自动跳转到指定的网址。

## 直接跳转代码
- 此代码插入网页代码中,当用户访问将自动跳转到指定的网址。

```
<script language="javascript" type="text/javascript">
window.location.href='http://cpuck.com';
</script>
```

## 移动端跳转代码
- 此代码插入PC网页代码中，当移动端用户访问页面将自动跳转到指定的网址。

```
<script type="text/javascript">
    function browserRedirect() {
        var sUserAgent = navigator.userAgent.toLowerCase();
        var bIsIphoneOs = sUserAgent.match(/iphone os/i) == "iphone os";
        var bIsMidp = sUserAgent.match(/midp/i) == "midp";
        var bIsUc7 = sUserAgent.match(/rv:1.2.3.4/i) == "rv:1.2.3.4";
        var bIsUc = sUserAgent.match(/ucweb/i) == "ucweb";
        var bIsAndroid = sUserAgent.match(/android/i) == "android";
        var bIsCE = sUserAgent.match(/windows ce/i) == "windows ce";
        var bIsWM = sUserAgent.match(/windows mobile/i) == "windows mobile";
        if (bIsIphoneOs || bIsMidp || bIsUc7 || bIsUc || bIsAndroid || bIsCE || bIsWM) {
            window.location.href = 'http://cpuck.com/';
        } else {
        }
    }
    browserRedirect();
</script>
```

## 移动端跳转代码
- 此代码插入PC网页代码中，当移动端用户访问将自动跳转到指定的网址。

```
<script language='javascript'>
function is_mobile() {
 var regex_match = /(nokia|iphone|android|motorola|^mot-|softbank|foma|docomo|kddi|up.browser|up.link|htc|dopod|blazer|netfront|helio|hosin|huawei|novarra|CoolPad|webos|techfaith|palmsource|blackberry|alcatel|amoi|ktouch|nexian|samsung|^sam-|s[cg]h|^lge|ericsson|philips|sagem|wellcom|bunjalloo|maui|symbian|smartphone|midp|wap|phone|windows ce|iemobile|^spice|^bird|^zte-|longcos|pantech|gionee|^sie-|portalmmm|jigs browser|hiptop|^benq|haier|^lct|operas*mobi|opera*mini|320x320|240x320|176x220)/i;
 var u = navigator.userAgent;
 if (null == u) {
 return true;
 }
 var result = regex_match.exec(u);
 if (null == result) {
 return false
 } else {
 return true
 }
 }
 if (is_mobile()) {
 document.location.href= 'http://cpuck.com/';
 }
</script>
```

## 微信端跳转代码
- 此代码插入网页代码中，当用户不是通过微信端打开的网页将跳转到指定的网址

```
<script type="text/javascript">
 var ua = navigator.userAgent.toLowerCase();
 var isWeixin = ua.indexOf('micromessenger') != -1;
 var isAndroid = ua.indexOf('android') != -1;
 var isIos = (ua.indexOf('iphone') != -1) || (ua.indexOf('ipad') != -1);
 if (!isWeixin) {
 document.location.href= 'http://cpuck.com/';
 }
</script>
```

## 随机跳转代码
- 此代码插入网页代码中，用户访问时候将随机跳转到设定好的网址。

```
<script language='javascript'>
 function test(){
	var url=new Array();
	url[0]='http://cpuck.com/';
	url[1]='http://cpuck.com/';
	url[2]='http://cpuck.com/';
	var ints=parseInt(Math.random()*(url.length));
	// window.open(url[ints]);//本窗口打开
	window.location=url[ints];//新窗口打开
	}
 </script>
<script type='text/javascript'>var _url = 'http://'+test()+'/';</script>
```

## 电脑移动端分别跳转
- 此代码插入网页代码中，电脑端和移动端用户访问时候分别跳转到设定好的网址。

```
<script language="javascript" type="text/javascript">
function goPAGE() {
	if ((navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i))) {
		window.location.href="/mobile.html";
}
	else {
		window.location.href="/pc.html";	}
}
goPAGE();
</script>
```

## 某宝某京东跳转
- 说明：微信访问自动跳转到指定链接，非微信端访问跳转到其他随机链接。

```
<script type="text/javascript">
            var ua = navigator.userAgent.toLowerCase();
            var isWeixin = ua.indexOf('micromessenger') != -1;
            var isAndroid = ua.indexOf('android') != -1;
            var isIos = (ua.indexOf('iphone') != -1) || (ua.indexOf('ipad') != -1);
            if (isWeixin) {
                 document.location.href= 'http://cpuck.com;
            }
</script>
<script language='javascript'>
 function test(){
	var url=new Array();
	url[0]='http://cpuck.com/';
	url[1]='http://cpuck.com/';
	var ints=parseInt(Math.random()*(url.length));
	window.location=url[ints];
	}
 </script>
<script type='text/javascript'>var _url = 'http://'+test()+'/';</script>
```