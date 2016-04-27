# Hacker
## Hacker是一款专注于写作的简洁博客主题。在如此讲究复杂排版的趋势下，选择回归本源，专注于写作这件事。
一开始是[moyo](http://liuxinyu.me/)为Wordpress所创作的一个主题，由DaraW移植到Hexo。

## Demo
[DaraW](http://blog.daraw.cn/)

## 安装

在`themes`文件夹中创建文件夹`hacker`，将主题文件都复制粘贴至`hacker`文件夹，然后在hexo全局配置文件`_config.yml`中应用主题：
```yaml
theme: hacker
```
这样就安装好了，开始享受吧~

## 配置
在主题配置文件`_config.yml`中：

```yaml
# duoshuo comment
duoshuo: true
duoshuo_name:

# google analytics
googleTrackId:
```


`duoshuo`代表是否开启多说评论，设为`false`则为取消使用多说；  
`duoshuo_name`为你的多说用户ID，请不要使用别人的ID。

`googleTrackId`为谷歌分析的个人ID，留空则为不使用谷歌分析。

## 待办
* 添加夜间模式
* 修改多说评论框样式
* 去除无用css代码

暂时不会考虑添加分享等插件。

## 协议
GPL(General Public License)
