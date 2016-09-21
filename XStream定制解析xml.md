---
date: 2015-08-01 12:54
tags:
- Java
- Xml
- XStream
title: XStream定制解析xml
---

最近项目中用到XStream解析报文，需要解析报文格式为：
```xml
<root>
  <headParam1>param1</headParam1>
  <headParam2>param2</headParam2>
  <body>
    <order>
      <oid>1</oid>
      <info>测试1</info>
    </order>
  </body>
</root>
```
或
```xml
<root>
  <headParam1>param1</headParam1>
  <headParam2>param2</headParam2>
  <body>
    <orderlist>
      <order>
        <oid>1</oid>
        <info>测试1</info>
      </order>
      <order>
        <oid>2</oid>
        <info>测试2</info>
      </order>
    </orderlist>
  </body>
</root>
```
报文头节点都是确定了的，只是body节点下不确定是单值节点order还是多值节点orderlist，所以不能用注解写死body节点的内容。   
通过google资料发现可以定制自己的转换器（Converter），于是就试着写了一个。  
例子如下：
Root.java
```java
import com.thoughtworks.xstream.annotations.XStreamAlias;

/**
 * 根节点，xml解析完后的完整对象
 * Created by Yealove on 2015-07-27.
 */
@XStreamAlias("root")
public class Root {

    /**
     * 头信息
     */
    @XStreamAlias("headParam1")
    private String headParam1;

    /**
     * 头信息
     */
    @XStreamAlias("headParam2")
    private String headParam2;

    @XStreamAlias("body")
    private Body body;

    @Override
    public String toString() {
        return "Root{" +
                "headParam1='" + headParam1 + '\'' +
                ", headParam2='" + headParam2 + '\'' +
                ", body=" + body +
                '}';
    }

    public String getHeadParam1() {
        return headParam1;
    }

    public void setHeadParam1(String headParam1) {
        this.headParam1 = headParam1;
    }

    public String getHeadParam2() {
        return headParam2;
    }

    public void setHeadParam2(String headParam2) {
        this.headParam2 = headParam2;
    }

    public Body getBody() {
        return body;
    }

    public void setBody(Body body) {
        this.body = body;
    }
}
```
Body.java
```java
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.annotations.XStreamConverter;
import com.thoughtworks.xstream.annotations.XStreamOmitField;

/**
 * body节点
 * Created by Yealove on 2015-07-27.
 */
@XStreamAlias("body")
public class Body {

    /**
     * 生成xml时，body节点下子节点的名称
     * 提供给转换器使用
     */
    @XStreamOmitField
    private String nodeName;

    /**
     * 子节点
     */
    @XStreamConverter(BodyConverter.class)
    private Object subNode;

    @Override
    public String toString() {
        return "Body{" +
                "subNode=" + subNode +
                '}';
    }

    public Object getSubNode() {
        return subNode;
    }

    public void setSubNode(Object subNode) {
        this.subNode = subNode;
    }

    public String getNodeName() {
        return nodeName;
    }

    public void setNodeName(String nodeName) {
        this.nodeName = nodeName;
    }
}
```
Order.java
```java
import com.thoughtworks.xstream.annotations.XStreamAlias;

/**
 * 订单
 * Created by Yealove on 2015-07-27.
 */
@XStreamAlias("order")
public class Order {

    @XStreamAlias("oid")
    private String oid;

    @XStreamAlias("info")
    private String info;


    public Order(String oid, String info) {
        this.oid = oid;
        this.info = info;
    }

    @Override
    public String toString() {
        return "Order{" +
                "oid='" + oid + '\'' +
                ", info='" + info + '\'' +
                '}';
    }

    public String getOid() {
        return oid;
    }

    public void setOid(String oid) {
        this.oid = oid;
    }

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }
}
```
BodyConverter.java
```java
import com.thoughtworks.xstream.converters.Converter;
import com.thoughtworks.xstream.converters.MarshallingContext;
import com.thoughtworks.xstream.converters.UnmarshallingContext;
import com.thoughtworks.xstream.io.HierarchicalStreamReader;
import com.thoughtworks.xstream.io.HierarchicalStreamWriter;

import java.util.ArrayList;
import java.util.List;

/**
 * Body节点转换器
 * Created by Yealove on 2015-07-28.
 */
public class BodyConverter implements Converter {

    /**
     * body节点下待解析节点对应Bean的Class
     */
    private Class cls;

    public BodyConverter() {
    }

    public BodyConverter(Class cls) {
        this.cls = cls;
    }

    public void marshal(Object source, HierarchicalStreamWriter writer, MarshallingContext context) {
        Body body = (Body) source;
        writer.startNode(body.getNodeName());
        context.convertAnother(body.getSubNode());
        writer.endNode();
    }

    public Object unmarshal(HierarchicalStreamReader reader, UnmarshallingContext context) {
        Body body = new Body();

        reader.moveDown();
        //当body的子节点为list结尾或复数时，则按List塞值给Body的子节点
        if (reader.getNodeName().endsWith("list") || reader.getNodeName().endsWith("s")) {
            List items = new ArrayList();
            while (reader.hasMoreChildren()) {
                reader.moveDown();
                items.add(context.convertAnother(reader, cls));
                reader.moveUp();
            }
            body.setSubNode(items);
        } else {
            body.setSubNode(context.convertAnother(reader, cls));
        }
        reader.moveUp();
        
        return body;
    }

    public boolean canConvert(Class type) {
        return Body.class.isAssignableFrom(type);
    }

    public Class getCls() {
        return cls;
    }

    public void setCls(Class cls) {
        this.cls = cls;
    }
}
```
测试类XStreamTester.java
```java
import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.io.xml.DomDriver;

import java.util.ArrayList;

/**
 * 测试类
 * Created by Yealove on 2015-07-27.
 */
public class XStreamTester {

    public static void main(String[] args) {

        System.out.println("--------------将对象解析为xml-----------------");
        System.out.println(testBean2Xml());
        System.out.println("----------------------------");

        System.out.println(testList2Xml());



        System.out.println("--------------将xml解析为对象-----------------");
        XStream xStream = new XStream(new DomDriver());
        xStream.registerConverter(new BodyConverter(Order.class));
        xStream.autodetectAnnotations(true);
        xStream.processAnnotations(Root.class);

        System.out.println(xStream.fromXML(testBean2Xml()));
        System.out.println("-----");
        System.out.println(xStream.fromXML(testList2Xml()));
    }

    /**
     * 测试body节点下是单个对象
     *
     * @return
     */
    private static String testBean2Xml() {

        Body body = new Body();
        body.setNodeName("order");
        body.setSubNode(new Order("1", "测试1"));

        Root root = new Root();
        root.setHeadParam1("param1");
        root.setHeadParam2("param2");
        root.setBody(body);

        XStream xStream = new XStream();
        xStream.autodetectAnnotations(true);
        xStream.registerConverter(new BodyConverter(Order.class));

        return xStream.toXML(root);
    }

    /**
     * 测试body节点下是个list
     *
     * @return
     */
    private static String testList2Xml() {

        Body body = new Body();
        body.setNodeName("orderlist");
        ArrayList<Order> orders = new ArrayList<Order>();
        orders.add(new Order("1", "测试1"));
        orders.add(new Order("2", "测试2"));
        body.setSubNode(orders);

        Root root = new Root();
        root.setHeadParam1("param1");
        root.setHeadParam2("param2");
        root.setBody(body);

        XStream xStream = new XStream();
        //xStream.aliasSystemAttribute(null, "class");
        xStream.autodetectAnnotations(true);
        xStream.registerConverter(new BodyConverter(Order.class));

        return xStream.toXML(root);
    }
}
```
   
   
<br/>
   
在写转换器BodyConverter中unmarshal方法时，开始不知道怎么解析多个同级节点，最后查看XStream自带的转换器ArrayConverter的unmarshal方法才得以解决。
ArrayConverter的unmarshal方法如下：
```java
public Object unmarshal(HierarchicalStreamReader reader, UnmarshallingContext context) {
        // read the items from xml into a list
        List items = new ArrayList();
        while (reader.hasMoreChildren()) {
            reader.moveDown();
            Object item = readItem(reader, context, null); // TODO: arg, what should replace null?
            items.add(item);
            reader.moveUp();
        }
        // now convertAnother the list into an array
        // (this has to be done as a separate list as the array size is not
        //  known until all items have been read)
        Object array = Array.newInstance(context.getRequiredType().getComponentType(), items.size());
        int i = 0;
        for (Iterator iterator = items.iterator(); iterator.hasNext();) {
            Array.set(array, i++, iterator.next());
        }
        return array;
    }
```