---
title: Gson学习笔记
date: 2016-01-30 
author:  KevinWu
categories: Android
tags: 
	- Java 
	- Android
	- Gson
	- Json
	- 解析
---
## Gson是什么
要了解Gson是什么，恐怕得先说一下Json这个东西
（Json的背景知识就直接引用网络上的了http://www.open-open.com/lib/view/open1407376535942.html）。
### 什么是Json
JSON即JavaScript Object Natation, 它是一种轻量级的数据交换格式, 与XML一样, 是广泛被采用的客户端和服务端交互的解决方案.
<!--more-->
### Json对象
JSON中对象(Object)以"{"开始, 以"}"结束. 对象中的每一个item都是一个key-value对, 表现为"key:value"的形式, key-value对之间使用逗号分隔. 如:{"name":"coolxing", "age"=24, "male":true, "address":{"street":"huiLongGuan", "city":"beijing", "country":"china"}}. JSON对象的key只能是string类型的, 而value可以是string, number, false, true, null, Object对象甚至是array数组, 也就是说可以存在嵌套的情况.
### Json数组
JSON数组(array)以"["开始, 以"]"结束, 数组中的每一个元素可以是string, number, false, true, null, Object对象甚至是array数组, 数组间的元素使用逗号分隔. 如["coolxing", 24, {"street":"huiLongGuan", "city":"beijing", "country":"china"}].

好了Json的基本背景介绍完了，先看看一个天气Api返回的Json对象吧（截取部分）。
完整的json返回数据示例地址：https://github.com/KevinWu1993/YTWeatherPro/blob/dev/doc/%E5%AE%8C%E6%95%B4json%E8%BF%94%E5%9B%9E%E6%95%B0%E6%8D%AE.md

```
{
    "reason": "查询成功",
    "result": {
        "data": {
            "realtime": {
                "city_code": "101210701",
                "city_name": "温州",     /*城市*/
                "date": "2014-10-15",  /*日期*/
                "time": "09:00:00",     /*更新时间*/
                "week": 3,
                "moon": "九月廿二",
                "dataUptime": 1413337811,
                "weather": {    /*当前实况天气*/
                    "temperature": "19",     /*温度*/
                    "humidity": "54",     /*湿度*/
                    "info": "雾",
                    "img": "18"
                },
                "wind": {
                    "direct": "北风",
                    "power": "1级",
                    "offset": null,
                    "windspeed": null
                }
            },
            "life": {     /*生活指数*/
                "date": "2014-10-15",
                "info": {
                    "chuanyi": [     /*穿衣指数*/
                        "较舒适",
                        "建议着薄外套或牛仔衫裤等服装。年老体弱者宜着夹克衫、薄毛衣等。昼夜温差较大，注意适当增减衣服。"
                    ],
                    "ganmao": [    /*感冒指数*/
                        "较易发",
                        "昼夜温差较大，较易发生感冒，请适当增减衣服。体质较弱的朋友请注意防护。"
                    ],
                    "kongtiao": [   /*空调指数*/
                        "较少开启",
                        "您将感到很舒适，一般不需要开启空调。"
                    ],
                    "wuran": [     /*污染指数*/
                        "良",
                        "气象条件有利于空气污染物稀释、扩散和清除，可在室外正常活动。"
                    ],
                    "xiche": [     /*洗车指数*/
                        "较适宜",
                        "较适宜洗车，未来一天无雨，风力较小，擦洗一新的汽车至少能保持一天。"
                    ],
                    "yundong": [     /*运动指数*/
                        "较适宜",
                        "天气较好，但风力较大，推荐您进行室内运动，若在户外运动请注意防风。"
                    ],
                    "ziwaixian": [   /*紫外线*/
                        "中等",
                        "属中等强度紫外线辐射天气，外出时建议涂擦SPF高于15、PA+的防晒护肤品，戴帽子、太阳镜。"
                    ]
                }
            },
         ............
```
上面是天气Api返回的json对象，之前开发中是用org.json工具进行解析的，可能我对那个工具也没有参悟得很透彻，我感觉我用的是最笨的方法，下面给出一小段之前写的解析代码：
```java
//取得result根
 JSONObject jsonresult = new JSONObject(value);
 String resultSTR = jsonresult.getString("result");
 
 //取得DATA根
 JSONObject jsonDATA = new JSONObject(resultSTR);
 String dataSTR = jsonDATA.getString("data");
 
//取得realtime根
 JSONObject jsonRT = new JSONObject(dataSTR);
 String rtSTR = jsonRT.getString("realtime");
JSONObject jsonRealTime = new JSONObject(rtSTR);//这个是realtime的json对象
String city = jsonRealTime.getString("city_name"); //取得城市名称
 String updatetime = jsonRealTime.getString("date") + " " + jsonRealTime.getString("time");//获得更新时间，即为返回的数据的日期加更新具体时间
```
好了！废话说了这么多，该谈谈Gson了，回到那个问题，Gson是什么？
其实也就是一个工具。

---

## 使用Gson前准备工作
要使用它，我们要先下载它的依赖包，可以直接去github开源项目地址下载：https://github.com/google/gson
我也搬运过来了，直接去下载也可以：http://download.csdn.net/detail/kevinwu93/9422464

下载后，我们需要在工程项目中依赖它这个jar包，以Android Studio为例，可以放到module中的libs目录下，如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20160130141817334)
接下来还需要在gradle文件中添加依赖语句
```
compile files('libs/gson-2.5.jar')
```

这样，我们就导入这个工具了，接下来就可以使用了。

---

## 正式使用Gson
这里主要讨论使用Gson的fromJson()方法解析Json字符串，获取相应的数据，至于toJson()方法，以后应该还会写篇文章。

先来看看fromJson()这个方法：
fromJson()这个方法的用法如下：
```java
new Gson().fromJson(Json_string,class)
```

参数需要json字符串和一个实体，所以下面先来针对以上天气的返回数据创建一个实体：
```java
public class WeatherEntity {
    public String reason;
    public ResultEntity result;
    public int error_code;//错误码
}
```

在这个天气实体中我分成了三部分，第一部分为一个字符串，对应返回的json数据的reason字段：
```
"reason": "查询成功"
```

最后的error_code对应json数据中的error_code字段：
```
"error_code": 0
```

下面说一下中中间的public ResultEntity result;这个实体类型
因为返回的result字段的数据比较复杂，所以继续细分出来，先看看ResultEntityEntity这个类的代码：
```java
public class ResultEntity {
    public DataEntity data;
}
```

因为result中又包含了一个data字段，所有我再分了一层，再来看看DataEntity部分的代码
``` java
public class DataEntity {
    public RealtimeEntity realtime;
    public LifeEntity life;
    public ArrayList<AdayWeatherEntity> weather=new ArrayList<AdayWeatherEntity>();
    public Pm25Entity pm25;
    public int date;
    public int isForeign;
}
```

分析这段代码前，先看一下返回的json的数据的结构（建议参照完整json数据返回内容）：
```
          "realtime": {
                "city_code": "101210701",
                "city_name": "温州",     /*城市*/
                "date": "2014-10-15",  /*日期*/
                "time": "09:00:00",     /*更新时间*/
                 ............................省略
            },
            "life": {     /*生活指数*/
                "date": "2014-10-15",
                "info": {
                    "chuanyi": [     /*穿衣指数*/
                        "较舒适",
                        "建议着薄外套或牛仔衫裤等服装。年老体弱者宜着夹克衫、薄毛衣等。昼夜温差较大，注意适当增减衣服。"
                    ],
                   ............................省略
            },
            "weather": [   /*未来几天天气预报*/
                {
                    "date": "2014-10-15",
                    "info": {
                        "day": [     /*白天天气*/
               ............................省略
            ],
            "pm25": {    /*PM2.5*/
                "key": "Wenzhou",
                "show_desc": 0,
                "pm25": {
                    "curPm": "97",
                    "pm25": "72",
                    "pm10": "97",
                    "level": 2,
                    "quality": "良",
                    "des": "可以接受的，除极少数对某种污染物特别敏感的人以外，对公众健康没有危害。"
                },
                "dateTime": "2014年10月15日09时",
                "cityName": "温州"
            },
            "date": null,
            "isForeign": 0
```

可以看到这里有几大字段，分别为：
- realtime
- life
- weather
- pm25
- date
- isForeign

对于date和isForeign这两个字段来说，单一数据，可以直接用对应的数据类型做实体，解析很简单。
对应realtime等类型的数据，结构相对来说还是比较复杂的，这里再单独出来进行解析，就用realtime做例子，其它的有需要可以参考github上项目的完整代码，这里不详细说明。
先来看看RealtimeEntity这个类：
``` java
public class RealtimeEntity {
    public String city_code;//城市代码
    public String city_name;//城市名称
    public String date;//日期
    public String time;//更新时间
    public int week;//星期
    public String moon;//农历、
    public long dataUptime;//更新时间戳
    public RealTimeWeatherEntity weather;//实时天气实体
    public RealtimeWindEntity wind;//风速风力信息实体
}
```

对于单一数据没嵌套的直接解析，对于还有嵌套的，继续细分，接下来看看RealTimeWeatherEntity和RealtimeWindEntity这两个类：
```java
public class RealTimeWeatherEntity {
    public String tempertrue;//实时温度
    public String humidity;//湿度
    public String info;//天气信息
    public String img;//天气对应图片
}
```

```java
public class RealtimeWindEntity {
    public String direct;//风向
    public String power;//风力
    public String offset;//风向偏移量
    public String windspeed;//风速
}
```
这两个类都只是对数据进行定义，相当于把json中的数据一一对应地描述出来，这样进行了多层细分后，realtime这个字段返回的数据就每个都有了对应的变量来描述，其他的也同样道理，在对所有的数据进行实体定义后，就可以调用Gson方法进行解析了，解析部分的代码如下（我把测试的json数据放在个txt文件里面了）：
```java
/**
    *测试gson实体
    *@author KevinWu
    *create at 2016/1/29 22:15
    */
    @Test
    public void testEntity(){
        String jsonStr=importStr();//导入要测试的数据
        System.out.println(jsonStr);
        Gson gson=new Gson();
        WeatherEntity w=gson.fromJson(jsonStr,WeatherEntity.class);
        System.out.println(w.result.data.pm25.pm25.pm10);
    }
 
    private String importStr(){
        String str="";
        File myFile=new File("./json_test.txt");
        try {
            FileReader myFileReader=new FileReader(myFile);
            BufferedReader myBufferedReader=new BufferedReader(myFileReader);
            String temp;
            while((temp=myBufferedReader.readLine())!=null){
                str=str+temp;
            }
            myBufferedReader.close();
            myFileReader.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return str;
    }
```

就这样。

Github项目地址：https://github.com/KevinWu1993/YTWeatherPro



