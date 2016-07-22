---
title: 新浪免费天气Api简单使用说明
date: 2015-11-28
author:  KevinWu
categories: Android
tags: 
	- Java 
	- Android
	- 天气
	- API
	- 解析
---
最近在做数据库的大作业，有一个天气功能要做，之前做过一个天气app，用的是百度车联网的Api，得到的信息有点少，但是对于那个app的定位来说是够用了，想玩玩那个软件的可以去以下链接：http://www.coolapk.com/apk/fsyt.ytweather
<!--more-->
下面开始说说新浪这个Api，示例地址为：
http://php.weather.sina.com.cn/xml.php?city=%C4%CF%B2%FD&password=DJOYnieT8234jlsK&day=0

其中的password是固定的，不要更改。
city为你要获取的城市，这里是获取南昌的天气，南昌对应的gb2312的编码为%C4%CF%B2%FD，在Java中转化编码也很简单，示例为：

```
String strCity = URLEncoder.encode("南昌", "GB2312");
```
day为要获取的日期参数，0表示当天，1表示明天，以此类推。

下面分析一下返回的数据：
返回的数据用浏览器访问可以看到如下：

南昌 多云 多云 duoyun duoyun 无持续风向 无持续风向 ≤3 ≤3 15 9 0 16 16 1 6 3 4 暂无 暂无 暂无 套装、夹衣、风衣、夹克衫、西服套装、马甲衬衫配长裤 轻度 最弱 较凉 暂无 暂无 夹衣类 适宜开启(制热) 暂无 暂无 对空气污染物扩散无明显影响 紫外线最弱 老年、幼儿、体弱者外出需要带上薄围巾、薄手套。 适宜开启空调 暂无 2 易发期 天气很凉，季节转换的气候，慎重增加衣服；较易引起感冒； 5 不适宜 虽然晴空万里，但是天气较凉，多数人不适宜户外运动； 2015-11-30 2015-11-30 2015-11-30 2015-11-28 17:10:11

来源： <http://php.weather.sina.com.cn/xml.php?city=%C4%CF%B2%FD&password=DJOYnieT8234jlsK&day=2>

这是把day设为2时获取到的数据，查看网页的源代码可以看到：

```
<!-- saved from url=(0088)http://php.weather.sina.com.cn/xml.php?city=%C4%CF%B2%FD&password=DJOYnieT8234jlsK&day=2 -->
<html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><style type="text/css"></style></head><body><profiles>
<weather>
<city>南昌</city>
<status1>多云</status1>
<status2>多云</status2>
<figure1>duoyun</figure1>
<figure2>duoyun</figure2>
<direction1>无持续风向</direction1>
<direction2>无持续风向</direction2>
<power1>≤3</power1>
<power2>≤3</power2>
<temperature1>15</temperature1>
<temperature2>9</temperature2>
<ssd>0</ssd>
<tgd1>16</tgd1>
<tgd2>16</tgd2>
<zwx>1</zwx>
<ktk>6</ktk>
<pollution>3</pollution>
<xcz></xcz>
<zho></zho>
<diy></diy>
<fas></fas>
<chy>4</chy>
<zho_shuoming>暂无</zho_shuoming>
<diy_shuoming>暂无</diy_shuoming>
<fas_shuoming>暂无</fas_shuoming>
<chy_shuoming>套装、夹衣、风衣、夹克衫、西服套装、马甲衬衫配长裤</chy_shuoming>
<pollution_l>轻度</pollution_l>
<zwx_l>最弱</zwx_l>
<ssd_l>较凉</ssd_l>
<fas_l>暂无</fas_l>
<zho_l>暂无</zho_l>
<chy_l>夹衣类</chy_l>
<ktk_l>适宜开启(制热)</ktk_l>
<xcz_l>暂无</xcz_l>
<diy_l>暂无</diy_l>
<pollution_s>对空气污染物扩散无明显影响</pollution_s>
<zwx_s>紫外线最弱</zwx_s>
<ssd_s>老年、幼儿、体弱者外出需要带上薄围巾、薄手套。</ssd_s>
<ktk_s>适宜开启空调</ktk_s>
<xcz_s>暂无</xcz_s>
<gm>2</gm>
<gm_l>易发期</gm_l>
<gm_s>天气很凉，季节转换的气候，慎重增加衣服；较易引起感冒；</gm_s>
<yd>5</yd>
<yd_l>不适宜</yd_l>
<yd_s>虽然晴空万里，但是天气较凉，多数人不适宜户外运动；</yd_s>
<savedate_weather>2015-11-30</savedate_weather>
<savedate_life>2015-11-30</savedate_life>
<savedate_zhishu>2015-11-30</savedate_zhishu>
<udatetime>2015-11-28 17:10:11</udatetime>
</weather>
</profiles>
</body></html>
```

下面我将建立一个表格来列出这些对应的标签的说明（可能有误，个人分析结果）

标签     | 说明
-------- | ---
city |对应的查询城市
status1|白天天气情况
status2|夜间天气情况
figure1|白天天气情况拼音
figure2|夜间天气情况拼音
direction1|白天风向
direction2|夜晚风向
power1|白天风力
power2|夜间风力
temperature1|白天温度
temperature2|夜间温度
ssd|体感指数
tgd1|白天体感温度
tgd2|夜间体感温度
zwx|紫外线强度
ktk|空调指数
pollution|污染指数
xcz|洗车指数
zho|综合指数？这个我不确定
diy|没猜出来是什么指数，没有数值
fas|同上
chy|穿衣指数
zho_shuoming|zho的说明，然而zho是什么指数我也不确定
diy_shuoming|同上
fas_shuoming|同上
chy_shuoming|穿衣指数说明
pollution_l|污染程度
zwx_l|紫外线指数概述
ssd_l|体感指数概述
fas_l|这个不知道
zho_l|这个也不清楚
chy_l|穿衣指数概述（可理解为穿衣建议）
ktk_l|空调指数概述
xcz_l|洗车指数概述
diy_l|这个不知道
pollution_s|污染指数详细说明
zwx_s|紫外线详细说明
ssd_s|体感详细说明
ktk_s|空调指数详细说明
xcz_s|洗车详细说明
gm|感冒指数
gm_l|感冒指数概述
gm_s|感冒指数详细说明
yd|运动指数
yd_l|运动指数概述
yd_s|运动指数详细说明
savedate_weather|天气数据日期
savedate_life|生活数据日期
savedate_zhishu|指数数据日期
udatetime|更新时间



好了，这么长的一张表，终于列完了，下面提供一下芋头天气这个app所用到的各种天气情况的图标，有需要的可以直接拿去用。
http://download.csdn.net/detail/kevinwu93/9308497

![](http://img.blog.csdn.net/20151128201903284)





