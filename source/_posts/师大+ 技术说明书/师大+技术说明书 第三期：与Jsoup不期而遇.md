---
title: 师大+ 技术说明书 第三期：与Jsoup不期而遇
date: 2016-05-07 
author:  KevinWu
categories: 师大+ 技术说明书
tags: 
	- Java 
	- Android
	- Jsoup
	- 工具
---
说到HTML解析，最先想到的是能不能拓展下DOM来解析一个特定HTML文档，后来无意上网中发现了Jsoup这个工具，真是有种相见恨晚的感觉。

这篇文章作为初期引导知识，将不详细介绍如何用Jsoup去解析一个HTML文档，具体操作下一期将会仔细介绍。
<!--more-->

## 工欲善其事，必先利其器

要了解jsoup怎么解析HTML的，有必要先了解一下要解析的对象的结构，才能更高效去完成它的解析代码的编写，而知识也是一连串的东西，所以先来了解一下HTML的编程接口DOM。

### DOM
根据W3C DOM规范，DOM是HTML与XML的应用编程接口（API）。

### HTML DOM
HTML DOM是符合W3C标准的HTML的标准对象模型和标准编程接口，它定义了所有HTML元素的对象和属性，以及访问它们的方法（接口），说白了就是如何获取、修改、添加或删除HTML元素的标准。

引用一下w3cschool上面的HTML DOM树的示意图：

![HTML DOM](http://www.w3school.com.cn/i/ct_htmltree.gif)

整个树形图下来，可以看出来包含关系，简单点说就是，在使用Jsoup解析一个body元素时，那么会连带把body里面的元素一并解析出来。

### DOM节点
DOM节点这里直接用一个例子介绍下：
``` html
<html>
  <head>
    <title>DOM 节点</title>
  </head>
  <body>
    <h1>DOM 节点网页标题</h1>
    <p>Hello world!</p>
  </body>
</html>
```
在上面的HTML文档中，节点分别有

- `<html>`
- `<head>`
- `<title>`
- `<body>`
- `<h1>`
- `<p>`
 
其中`<html>`节点没有父节点，它就是根节点，`<head>`和`<body>`节点的父节点是`<html>`节点。

像`<h1>`和`<p>`这两个节点就称为同胞节点。

到这里，基本了解了DOM是什么，如果你想了解更多知识可以参考w3cschool的教程（虽然是Javascript的教程，不过还是很有参考意义的）：
http://www.w3school.com.cn/htmldom/index.asp

## 磨刀霍霍向猪羊
上面了解了铺垫知识，下面可以对Jsoup开刀了。

Jsoup不仅可以解析本地文档，还可以直接给一个url让它去解析，但这里主要说的是解析，所以以本地文档为准。

下面先简略看看解析用到的几个类。

### Node
这个类是一个抽象类，定义为一个抽象的节点模型Elements和Documents都是它的子类，这两个类后面会介绍。

Node类有几个成员变量：

- Node parentNode;
这个节点对象指向的是父类节点，如果你了解二叉树的结构，对于这个应该比较容易理解。

- List<Node> childNodes;
用于记录子节点的集合。

- Attributes attributes;
记录属性值的变量。

- String baseUri;
记录该节点的基础uri。

- int siblingIndex;
记录在兄弟节点列表中该节点的下标，下标重0开始。

- private static final List<Node> EMPTY_NODES = Collections.emptyList();
利用enptyList()方法给产生一个空的子节点（因为不确定是一个子节点还是后面跟着一系列子节点，所以是一个List）。

了解了这几个成员变量后，来看一下它的构造方法，Node类有三个构造方法：
``` java
/**
     创建一个新的节点
     @param baseUri base URI
     @param attributes attributes (不排除是一个null值)
 */
    protected Node(String baseUri, Attributes attributes) {
        Validate.notNull(baseUri);
        Validate.notNull(attributes);
        
        childNodes = EMPTY_NODES;
        this.baseUri = baseUri.trim();
        this.attributes = attributes;
    }

/**
     创建一个新的节点，实际上也是调用前面一个方法来创建的
 */
    protected Node(String baseUri) {
        this(baseUri, new Attributes());
    }

    /**
     * 这个是默认的构造方法，不会初始化base uri，子节点和属性，所以使用这个构造方法时要格外小心
     */
    protected Node() {
        childNodes = EMPTY_NODES;
        attributes = null;
    }

```
粗略看一下Node类部分方法，这个不了解关系也不大，不想了解的可以略过这里，当然如果想深究原理的话可以仔细阅读下源码。这里我就直接把说明放在代码的注释里面了，代码给个方法头说明一下：
``` java
//取得当前节点的名字
public abstract String nodeName();
//通过key取得对应的属性
public String attr(String attributeKey);
//取得所有元素的属性
public Attributes attributes();
//设置节点的属性，如果对应的属性已存在，则会被替代
public Node attr(String attributeKey, String attributeValue);
//检测该元素是否含有该属性，有则返回true，没有则返回false
public boolean hasAttr(String attributeKey);
//移除一个属性
public Node removeAttr(String attributeKey);
//从关联的URL属性里取得一个绝对地址
public String absUrl(String attributeKey);
//取得Document对象与当前Node结合
public Document ownerDocument();
//对节点填充支持的html对象
public Node wrap(String html);
//把该节点从DOM树里面移除，同时把子节点上移
public Node unwrap();
//检测该节点的同胞节点，并以list形式返回
public List<Node> siblingNodes();
//深度优先遍历该节点及子节点
public Node traverse(NodeVisitor nodeVisitor);
```
### Element
Element继承自Node，解析的方法都在这个类里面，废话少说，先看看构造方法：
``` java
 /**
     * Create a new, standalone Element. (Standalone in that is has no parent.)
     * 
     * @param tag tag of this element
     * @param baseUri the base URI
     * @param attributes initial attributes
     * @see #appendChild(Node)
     * @see #appendElement(String)
     */
    public Element(Tag tag, String baseUri, Attributes attributes) {
        super(baseUri, attributes);
        
        Validate.notNull(tag);    
        this.tag = tag;
    }
    
    /**
     * Create a new Element from a tag and a base URI.
     * 
     * @param tag element tag
     * @param baseUri the base URI of this element. It is acceptable for the base URI to be an empty
     *            string, but not null.
     * @see Tag#valueOf(String)
     */
    public Element(Tag tag, String baseUri) {
        this(tag, baseUri, new Attributes());
    }
```

先来了解一下Element类中常用的几个方法：

- getAllElements() 
返回所有的元素，包括元素的子元素，以及子元素的子元素

- getElementById(String id) 
根据元素的ID查找元素

- getElementsByAttribute(String key) 
根据属性名查找元素
 
- getElementsByAttributeValue(String key, String value)
根据属性键值对查代元素

- getElementsByClass(String className) 
根据css的class查找元素

- getElementsByTag(String tagName) 
根据标签名称查找元素

然后来看一个Element类在解析中比较重要的方法：

#### select

原注释讲的太详细，我就给直接放出来了，这个方法通过css样式把对应的Element 选择出来，其内部是通过调用Selector类的select方法来实现的，返回的是Elements对象，有兴趣的童鞋可以去看看Elements的源码，这里不展开，Elements实际上就是一个拓展的ArrayList，泛型就是Element：
``` java
/**
     * Find elements that match the {@link Selector} CSS query, with this element as the starting context. Matched elements
     * may include this element, or any of its children.
     * <p>
     * This method is generally more powerful to use than the DOM-type {@code getElementBy*} methods, because
     * multiple filters can be combined, e.g.:
     * </p>
     * <ul>
     * <li>{@code el.select("a[href]")} - finds links ({@code a} tags with {@code href} attributes)
     * <li>{@code el.select("a[href*=example.com]")} - finds links pointing to example.com (loosely)
     * </ul>
     * <p>
     * See the query syntax documentation in {@link org.jsoup.select.Selector}.
     * </p>
     * 
     * @param cssQuery a {@link Selector} CSS-like query
     * @return elements that match the query (empty if none match)
     * @see org.jsoup.select.Selector
     * @throws Selector.SelectorParseException (unchecked) on an invalid CSS query.
     */
    public Elements select(String cssQuery) {
        return Selector.select(cssQuery, this);
    }
```
了解了这个select，我们就可以通过传入特定的css来解析我们要的html网页了，一般调用这个用的是Document实例，Document是继承自Element的子类，用来表示一个HTML文档，这里不深究，通过这个方法返回了Elements实例后，我们可以运用ArrayList遍历的方式，逐个元素遍历，就可以取得每一个符合类型的元素了。

上面讲的都是如何选择的过程，但是注意到我们用的是Document实例去调用select方法，那么这个表示HTML文档的Document实例是怎么来的呢？

这个就简单了，它是通过Jsoup类的parse方法返回的，Jsoup的parse方法内部又是通过调用Parser类中的parse方法来解析的，而Parser类中parse方法又是调用TreeBuilder类中的parse方法解析，追踪下去会发现实际上是在TreeBuilder类中的runParser()方法中完成解析工作的，这里就不详细介绍这些过程了，如果想了解这些详细过程，建议先找DOM解析xml文档的相关文章看一下，原理是类似的。

这一期基本就讲完了，主要先了解一下Jsoup，下一期将针对一个网页返回数据来实战，进行解析。
