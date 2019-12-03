---
layout: _post
title: js获取客户端ip地址（RTCPeerConnection）
date: 2019-11-22 10:39:35
tags:
---

## js 获取客户端 ip 地址 （RTCPeerConnection）

> 获取客户端内网 ip （类似 192.168.xxx.xxx)，根据浏览器的支持使用方法。

1. 仅支持 IE 且允许 Activex 运行，利用 ActiveObject 来获取。
2. 利用平台接口：新浪，太平洋等接口
3. 使用 WebRTC(Web Real-Time Communications),实时通信连接，它允许网络应用或者站点，在不借助中间媒介的情况下，建立浏览器之间点对点（Peer-to-Peer）的连接，实现视频流和（或）音频流或者其他任意数据的传输。WebRTC 可以获取 ip 地址，经测试，在 chrome，opera，firefox，safari 均可正常获取，对于 IE 和 Edge，可以采用第一种方式进行兼容。

##### 主流浏览器 chrome，opera，firefox，safari

```html
<body>
  <button id="handle">点击获取IP</button>
  <p id="yourIp"></p>
  <script text="text/javascript">
    document.getElementById("handle").onclick = function getYourIp() {
      var RTCPeerConnection =
        window.RTCPeerConnection ||
        window.webkitRTCPeerConnection ||
        window.mozRTCPeerConnection;
      //   定义构造函数RTCPeerConnection
      if (RTCPeerConnection)
        // 匿名函数自动执行,封装作用域
        (function() {
          //   创建rtc实例（配置未填写）
          var rtc = new RTCPeerConnection({ iceServers: [] });
          if (1 || window.mozRTCPeerConnection) {
            // 创建与远端的通信渠道
            rtc.createDataChannel("", { reliable: false });
          }
          // 事件监控本地向远端发送信息的icecandidate
          rtc.onicecandidate = function(evt) {
            if (evt.candidate) grepSDP("a=" + evt.candidate.candidate);
          };
          // 新建或更新 初始化会话信息通道
          rtc.createOffer(
            function(offerDesc) {
              grepSDP(offerDesc.sdp);
              // 更新通道信息链接下的本地描述信息
              rtc.setLocalDescription(offerDesc);
            },
            function(e) {
              console.warn("offer failed", e);
            }
          );

          var addrs = Object.create(null);
          addrs["0.0.0.0"] = false;
          // 更新展示地址的信息
          function updateDisplay(newAddr) {
            // 返回地址是0.0.0.0时不展示，直接跳出
            if (newAddr in addrs) return;
            else addrs[newAddr] = true;
            // Object.keys返回对象可枚举的自身属性字符串数组
            var displayAddrs = Object.keys(addrs).filter(k => addrs[k]);
            // 如果返回的展示地址格式不正确，从数组中删除
            for (var i = 0; i < displayAddrs.length; i++) {
              if (displayAddrs[i].length > 16) {
                displayAddrs.splice(i, 1);
                i--;
              }
            }
            // 展示结果处
            document.getElementById("yourIp").innerText = displayAddrs[0];
          }
          function grepSDP(sdp) {
            var hosts = [];
            // 正则表达式将传入的字符串分割（回车符换行符）成数组
            sdp.split("\r\n").forEach(function(line, index, arr) {
              // ~按位取反运算符，因为-1按位取反是0，正好用来处理indexOf返回值的-1与非-1，并进行判断
              if (~line.indexOf("a=candidate")) {
                // onicecandidate监控返回数据处理
                var parts = line.split(" "),
                  addr = parts[4],
                  type = parts[7];
                // 获取地址类型是host 则更新显示信息
                if (type === "host") updateDisplay(addr);
              } else if (~line.indexOf("c=")) {
                // createOffer建立通道返回数据处理
                var parts = line.split(" "),
                  addr = parts[2];
                updateDisplay(addr);
              }
            });
          }
        })();
      else {
        alert("请使用主流浏览器：chrome,firefox,opera,safari");
      }
    };
  </script>
</body>
```

##### IE 浏览器需要允许 ActiveX 控件

###### 主要应用知识点

- IE 控件的事件、属性、使用条件
- 客户端获取浏览器信息

```html
<body>
  <!--注入ActiveX控件-->
  <object
    id="locator"
    classid="CLSID:76A64158-CB41-11D1-8B02-00600806D9B6"
    VIEWASTEXT
    style="display:none;visibility:hidden"
  ></object>
  <object
    id="foo"
    classid="CLSID:75718C9A-F029-11d1-A1AC-00C04FB6C223"
    style="display:none;visibility:hidden"
  ></object>
  <!-- 控件安装并允许时事件 -->
  <script
    language="JScript"
    event="OnObjectReady(objObject,objAsyncContext)"
    for="foo"
  >
    var ipv4;
    if (
      objObject.IPEnabled != null &&
      objObject.IPEnabled != "undefined" &&
      objObject.IPEnabled == true
    ) {
      if (
        objObject.IPEnabled &&
        objObject.IPAddress(0) != null &&
        objObject.IPAddress(0) != "undefined"
      )
        ipv4 = objObject.IPAddress(0);
      console.log(ipv4);
    }
  </script>

  <button id="handle">点击获取IP</button>
  <p id="yourIp"></p>
  <script text="text/javascript">
    document.getElementById("handle").onclick = function getUserIP() {
      var myBrowserInfo = myBrowser();
      //判断浏览器是否是IE
      if (myBrowserInfo.indexOf("IE") > -1) {
        var service = locator.ConnectServer();
        service.Security_.ImpersonationLevel = 3;
        service.InstancesOfAsync(foo, "Win32_NetworkAdapterConfiguration");
        // 展示结果处
        document.getElementById("yourIp").innerText = ipv4;
      } else {
        //获取客户端本地IP地址（不等于IE浏览器）
        getYourIp(); //同主流浏览器 chrome，opera，firefox，safari,调用上面写的函数即可
      }
    };
    //浏览器判断
    function myBrowser() {
      var userAgent = navigator.userAgent; //取得浏览器的userAgent字符串
      console.log(userAgent);
      var isOpera = userAgent.indexOf("Opera") > -1; //判断是否Opera浏览器

      var isIE11 =
        userAgent.toLowerCase().indexOf("trident") > -1 &&
        userAgent.indexOf("rv") > -1;
      if (isIE11) {
        var reIE11 = new RegExp("rv:(\\d+\\.\\d+)");
        reIE11.test(userAgent);
        var fIEVersion = parseFloat(RegExp["$1"]);
        console.log(fIEVersion);

        if (fIEVersion == 11) {
          return "IE11";
        }
      }
      var isIE =
        userAgent.indexOf("compatible") > -1 &&
        userAgent.indexOf("MSIE") > -1 &&
        !isOpera; //判断是否IE浏览器
      var isEdge = userAgent.indexOf("Edge") > -1; //判断是否IE的Edge浏览器
      var isFF = userAgent.indexOf("Firefox") > -1; //判断是否Firefox浏览器
      var isSafari =
        userAgent.indexOf("Safari") > -1 && userAgent.indexOf("Chrome") == -1; //判断是否Safari浏览器
      var isChrome =
        userAgent.indexOf("Chrome") > -1 && userAgent.indexOf("Safari") > -1; //判断Chrome浏览器

      if (isIE) {
        var reIE = new RegExp("MSIE (\\d+\\.\\d+);");
        reIE.test(userAgent);
        var fIEVersion = parseFloat(RegExp["$1"]);
        console.log(fIEVersion);

        if (fIEVersion == 7) {
          return "IE7";
        } else if (fIEVersion == 8) {
          return "IE8";
        } else if (fIEVersion == 9) {
          return "IE9";
        } else if (fIEVersion == 10) {
          return "IE10";
        } else {
          return "0";
        } //IE版本过低
        return "IE";
      }

      if (isOpera) {
        return "Opera";
      }
      if (isEdge) {
        return "Edge";
      }
      if (isFF) {
        return "FF";
      }
      if (isSafari) {
        return "Safari";
      }
      if (isChrome) {
        return "Chrome";
      }
    }
  </script>
</body>
```

<table><td bgcolor=#F56C6C  >IE浏览器的控件方法依然存在问题，没有跳出提示安装控件，onReady事件没有触发，locator组件属性,错误信息：找不到成员</td></table>
