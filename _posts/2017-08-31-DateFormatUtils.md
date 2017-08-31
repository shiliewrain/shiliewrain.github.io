---
layout : post
title : "使用DateFormatUtils格式化时间造成的bug"
category : bug
tags : bug DateFormatUtils 时间格式化
---
* content
{:toc}

　　今天PD在pre环境中发现列表中的发货时间和订单详情页的发货时间对不上，而我也发现通过计算得到的超时时长对不上。最后发现是我在转换时间戳的时候，使用了错误的方法，将时间转到了UTC时区，难怪时间总是相差8个小时，尴尬...




#### DateFormatUtils

　　在Apache Commons项目的Lang里面，有两个类：DateUtils和DateFormatUtils，专门用于处理时间日期转换。它们在org.apache.commons.lang.time包下。[DateFormatUtilsApi](http://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/time/DateFormatUtils.html)

　　我在转换的时候，使用了formatUTC(date, pattern)这个方法，直接导致我将获得时间减去了8个小时，调到了UTC时区。其实直接使用format(date, pattern)就可以了。

　　在DateFormatUtils中，有以下两个方法：

```java
	public static String format(Date date, String pattern, TimeZone timeZone, Locale locale) {
        FastDateFormat df = FastDateFormat.getInstance(pattern, timeZone, locale);
        return df.format(date);
    }

    public static String format(Calendar calendar, String pattern, TimeZone timeZone, Locale locale) {
        FastDateFormat df = FastDateFormat.getInstance(pattern, timeZone, locale);
        return df.format(calendar);
    }
```

　　调用DateFormatUtils中的相关方法，最后都是调用这两个方法中的其中一个。获取FastDateFormat的一个实例，保存三个参数：时间格式样式、使用的时区和本地的时区，然后调用该实例的format方法。

```java
	public String format(Date date) {
        Calendar c = new GregorianCalendar(mTimeZone);
        c.setTime(date);
        return applyRules(c, new StringBuffer(mMaxLengthEstimate)).toString();
    }

    public String format(Calendar calendar) {
        return format(calendar, new StringBuffer(mMaxLengthEstimate)).toString();
    }

    public StringBuffer format(Calendar calendar, StringBuffer buf) {
        if (mTimeZoneForced) {
            calendar = (Calendar) calendar.clone();
            calendar.setTimeZone(mTimeZone);
        }
        return applyRules(calendar, buf);
    }

    protected StringBuffer applyRules(Calendar calendar, StringBuffer buf) {
        Rule[] rules = mRules;
        int len = mRules.length;
        for (int i = 0; i < len; i++) {
            rules[i].appendTo(buf, calendar);
        }
        return buf;
    }
```

　　到这里，就知道，设置时区对Calendar对象有影响。

#### Calender

　　在Calender中，依据时区来创建一个Calender实例的代码如下：

```java
private static Calendar createCalendar(TimeZone zone, Locale aLocale)
    {
        Calendar cal = null;
        String caltype = aLocale.getUnicodeLocaleType("ca");
        if (caltype == null) {
            // Calendar type is not specified.
            // If the specified locale is a Thai locale,
            // returns a BuddhistCalendar instance.
            if ("th".equals(aLocale.getLanguage())
                    && ("TH".equals(aLocale.getCountry()))) {
                cal = new BuddhistCalendar(zone, aLocale);
            } else {
                cal = new GregorianCalendar(zone, aLocale);
            }
        } else if (caltype.equals("japanese")) {
            cal = new JapaneseImperialCalendar(zone, aLocale);
        } else if (caltype.equals("buddhist")) {
            cal = new BuddhistCalendar(zone, aLocale);
        } else {
            // Unsupported calendar type.
            // Use Gregorian calendar as a fallback.
            cal = new GregorianCalendar(zone, aLocale);
        }
        return cal;
    }
```

　　暂时就只看到这吧。