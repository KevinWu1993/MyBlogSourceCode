---
title: DIY一个智能插座
date: 2016-06-24  21:22
author:  KevinWu
categories: 酷玩
tags: 
	- 智能
	- 硬件
	- 手工
	- 单片机
	- DIY
	- 插座
---
像网上随便搜都能搜到的各种物联网啊云啊的概念就不扯了，扯再多还不如自己动手接个线焊个锡写几行代码来的实在，所以这次没有背景知识，直入正题。

## 材料准备


材料 | 数量
-- | --
插线板（有人叫插座） | 1
ESP8266wifi芯片 | 1 
AC-DC电源转换模块（220V～5V）| 1
继电器 | 1
杜邦线 | 若干
USB转TTL小板 | 1


算上上面的东西，就算买个好点的插座，线粗点，也不用超过60块钱，比网上卖的智能插座便宜多了。

### 硬件改造

这一步纯手工活，需要有点用电烙铁和电路的基础，但这一步也要特别细心，220V的电源，如果不小心没焊接牢固或者焊接错了，短路可不是什么好玩的事哇。

可以上些图看看这个过程和成品（成平丑的像个定时炸弹是因为没有热熔胶，某些固定工作临时临急用黑胶布代替，当然作为第一个实验品也没考虑买个大点的插座把那些芯片什么的都塞到里面去）：

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/27477687.jpg)

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/17273104.jpg)

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/7623646.jpg)

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/19241771.jpg)

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/98569802.jpg)

### 总结一下改造步骤

1. 拆解插线板2. 从插线板的火线和零线引出电源到AC-DC电源转换器，电源转换器引出线，并联连接ESP8266芯片及继电器，提供5V工作电源3. 用万用表测试继电器短路情况4. 继电器与插线板入口火线串联接入5. 继电器连接ESP8266的GPIO控制端口

## 本地局域网控制的智能插座

可以先把这个的缺点讲一下，这是种仅仅做出来，但不实用的插座，这种插座只能通过手机控制端连接智能插座的wifi控制芯片来控制插座的开关，或者通过连接同一个无线路由器，通过设置转发的方式来控制。

> 原理很简单，就是Socket通信。



虽然原理很简单，而且我也不推荐这个方案，不过这个方案在我研究它的接口的时候却是最煎熬的，因为ESP8266的固件是已经编译好了的，单片机没文档，没固件源码也很难上手，所以想着用官方示例的控制APP来反编译把API找出来，然而像这类东西，大部分都是写单片机的程序员，安卓这边写的少，很多控制APP都是用易安卓搞出来的，反编译。。。不忍直视，最后只能抓包来找了。。。

因为官方没有接口文档，这也不是我主要推荐的方案，我就不补充文档了，如果需要仿照做的直接看我代码也能看出来。

直接开始撸代码。

当然，采用MVP框架来搭建的话，建模是少不了的，先建立灯光控制的模型（这个官方AT固件本来是控制三色灯的，只是我发现也能控制IO口）：

``` java


package cn.kevinwu.esp8266_controller.model;

import cn.kevinwu.esp8266_controller.dao.LightDAO;

/**

 * Created by KevinWu on 16-5-29.

 * KevinWu.cn

 */

public class LightModel implements IModel<LightModel> {
    private LightDAO dao;
    private int freq;
    private int io0;
    private int io1;
    private int io2;
    private int io4;
    private int io5;
    private int io12;
    private int io13;
    private int io14;
    private int io15;
    private RGBBean rgb;
    public LightModel(){
        this.dao=new LightDAO();
    }
    public LightModel(int freq,int io0,int io1,int io2,
                      int io4,int io5,int io12,int io13,
                      int io14,int io15,RGBBean rgb){
        this.freq=freq;
        this.io0=io0;
        this.io1=io1;
        this.io2=io2;
        this.io4=io4;
        this.io5=io5;
        this.io12=io12;
        this.io13=io13;
        this.io14=io14;
        this.io15=io15;
        this.rgb=rgb;
    }
    @Override

    public void loadingNet() {

        dao.loadingNet();

    }
    @Override
    public void postingNet(LightModel model) {
        dao.postingNet(model);
    }
    public int getFreq() {
        return freq;
    }
    public void setFreq(int freq) {
        this.freq = freq;
    }
    public int getIo0() {
        return io0;
    }
    public void setIo0(int io0) {
        this.io0 = io0;
    }
    public int getIo1() {
        return io1;
    }
    public void setIo1(int io1) {
        this.io1 = io1;
    }
    public int getIo2() {
        return io2;
    }
    public void setIo2(int io2) {
        this.io2 = io2;
    }
    public int getIo4() {
        return io4;
    }
    public void setIo4(int io4) {
        this.io4 = io4;
    }
    public int getIo5() {
        return io5;
    }
    public void setIo5(int io5) {
        this.io5 = io5;
    }
    public int getIo12() {
        return io12;
    }
    public void setIo12(int io12) {
        this.io12 = io12;
    }
    public int getIo13() {
        return io13;
    }
    public void setIo13(int io13) {
        this.io13 = io13;
    }
    public int getIo14() {
        return io14;
    }
    public void setIo14(int io14) {
        this.io14 = io14;
    }
    public int getIo15() {
        return io15;
    }
    public void setIo15(int io15) {
        this.io15 = io15;
    }
    public RGBBean getRgb() {
        return rgb;
    }
    public void setRgb(RGBBean rgb) {
        this.rgb = rgb;
    }
}

```

虽然不需要数据库，不过数据访问对象习惯性用来操作数据，如下：

``` java


package cn.kevinwu.esp8266_controller.dao;

import android.os.Handler;
import android.os.Looper;
import android.util.Log;
import com.squareup.okhttp.Headers;
import org.greenrobot.eventbus.EventBus;
import cn.kevinwu.esp8266_controller.api.Esp8266API;
import cn.kevinwu.esp8266_controller.event.E;
import cn.kevinwu.esp8266_controller.event.EventModel;
import cn.kevinwu.esp8266_controller.model.LightModel;
import cn.kevinwu.esp8266_controller.netutil.MyNet;
import cn.kevinwu.esp8266_controller.netutil.callback.StringCallback;


/**

 * Created by KevinWu on 16-5-29.

 * KevinWu.cn

 */

public class LightDAO implements IDAO<LightModel>{

    public static final String TAG="LightDAO";

    @Override

    public void loadingNet() {
        final Handler handler=new Handler(Looper.getMainLooper());
        MyNet.get(Esp8266API.Light_Url)
        .enqueue(new StringCallback() {
            @Override
            public void onSuccess(String result, int responseCode, Headers headers) {
                Log.d(TAG,"responseCode is "+responseCode);
                Log.d(TAG,"result is "+ result);
                    handler.post(new Runnable() {
                        @Override
                        public void run() {
                            EventBus.getDefault().post(new EventModel<Void>(E.GET_LIGHT_STATUS_SUCCESS));
                        }
                    });
            }

            @Override
            public void onFailure(String error) {
            }
        });
    }

    @Override
    public void postingNet(LightModel model) {
        final Handler handler=new Handler(Looper.getMainLooper());
        MyNet.postJson(Esp8266API.Light_Url)
                .addJsonObject(model)
                .enqueue(new StringCallback() {
                    @Override
                    public void onSuccess(String result, int responseCode, Headers headers) {
                        Log.d(TAG,"response code is "+responseCode);
                        Log.d(TAG,"post back data is "+result);
                        handler.post(new Runnable() {
                            @Override
                            public void run() {
                                EventBus.getDefault().post(new EventModel<Void>(E.POST_LIGHT_STATUS_SUCCESS));
                            }
                        });
                    }

                    @Override
                    public void onFailure(String error) {
                        handler.post(new Runnable() {
                            @Override
                            public void run() {
                                EventBus.getDefault().post(new EventModel<Void>(E.POST_LIGHT_STATUS_FAILURE));
                            }
                        });
                    }
                });
    }

}

```

网络访问用的是OKHTTP，经过了简单的封装，通信用的是EventBus，至于回调数据，其实是有json返回数据的，因为根据返回码的不同也能知道操作结果如何了，所以就直接取String了。

而对应Socket通信的url如下：

``` java
public static final String Light_Url="http://192.168.4.1/config?command=light";
```

这种方案写出来的控制APP（没考虑UI）：

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/36445265.jpg?imageMogr2/thumbnail/400x400)

到这里，这种方案已经很明确了，再次强调，不推荐这种方案，如果真的感兴趣可以查看完整项目源代码：

https://github.com/KevinWu1993/ESP8266-Controller





## 基于ESPUSH云服务的的智能插座

ESPUSH是一个物联网云开发平台，而且是免费提供使用的，目前提供开发版硬件出售（不是打广告），挺适合初学者研究下的。

对于ESPUSH，这里就不做过多介绍了，有兴趣的可以自行去官网看看：

https://espush.cn/

进入正题，在使用这个云开发平台前，你同样需要先动手连接好插座，然后到ESPUSH平台注册一个账号，然后添加一个硬件设备，这个流程没什么需要注意的，不细说。

因为这里只是用到GPIO的状态接口和控制接口，所以节选部分ESPUSH官方接口文档如下：



---

### 协议的整体描述

请求URL结构为：**http://接口域名/openapi/接口命令/?接口参数**，其中各字段取值如下：


- 接口域名，域名，当前取值espush.cn 使用https://协议
- openapi，固定取值，区分主站资源
- 接口命令，接口命令资源地址，详见下文
- 接口参数，使用HTTP GET方式传递的参数，包括通用参数、接口指定参数等
### 通用参数

各接口URL都需要在URL中传递通用参数，参数列表简单摘要如下：


字段 | 值类型 | 必填项 | 示例
-|-|-|-
appid | 整型 | 是 | 在https://espush.cn/webv2/apps/中进行管理
timestamp | 整型 |是 | 形如1434447718
sign | 32位字符串 | 是 | 生成方式参考下文




说明：


- **espush.cn的服务端API并不认证用户名与密码，只针对APPID与APPKEY，所以保护你的APPKEY，切勿泄露。**
- 请求请定义您自己的UserAgent，目前平台无限制，但为了做简要区分，请尽量不要使用任何带有robot字样的UserAgent。
- 目前接口的调用频次无限制，但针对部分请求，频繁调用可能会导致设备重置。
### 通用返回结果

- 接口使用非常简单的HTTP JSON API形式，成功的接口依据内容的不同返回内容也有不同，但都有共同的字段 msg，用于表明此次请求的返回说明，成功的请求使用200作为其返回码.
- 使用HTTP返回码，如 5XX代表服务端的错误，特别的，当与设备连接的网关服务器出错时，多返回502或504，并在msg中告知具体原因。4XX代表客户端的错误，通常401为sign取值错误，400是参数缺失，404是对应资源找不到。
- 服务端要求任何资源的请求末尾字符为 /，？后的GET参数不计在内，如不应该请求https://espush.cn/openapi/apps?timestamp=1466598336&sign=a34e0360a05c739a08dd9ab404aec66b&appid=1234而是应该发出https://espush.cn/openapi/apps/?timestamp=1466598336&sign=a34e0360a05c739a08dd9ab404aec66b&appid=1234这样的请求，区别在于 apps 后是否有结束符 /，前者将会收到一个301的重定向返回。
### 签名认证方式


内容签名sign的计算方法如下：

- 提取请求的方法本身的小写字母表示，如 get, post等，记作字符串A
- 提取请求的其他参数（GET与POST参数均需要，COOKIES参数不计入）但不包括sign，并按Key-Value的形式并转换为小写字母表示（未做urlencode），再按Key的降序排列（排序时，Value未参与，只对Key的小写字母表示做字母表的降序排列），组成如下形式：*key3=v3&key2=v4&key1=v1*，此处记作字符串B
- 提取APPKEY，记字符串C
- Sign的值为 S = lower(MD5(A+B+C))
如POST请求，现有如下信息：APPID为1234，APPKEY为25b28f0ffb9711e4a96d446d579b49a1，无其他额外的请求参数，则只有通用参数appid、timestamp与sign，则字符串S应为:**gettimestamp=1433814203&appid=123425b28f0ffb9711e4a96d446d579b49a1**，对其进行MD5运算，取运算后的**小写字母**表示，即得到最终sign值:

**9f0b613de12d5bb0451c556900a39559**



### 接口详细定义

#### 获取模块各GPIO口简要信息

> 

接口地址：gpio_status/:chipid/


- 功能说明，获取设备GPIO口电平状态，此API与下一接口，主要用于首页App，远控之用。
- 请求参数，无，使用通用参数
- 请求方法，GET
- 请求示例

> curl "https://espush.cn/openapi/gpio_status/8454703/?timestamp=1466642946&sign=4d37cfc0233fc13b4858f0a35a48926a&appid=1234"
- 返回示例

> {"result": "\u0001\u0000\u0001\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000"}


获取模块各GPIO口的状态

#### 简单设置模块GPIO电平接口

> 

接口地址：set_gpio_edge/:chipid/:pin/:edge/


- 功能说明，设置指定模块的GPIO口电平态，本处只限简单使用，输出高低电平，内部未做任何上拉使能、下拉使能处理，如需要输出pwm波形等，建议自行完成。
- 请求参数，无，使用通用参数；URL中，pin、edge等均为数字，其中edge仅限 0、1，分别代表 低水位电平、高水位电平
- 请求方法，POST
- 请求示例

> curl -X POST "https://espush.cn/openapi/set_gpio_edge/8454703/5/1/?timestamp=1466643023&sign=58415f36b76cede2e85c7fabcec7538d&appid=1234"
- 返回示例

> {"result": "\u0000"}


---



基于以上文档，第一步先定义好api的类，这里直接把完整代码放出来：

``` java
package cn.kevinwu.esp8266_cloud_controller.api;

/**

 * Created by KevinWu on 16-5-30.

 * KevinWu.cn

 */

public class EspushAPI {
    public static final String URL_HOST = "https://espush.cn/openapi/";
    public static final String URL_GET_APPS = URL_HOST + "apps/";
    public static final String URL_GET_DEVICE_LIST = URL_HOST + "openapi/devices/list/";
    public static final String URL_GET_IO_STATUS = URL_HOST + "gpio_status/";
    public static final String URL_REFRESH_DEVICE_ALIVE = URL_HOST + "manual_refresh/";
    public static final String URL_PUSH_MESSAGE = URL_HOST + "dev/push/message/";
    public static final String URL_SET_IO_EDGE = URL_HOST + "set_gpio_edge/";
}

```

这上面用到的只有`URL_HOST` 、 `URL_GET_IO_STATUS` 、 和`URL_SET_IO_EDGE`三个，其他的，额，其实当时想着以后如果深入研究就会用到的，嗯，深入研究。。。

下面看一下网络访问的方法：

``` java

private void getStatus(){
        if(SPUtil.get(MyApplication.AppContext, SPStaticKey.PLUS_SP_FILE,SPStaticKey.CHIPID,"null").equals("null")
                ||SPUtil.get(MyApplication.AppContext, SPStaticKey.PLUS_SP_FILE,SPStaticKey.APPID,"null").equals("null")){
            setTogglebtnAndText();
        }else{
            Setting.chipId=(String)SPUtil.get(MyApplication.AppContext, SPStaticKey.PLUS_SP_FILE,SPStaticKey.CHIPID,"null");
            Setting.appid=(String)SPUtil.get(MyApplication.AppContext, SPStaticKey.PLUS_SP_FILE,SPStaticKey.APPID,"null");
        }
        String url= EspushAPI.URL_GET_IO_STATUS+ Setting.chipId+"/?sign="
                + SignUtil.getSign()+"&timestamp="+ TimeUtil.getTimestamp()+"&appid="+Setting.appid;
        MyNet.get(url)
                .enqueue(new JsonCallback<GPIOStatusRTBean>() {
                    @Override
                    public void onSuccess(final GPIOStatusRTBean entity, final int responseCode, Headers headers) {
                        Log.d(TAG,"网络访问成功");
                        Log.d(TAG,"状态码为"+responseCode);
                        handle.post(new Runnable() {
                            @Override
                            public void run() {
                                if(entity.getResult()!=null&&responseCode==200){
                                    try {
                                        String result=URLEncoder.encode (entity.getResult(), "UTF-8" );
                                        Log.d(TAG,result);
                                        //其他都低电平，io14高电平状态时，返回%00%00%00%00%00%00%00%00%00%00%01%00
                                        //因为分割后io14在下标11的位置，所以先直接写死
                                        io14_Status=Integer.parseInt(result.split("%")[11]);
                                        Log.d(TAG,"--"+io14_Status);
                                        setTogglebtnAndText();
                                    } catch (UnsupportedEncodingException e) {
                                        e.printStackTrace();
                                        io14_Status=-1;
                                        setTogglebtnAndText();
                                    }
                                }else if(responseCode==502){
                                    io14_Status=2;
                                    setTogglebtnAndText();
                                }
                            }
                        });
                    }

                    @Override
                    public void onFailure(String error) {
                        handle.post(new Runnable() {
                            @Override
                            public void run() {
                                io14_Status=2;
                                setTogglebtnAndText();
                            }
                        });
                    }
                });
    }
    private void setStatus(int status){
        String url= EspushAPI.URL_SET_IO_EDGE+ Setting.chipId+"/14/"+status+"/?sign="
                + SignUtil.getSign()+"&timestamp="+ TimeUtil.getTimestamp()+"&appid="+Setting.appid;
        MyNet.get(url)
                .enqueue(new JsonCallback<GPIOStatusRTBean>() {
                    @Override
                    public void onSuccess(final GPIOStatusRTBean entity, int responseCode, Headers headers) {
                        handle.post(new Runnable() {
                            @Override
                            public void run() {
                                getStatus();
                            }
                        });
                    }

                    @Override
                    public void onFailure(String error) {
                        handle.post(new Runnable() {
                            @Override
                            public void run() {
                                io14_Status=2;
                                setTogglebtnAndText();
                            }
                        });
                    }
                });
    }

```



同第一种方案，网络访问之类的操作，用了封装后的OKHTTP，这里当时是省时间直接写上去了，也没有用EventBus之类的，也没考虑mvp的架构问题，不过如果抽离出来也花不了多少时间。。

对于芯片ID和APPID，当然不能暴露出来啦，所以用了sharedpreferences文件保存在手机本地，第一次使用的时候要求设置，因为没有做二维码扫描，而appkey可谓长的可怕，所以就直接写出来了，如果有需要参考这个项目自己写的童鞋记得改改`config`包下面的`Setting`类。

因为比较痴迷UI设计，所以简简单单的一个按钮的UI我也是思考了许久的，最终，也有别于第一种方案那种丑到炸的UI，而且引用了coolswitch实现了比较舒服的动画效果。

预览图如下：

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/78446360.jpg?imageMogr2/thumbnail/400x400)

![](http://7xvo7h.com1.z0.glb.clouddn.com/16-6-24/50208405.jpg?imageMogr2/thumbnail/400x400)

项目地址： https://github.com/KevinWu1993/ESP8266-Cloud-Controller




