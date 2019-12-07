---
title: 百度统计api
date: 2019-12-07 15:29:59
tags:
---

## 如何给网站加入百度统计 api

**功能目标：页面访问量和下载次数统计**

#### 一、通过百度账号创建百度统计

- 新增网站
- 代码安装，添加到了 index.html 模板
  ![image.png](http://static.vue-js.com/FhhIomJVE30CBg8BXiPICXm88r5n)
  <br>
- 在 main.js 中，切换路由时的统计
  ![image.png](http://static.vue-js.com/FgNjZ7QbxCQi_DuhGyOZ-xTQ4Uno)
  <br>
- 至此，账号中已经开始有报告的数据
  ![image.png](http://static.vue-js.com/Frmsc-jdiiUPWUaVhSD4I2CpOnSn)
  <br>
- 数据导出服务

1. 百度开发者中心控制台，账号创建工程，记录 API Key 和 Secret Key
2. 手动获取 accessToken 和 refresh Token

   - 输入网址如下获取，authorization code  
      `http://openapi.baidu.com/oauth/2.0/authorize?response_type=code&client_id={API KEY}&redirect_uri=oob&scope=basic&display=popup`
     <br>
     ![image.png](http://static.vue-js.com/FkucwulczyUQArJKeVYSdDZu4bPe)
     <br>
   - 通过 authorization code 获取 token,输入网址如下
     `http://openapi.baidu.com/oauth/2.0/token?grant_type=authorization_code&code={CODE}&client_id={CLIENT_ID}&client_secret={CLIENT_SECRET}&redirect_uri=oob`
     <br>
     ![image.png](http://static.vue-js.com/FjO3exhXOek_2y2IlOaL-hWNWI1X)
     <br>

3. accessToken 和 refreshToken 交给服务端存储，在 accessToken 有效期到期前，刷新存储的凭证
4. 调用百度统计 api（这里写的固定的 token，因为功能还在测试）
   ![image.png](http://static.vue-js.com/FtDSNSUawVRkiNY_cMJVtepwHTG2)
   <br>
5. 报错如下
   ![image.png](http://static.vue-js.com/FnImJNqNiywgfIeg-y3LEAqW2e2f)
   <br>
   产生了 CORS 跨域问题，在开发模式下，我使用代理 devServer 之后是可以正常请求成功的
   ![image.png](http://static.vue-js.com/Fktj2E0c6NqPlY1kY1DxJM0l5yLz)
   <br>
   但是项目总归要上线的，npm run build 之后，还是会报错

- 转化分析数据
  在开发模式，使用代理请求的数据，metrics 添加了 **trans count **,但是返回的数据总数 sum 里依然没有事件转化的数据。（事件转化已经在按钮的 click 事件里面添加）
  请求返回数据如下：
  ![image.png](http://static.vue-js.com/Fo0O2pMhKQuAtybVZOdi90oLL0x6)
  <br>
  百度统计网站可以看到的转化次数如下
  ![image.png](http://static.vue-js.com/FueyXatKMsvqP-2Fkm5-TcJ_SIgC)
  <br>
  **<font color=red> 未解决问题：上面主要遇到的就是数据导出时，请求百度统计 api 时遇到跨域 cors 问题和请求数据的返回值中没有 trans_count </font>**
