---
title: 师大+ 技术说明书 第二期：让SPUtil变得更优雅
date: 2016-05-06 
author:  KevinWu
categories: 师大+ 技术说明书
tags: 
	- Java 
	- Android
	- SharedPreferences
	- 工具
---
在第一期中建立了一个工具类——SPUtil，但是作为一个工具来说，那种逻辑使用起来还是不太优雅的感觉，毕竟调用起来代码还有好几行呢。

所以这一期，来讲讲把SPUtil变优雅的事。

在开始讲SPUtil之前，先来了解一下instanceof运算符。
<!--more-->
### instanceof
instanceof是一个判断数据类型或者对象是否是某一个特定类的实例，如果是就返回true，否则返回false。
简单用伪代码剖析下instanceof的原理：
``` java
boolean result;
try{
	T temp = (T)obj;
	result = true;
}catch(ClassCastException e){
	result = false;
}	
```
当调用obj instanceof T时，实际上就是在执行一个类型强转的操作，如果强转中不抛出ClassCastException异常就证明是该类型实例，返回true，否则返回false。

了解了instanceof后，我们看一下原本的SPUtil的get相关操作：
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
可以看到，基本每一种类型都有一种对应的方法，这样的好处是分工明确，但是不好的是这个工具看起来不够简洁，而且当使用工具的数据类型改动时，对应的调用代码也得做出相应的改动，两个字——麻烦。

既然这样，那通过instanceof就可以把这些操作集中到一个方法去，下面看看这个方法：
``` java
/**
     * 获取SharedPreferences文件里面的信息
     *
     * @param context
     * @param file
     * @param key
     * @return
     */
    public static Object get(Context context, String file, String key, Object defaultValue) {
        SharedPreferences sp = context.getSharedPreferences(file, Context.MODE_PRIVATE);
        if (defaultValue instanceof Integer) {
            return sp.getInt(key, (int) defaultValue);
        } else if (defaultValue instanceof Float) {
            return sp.getFloat(key, (float) defaultValue);
        } else if (defaultValue instanceof Boolean) {
            return sp.getBoolean(key, (boolean) defaultValue);
        } else if (defaultValue instanceof Long) {
            return sp.getLong(key, (long) defaultValue);
        } else if (defaultValue instanceof String) {
            return sp.getString(key, (String) defaultValue);
        }
        return null;
    }
```
可以看到，这是一个静态方法，连初始化context的步骤都省略了，作为一个工具类来说，这样的好处不言而喻，经过这样的简化，调用时直接一句代码搞掂！

假如要调用这个方法取得一个int型数据，伪代码如下：
``` java
int a=SPUtil.get(一般用ApplicationContext,文件名,键名,默认值);
```

同样，我们再来定义put方法，也不是一系列，而是简化成一个：
``` java
 /**
     * 往SharedPreferences文件里面写入信息的方法，根据参数类型决定写入的信息
     *
     * @param context
     * @param file
     * @param key
     * @param object
     */
    public static void put(Context context, String file, String key, Object object) {
        SharedPreferences sp = context.getSharedPreferences(file, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sp.edit();
        if (object instanceof Integer) {
            editor.putInt(key, (int) object);
        } else if (object instanceof Float) {
            editor.putFloat(key, (float) object);
        } else if (object instanceof Boolean) {
            editor.putBoolean(key, (boolean) object);
        } else if (object instanceof Long) {
            editor.putLong(key, (long) object);
        } else if (object instanceof String) {
            editor.putString(key, (String) object);
        } else {
            editor.putString(key, object.toString());
        }
        editor.commit();
    }
```

到此，一个比起第一期更优雅的SPUtil工具类就完成了。