---
title: 师大+技术说明书 第四期：Jsoup解析实战
date: 2016-05-08 17:19
author:  KevinWu
categories: 师大+ 技术说明书
tags: 
	- Java 
	- Android
	- Jsoup
	- 工具
	- 实战
---
在上一期的技术说明书中，介绍了一些使用Jsoup之前应该了解的知识，这一期开始用Jsoup对一个实际的HTML页面进行解析。

这一期主要是解析，所以不涉及如何建立链接并获取网络数据的讲解，关于那部分知识，可以参考OKhttp使用教程、HttpClient使用教程之类的文章，当然【师大+技术说明书】后续也会对这部分知识进行梳理。
<!--more-->
## 分析

解析一个HTML之前，我们要做的就是先对这个HTML文档进行分析，找出需要解析部分的特征。

这里选择[江西师范大学新闻网](http://news.jxnu.edu.cn/s/271/t/910/p/12/list.htm)的一个页面，把它里面的列表页解析出来，该页面样式如下：

![](http://i3.buimg.com/d68dfef3ffe20f18.png)

我们这里用火狐浏览器分析数据包的数据，在火狐浏览器中按下F12，即可打开调试页面（这个做web开发的童鞋应该比我们熟悉多了）

![](http://i3.buimg.com/4a7cdd58e821e2e2.png)

一般html的文本内容都在第一个数据包里面，我们选择第一个数据包，选择响应功能模块，把里面的响应文本内容复制下来，再在本地创建一个文本文件放进去，供后续测试使用。

因为整个页面的HTML响应文本内容太长，这里就不直接放出来整个响应文本，只节选一部分用于分析的文本内容：

``` html
  <div class="listright">
     <div class="top">
	   <h3 class="listname"> 师大要闻</h3>
	   <div class="listmap">当前位置：<a href='/s/271/t/910/main.htm' title='返回首页'>首页</a>&nbsp;|&nbsp;<a href='/s/271/t/910/p/12/list.htm' title='师大要闻'>师大要闻</a></div>
	   <div class="cr"></div>
	 </div>
	 <div class="bot">
	   <div class="listbox">
	      <link href='/css/divwin.css' type='text/css' rel='stylesheet'><script language='javascript' src='/include/classbase.js'></script><script language='javascript' src='/include/windowopener.js'></script><table width="100%" border="0" cellspacing="0" cellpadding="0"><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/29/b1/info76209.htm' target='_blank' style=''><font color=''>省委书记强卫五四青年节到我校调研党建与思想政治工作并召开大学生党员“两学一做”学习教育座谈会</font></a></td><td align='center' width='14%' nowrap>2016-05-04</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/2a/e2/info76514.htm' target='_blank' style=''><font color=''>中华美学精神高端论坛暨中华美学学会理事会在我校召开</font></a></td><td align='center' width='14%' nowrap>2016-05-08</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/2a/42/info76354.htm' target='_blank' style=''><font color=''>学校召开党委扩大会议专题学习强卫书记莅校视察时重要讲话精神</font></a></td><td align='center' width='14%' nowrap>2016-05-05</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/2a/40/info76352.htm' target='_blank' style=''><font color=''>全省高校毕业生就业工作评估专家组来校检查指导</font></a></td><td align='center' width='14%' nowrap>2016-05-05</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/29/9f/info76191.htm' target='_blank' style=''><font color=''>马克思青铜塑像在我校落成</font></a></td><td align='center' width='14%' nowrap>2016-05-04</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/29/0d/info76045.htm' target='_blank' style=''><font color=''>我校在读硕士研究生周国兵喜获美国University of Oklahoma博士全额奖学金</font></a></td><td align='center' width='14%' nowrap>2016-05-02</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/29/0c/info76044.htm' target='_blank' style=''><font color=''>我校辅导员在第五届全国辅导员职业能力大赛第五赛区复赛中荣获佳绩</font></a></td><td align='center' width='14%' nowrap>2016-04-30</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/28/b0/info75952.htm' target='_blank' style=''><font color=''>学校召开“两学一做”学习教育动员部署会</font></a></td><td align='center' width='14%' nowrap>2016-04-29</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/28/ae/info75950.htm' target='_blank' style=''><font color=''>省委教育工委书记黄小华、省教育厅长叶仁荪来校调研党建与思政工作</font></a></td><td align='center' width='14%' nowrap>2016-04-29</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/28/b2/info75954.htm' target='_blank' style=''><font color=''>第十五届“汉语桥”马达加斯加赛区选拔赛暨“中国大使奖学金”颁发仪式在塔那那利佛大学举行</font></a></td><td align='center' width='14%' nowrap>2016-04-28</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/28/b1/info75953.htm' target='_blank' style=''><font color=''>学校举办“一流本科”大讨论专题报告会</font></a></td><td align='center' width='14%' nowrap>2016-04-28</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/26/f2/info75506.htm' target='_blank' style=''><font color=''>江西理工大学客人来我校国家大学科技园考察</font></a></td><td align='center' width='14%' nowrap>2016-04-27</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/26/ec/info75500.htm' target='_blank' style=''><font color=''>【瑶湖讲坛】张建理：我与认知语言学——我的治学之路</font></a></td><td align='center' width='14%' nowrap>2016-04-27</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/26/e9/info75497.htm' target='_blank' style=''><font color=''>我校2016年校园心理剧大赛圆满落幕</font></a></td><td align='center' width='14%' nowrap>2016-04-27</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/25/06/info75014.htm' target='_blank' style=''><font color=''>韩国国立济州大学客人来校访问</font></a></td><td align='center' width='14%' nowrap>2016-04-26</td></tr></table></td>
            </tr><tr> 
              <td width="2%" height="25"><img src="/page/main910/images/d_26.gif" /></td>
              <td width="98%"><table width=100% border=0 cellspacing=0 cellpadding=0 class='columnStyle'><tr><td><a href='/s/271/t/910/25/0f/info75023.htm' target='_blank' style=''><font color=''>我校承办江西省高校第二届日语教师教学技能培训班</font></a></td><td align='center' width='14%' nowrap>2016-04-25</td></tr></table></td>
            </tr></table><table width=100% cellSpacing=0 cellPadding=0 border=0><tr><td height='10' align="center"><font style="font-size:9pt"><span width="100%">
```
分析上面的html文档内容可以发现，每个列表项都有`class='columnStyle'`这个元素，这个是类标签，而且这个类只有每个新闻标题列表项才存在，别的地方都没有，所以通过这个唯一标识就可以选择出每个新闻列表。

但这个解析可以先放到后面，下面开始对这些数据进行建模。

## 建模

建模之前，先确定下来那些信息是需要的，我们分析上面的数据发现，我们需要以下数据：

- 新闻标题
- 新闻发布时间
- 新闻的链接

基于这个需求，我们开始建立一个CampusNewsModel（实际开发中因为用的是MVP框架，所以模型没那么简单，这里只是节选分析需要的部分）：
``` java
public class CampusNewsModel{
    /**
     * 校内新闻
     */
    private String NewsTitle;
    private String NewsTime;
    private String NewsURL;

  public CampusNewsModel(String newsTitle, String newsTime, String newsURL) {
        NewsTitle = newsTitle;
        NewsTime = newsTime;
        NewsURL = newsURL;
    }

 public String getNewsTitle() {
        return NewsTitle;
    }

    public void setNewsTitle(String newsTitle) {
        NewsTitle = newsTitle;
    }

    public String getNewsTime() {
        return NewsTime;
    }

    public void setNewsTime(String newsTime) {
        NewsTime = newsTime;
    }

    public String getNewsURL() {
        return NewsURL;
    }

    public void setNewsURL(String newsURL) {
        NewsURL = newsURL;
    }

}
```

## 数据解析及模型的填充
建立了这个模型后，下面开始着手解析工作，为了后面的方便维护，可以先规范点代码，所以先建立一个HtmlUtil工具类：
``` java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.List;

import cn.edu.jxnu.awesome_campus.support.utils.common.TextUtil;

/**
 * Created by KevinWu on 2016/2/3.
 */
public class HtmlUtil {
    private Document doc;

    /**
     * 构造方法
     * 以字符串为参数解析静态html页面
     * 字符编码为后编，可选。
     *
     * @author KevinWu
     * create at 2016/2/3 12:59
     */
    public HtmlUtil(String htmlString, String... charsetName) throws UnsupportedEncodingException, NullHtmlStringException {
        if (TextUtil.isNull(htmlString)!=true) {
            if (charsetName.length == 1) {
                String ehtmlString = URLEncoder.encode(htmlString, charsetName[0]);
                this.doc = Jsoup.parse(ehtmlString);
            } else {
                this.doc = Jsoup.parse(htmlString);
            }
        } else {
            this.doc = null;
            throw new NullHtmlStringException("html解析数据空异常");
        }
    }

    /**
     * 返回解析后的字符串
     * 参数（css选择样式）
     *
     * @author KevinWu
     * create at 2016/2/3 13:46
     */
    public List<String> parseString(String cssQuery) {
        List<String> resultList = new ArrayList<>();
        Elements myElement = doc.select(cssQuery);
        for (int i = 0; i < myElement.size() ;i++) {
                resultList.add(myElement.get(i).text().toString());
        }
        return resultList;
    }

    /**
     * 返回解析后的原生html
     * 参数（css选择样式）
     *
     * @author KevinWu
     * create at 2016/2/3 16:09
     */
    public List<String> parseRawString(String cssQuery) {
        List<String> resultList = new ArrayList<>();
        Elements myElement = doc.select(cssQuery);
        for (int i = 0; i < myElement.size() ;i++) {
            resultList.add(myElement.get(i).toString());
        }
        return resultList;
    }
}
```
可以看到，在这个工具类中建立了两个解析的方法，其中，`parseString`方法是提供解析后的字符串，`parseRawString`方法提供解析后的html源字符串。这两个方法都让它返回String内容填充的List对象，到时候解析时直接以分组形式填充model会很方便。

HtmlUtil的构造方法定义了自定义异常，当待解析的字符串为空时则抛出该异常，构造方法中使用了一个自定义的工具类TextUtil，这个工具类只是用于检测字符串是否为空，当前使用中和equals类似，不展开解释。


回到前面的列表项，因为已经确定了`class='columnStyle'`可以作为对每个新闻列表的唯一标识符，所以可以以这个作为参数传递给`parseString`方法，但是要注意的是在解析里面，应该写成：
``` java
private final static String ITEM_CSS = "table[class=columnStyle]";//选择CSS
```

通过ITEM_CSS这个值调用`parseString`方法可以获取的结果如下：

``` txt
省委书记强卫五四青年节到我校调研党建与思想政治工作并召开大学生党员“两学一做”学习教育座谈会 2016-05-04
中华美学精神高端论坛暨中华美学学会理事会在我校召开 2016-05-08
学校召开党委扩大会议专题学习强卫书记莅校视察时重要讲话精神 2016-05-05
。。。。。。后面省略
```

可以看到，通过一次解析后，可以填充到对应model的NewsTitle和NewsTime字段，但是还有个**NewsURL**字段没有数据填充，而新闻的url信息并不在html解析后的文本中显示出来的，而是作为a标签里面的href值，所以，我们只有通过`parseRawString`方法，对每一条数据进行解析，再综合运用String类的`split`方法对数据进行分割（当然运用正则表达式来匹配提取也可以）。

所以假如我们调用`parseRawString`方法获取到了一个临时原生html列表tempRawStr，那么：
``` java
String tempRawStr = tempRawStrList.get(i).toString();
String newsUrl;
String cutUrlLeft[] = tempRawStr.split("href=\"");
     if (cutUrlLeft.length > 1) {
               String cutUrlRight[] = cutUrlLeft[1].split("\" target=\"_blank\"");
      if (cutUrlRight.length > 1) 
               newsUrl = cutUrlRight[0];
     }
```
通过以上代码即可将一条新闻的url取出来。

下面我们来建立一个CampusNewsParse校园新闻解析类，先确定下来使用规则：

- 通过HTML文本字符串参数够造一个CampusNewsParse解析对象。
- 通过一个get方法可以获取最终填充好的model列表

基于这个需求，我们写了如下代码：
``` java
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.List;

import cn.edu.jxnu.awesome_campus.model.home.CampusNewsModel;
import cn.edu.jxnu.awesome_campus.support.utils.html.HtmlUtil;
import cn.edu.jxnu.awesome_campus.support.utils.html.NullHtmlStringException;

/**
 * 校内要闻解析
 * 使用：通过传进来html后，执行getEndList()即可获取模型对象集
 * Created by KevinWu on 2016/2/3.
 */
public class CampusNewsParse {
    private final static int GROUPSIZE = 3;//信息分组大小
    private final static String ITEM_CSS = "table[class=columnStyle]";//选择CSS
    private final static String REFERENCE_STR = "2015-12-24";//参考时间字符串
    private String parseStr;//待解析字符串

    public List<CampusNewsModel> getEndList() {
        return endList;
    }

    private List<String> resultList;//结果列表
    private List<CampusNewsModel> endList;//最终返回列表

    /**
     * 构造时即执行解析，解析后自动填充结果
     *
     * @author KevinWu
     * create at 2016/2/6 19:03
     */
    public CampusNewsParse(String parseStr) {
        super();
        this.parseStr = parseStr;
        resultList = new ArrayList<>();
        endList = new ArrayList<>();
        parseData();
    }

    /**
     * 解析数据
     *
     * @author KevinWu
     * create at 2016/2/6 19:06
     */
    private void parseData() {
        try {
            HtmlUtil hu = new HtmlUtil(parseStr);
            List tempStrList = hu.parseString(ITEM_CSS);
            List tempRawStrList = hu.parseRawString(ITEM_CSS);
            for (int i = 0; i < tempStrList.size(); i++) {
                String tempStr = tempStrList.get(i).toString();
                String newsTitle = tempStr.substring(0, tempStr.length() - REFERENCE_STR.length()).trim();
                String newsTime = tempStr.substring(tempStr.length() - REFERENCE_STR.length(), tempStr.length());
                String tempRawStr = tempRawStrList.get(i).toString();
                String newsUrl;
                String cutUrlLeft[] = tempRawStr.split("href=\"");
                if (cutUrlLeft.length > 1) {
                    String cutUrlRight[] = cutUrlLeft[1].split("\" target=\"_blank\"");
                    if (cutUrlRight.length > 1) {
                        newsUrl = cutUrlRight[0];
                    } else {
                        continue;
                    }
                } else {
                    continue;
                }
                resultList.add(newsTitle);
                resultList.add(newsTime);
                resultList.add(newsUrl);
            }
            fillEndList();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (NullHtmlStringException e) {
            e.printStackTrace();
        }

    }

    /**
     * 填充最终模型
     *
     * @author KevinWu
     * create at 2016/2/6 21:25
     */
    private void fillEndList() {
        for (int i = 0; i <= resultList.size() - GROUPSIZE; i = i + GROUPSIZE) {
            endList.add(new CampusNewsModel(
                    resultList.get(i).toString(),
                    resultList.get(i + 1).toString(),
                    resultList.get(i + 2).toString()
            ));
        }
    }
}
```


以上便是运用Jsoup进行数据解析实战的说明书，下一期将来介绍如何在没有任何接口的情况下利用模拟登录接入一个网站的账户。
