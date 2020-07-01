---
layout: post
title: 使用OpenOffice将excel转pdf折行问题
tags: openoffice
author: neal

---
* content
{:toc}
### **excel转pdf出现折行问题**

**原因**

OpenOffice默认的纸张大小为A4，当尺寸超过A4纸时，会出现折行问题（基于jodconverter）；

可以通过修改XPrintable的`PageFormat`和`PageSize`解决

处理逻辑主要在抽象类`org.jodconverter.AbstractConversionTask`

中的`execute(final OfficeContext context)`方法；

```java
@Override
  public void execute(final OfficeContext context) throws OfficeException {

      /**省略了try-catch及部分代码*/
   	  XComponent document = null;
   	  //加载文件
      document = loadDocument(context);
      //处理文件(抽象方法)
      modifyDocument(context, document);
      //保存文件
      storeDocument(document);
   
  }
```

其中`modifyDocument(context,document)`方法是抽象方法，另外两个是私有方法，所以可以重写此方法：

```java
 @Override
    protected void modifyDocument(XComponent document) throws OfficeException {
        XRefreshable refreshable = UnoRuntime.queryInterface(XRefreshable.class, document);
        if (refreshable != null) {
            refreshable.refresh();
        }
        //获取XPrintable实例，修改PagerFormat和PaperSize的Value即可
        XPrintable xPrintable = UnoRuntime.queryInterface(XPrintable.class, document);
        PropertyValue[] printer = xPrintable.getPrinter();
        for (PropertyValue propertyValue:printer){
            if(propertyValue.Name.equalsIgnoreCase("PaperFormat")){
               //PaperFormat.USER表示用户自定义
               propertyValue.Value= PaperFormat.USER;
            }
            if(propertyValue.Name.equalsIgnoreCase("PaperSize")){
                Size A2 = new Size(42000, 59400);
                propertyValue.Value=A2;
            }
        }
        try {
            //重设printer值
            xPrintable.setPrinter(printer);
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        }
    }
```

`com.sun.star.view.PaperFormat` 用于定义纸张格式，不过经过测试好像`PaperSize` 只能定义标准纸张尺寸，不太确定；



*参考:*

1.  http://openofficejava.blogspot.com/2009/05/openofficeorg-api.html

2. https://blog.csdn.net/liuhualiang/article/details/14094019?utm_source=blogxgwz6