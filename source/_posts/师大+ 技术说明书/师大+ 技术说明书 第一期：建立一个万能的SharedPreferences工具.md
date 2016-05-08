---
title: 师大+ 技术说明书 第一期：建立一个万能的SharedPreferences工具
date: 2016-05-05 
author:  KevinWu
categories: 师大+ 技术说明书
tags: 
	- Java 
	- Android
	- SharedPreferences
	- 工具
	- 框架
---
App开发前期都会先去搭建框架，除了确定下来整体使用的架构后，还需要给框架里面加入一些工具类，来使这些功能使用的时候能尽量简化代码，使代码看起来更简洁，这次先来看看如果建立一个万能的SharePreferences工具——SPUtil。
<!--more-->

建立这个工具前，先来看看SharedPreferences这个东西：

### SharedPreferences
SharedPreferences是Android中一种轻量化数据存储方案，优于文本文件，操作又可以比数据库简单，一般用来存放应用程序的设置缓存。

SharedPreferences官方描述是：Interface for accessing and modifying preference data，意思是返回一个偏好数据修改的接口，可以理解为偏好设置的数据接口，获取它的方式是通过Context类的getSharedPreferences方法，这是一个抽象方法，有两个参数，分别是：

- String name：目标SharedPreferences的文件名，如果该preferences目标文件不存在的话，将会在你取得一个Editor接口并调用commit()方法时创建该文件。
- int mode：操作模式，有三种，分别为MODE_PRIVATE，MODE_WORLD_READABLE，MODE_WORLD_WRITEABLE，默认为MODE_PRIVATE。

了解了这个，下面来先定义一个初始化SPUtil的方法，这里选择在构造方法中初始化Context，然后再定义一个内部使用的初始化SharedPreferences接口的方法：

``` java
private Context context;

public SPUtil(Context context) {
        super();
        this.context = context;
    }

private SharedPreferences initSP(String spName) {
        SharedPreferences sp = context.getSharedPreferences(spName, 0);
        return sp;
    }
```


通过getSharedPreferences方法获取SharedPreferences接口对象后，即可通过一系列的get方法来读取SharedPreferences文件的数据，所以可以先把get系列方法完善了，这里应该注意一下这些方法的默认返回值，数值型的默认返回0，布尔型默认返回false，字符串默认返回null：
``` java

    /**
     * 获取int型Sharedpreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:27
     */
    public int getIntSP(String spName, String spKey) {
        SharedPreferences sp = initSP(spName);
        int spGet = sp.getInt(spKey, 0);
        return spGet;
    }

    /**
     * 获取long型Sharedpreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:27
     */
    public long getLongSP(String spName, String spKey) {
        SharedPreferences sp = initSP(spName);
        long spGet = sp.getLong(spKey, 0);
        return spGet;
    }

    /**
     * 获取float型Sharedpreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:27
     */
    public float getFloatSP(String spName, String spKey) {
        SharedPreferences sp = initSP(spName);
        float spGet = sp.getFloat(spKey, 0);
        return spGet;
    }

    /**
     * 获取boolean型Sharedpreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:27
     */
    public boolean getBooleanSP(String spName, String spKey) {
        SharedPreferences sp = initSP(spName);
        boolean spGet = sp.getBoolean(spKey, false);
        return spGet;
    }

    /**
     * 获取String型SharedPreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:55
     */
    public String getStringSP(String spName, String spKey) {
        SharedPreferences sp = initSP(spName);
        String spGet = sp.getString(spKey, null);
        return spGet;
    }
```

然后，添加完这一系列方法后，来添加一系列的put方法，这些方法都要用到Editor，下面都反复写了这个，如果可以抽出来，作为该工具的内部方法，会更好：

``` java
/**
     * 填充int型SharedPreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:59
     */
    public void putIntSP(String spName, String spKey, int value) {
        SharedPreferences sp = initSP(spName);
        SharedPreferences.Editor ed = sp.edit();
        ed.putInt(spKey, value);
        ed.commit();
    }

    /**
     * 填充float型SharedPreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:59
     */
    public void putFloatSP(String spName, String spKey, float value) {
        SharedPreferences sp = initSP(spName);
        SharedPreferences.Editor ed = sp.edit();
        ed.putFloat(spKey, value);
        ed.commit();
    }

    /**
     * 填充boolean型SharedPreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:59
     */
    public void putBooleanSP(String spName, String spKey, boolean value) {
        SharedPreferences sp = initSP(spName);
        SharedPreferences.Editor ed = sp.edit();
        ed.putBoolean(spKey, value);
        ed.commit();
    }

    /**
     * 填充String型SharedPreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:59
     */
    public void putStringSP(String spName, String spKey, String value) {
        SharedPreferences sp = initSP(spName);
        SharedPreferences.Editor ed = sp.edit();
        ed.putString(spKey, value);
        ed.commit();
    }

    /**
     * 填充long型SharedPreferences
     *
     * @author KevinWu
     * create at 2016/2/7 14:59
     */
    public void putLongSP(String spName, String spKey, long value) {
        SharedPreferences sp = initSP(spName);
        SharedPreferences.Editor ed = sp.edit();
        ed.putLong(spKey, value);
        ed.commit();
    }
```

get和push系列方法完成后，下面可以来使用这个工具了，使用起来非常简单：
如果是读取（伪代码）：
``` java
SPUtil sp=new SPUtil(一般用ApplicationContext)
int a=sp.getIntSP(文件名,键名);
```

如果是写入：
``` java
SPUtil mysp = new SPUtil(一般用ApplicationContext);
mysp.putStringSP(文件名,键名,值);
```

最后，再定义一个清空SharedPreferences文件的方法：
``` java
public void clearSP(String spName){
        SharedPreferences sp = initSP(spName);
        SharedPreferences.Editor ed = sp.edit();
        ed.clear();
        ed.commit();
    }
```

到此，一个万能的SharedPreferences工具已经完成了，其实。。也不算万能，但基本常用的功能都具备了。

但其实这个工具有很多缺陷，代码还是显得繁琐了，但对于工具来说代码繁琐还不是最大的问题，最大的问题是调用时显得略不优雅，至少都占了两行代码嘛。。。

下一期将对这个工具类进行重构，提供一个更优雅的SharedPreferences工具方案。
