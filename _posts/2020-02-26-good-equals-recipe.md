---
layout: post
title: 实现高质量equals的诀窍
tags: java基础
author: neal

---
* content
{:toc}

## 实现高质量equals的诀窍
1. **使用 "==" 操作符检查 "参数是否为这个对象的引用"：**
如果是，则返回true。在进行比较操作比较昂贵时，可以这样优化；
因为如果是对象的引用，无疑可以确定equals是相等的。
2. **使用`instanceof`操作符检查"参数是否为正确的类型":**
如果不是，则返回false。所谓正确的类型是指equals方法所在的类，
或者该类实现的接口。另外，`instanceof`操作符的第一个参数如果是null，
则一定返回false，所以equals方法入参(例如obj)没必要进行非空判断。




3. **把参数转换为正确的类型：**
因为之前进行过`instanceof`确认，可以确保转换成功。
4. **对于该类的每个关键的域(即对应属性),检查参数中的域是否与该对象中的对应域相匹配。**

## equals实现约定
1. **自反性：**
对于所有非`null`的引用值，`x.equals(x)`必须返回`true`。
2. **对称性：**
对于非`null`的引用值，x和y，当且仅当`x.equals(y)`返回`true`时，
`y.equals(x)`必须返回`true`。需要注意的一点是，当x和y存在继承关系并且子类增加了新的组件时，
很难同时保证`equals`约定的成立，必要时可考虑`组合`的方式进行实现。例如：
```
/**
 *父类
 */
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Point)) {
            return false;
        }
        Point p = (Point) obj;
        return p.x == x && p.y == y;
    }
}
/**
 *继承了Point
 */
public class ColorPoint extends Point {

    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    /**
     * 违反对称性，用Point和ColorPoint实例进行相互调用
     * 可能会出现不同的结果
     * @param obj
     * @return
     */
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof ColorPoint)) {
            return false;
        }
        return super.equals(obj) && ((ColorPoint) obj).color == color;
    }

    /**
     * 此即，无法在扩展可实例化的类的同时，既增加组件(此例中的color)，
     * 又保留equals的约定；
     * 可行的解决办法是通过组合优于继承的方式，将Point和Color作为ColorPoint的属性
     * 而不是继承Point类，
     * 这样equals方法可以分别调用Color和Point类的equals来完成
     * @param args
     */
    public static void main(String[] args) {
        Point p = new Point(0, 0);
        ColorPoint cp = new ColorPoint(0, 0, Color.RED);
        //进行相互调用，会出现不同的结果
        System.err.println(p.equals(cp));
        System.err.println(cp.equals(p));
        System.err.println(p.getClass());
        System.err.println(cp.getClass());


    }
}
```
3. **传递性：**
对于非`null`的任意引用值x、y和z，如果`x.equals(y)`返回`true`，并且`y.equals(z)`返回`true`，
那么`x.equals(z)`也必须返回`true`。
4. **一致性：**
对于非`null`的引用值，那么在同一个环境中，如果两个对象没有被修改，无论调用多少次，都应该返回一致的结果。
即不要使equals依赖不可靠的资源。反例为`java.net.URL`的`equals`方法，其依赖
于URL中主机对应ip地址，将一个主机名抓变成ip地址需要访问网络，随着时间推移就不能确保ip地址始终不变，
也就是说依赖了不可靠的资源。
5. **非空性：**
对于任何非`null`的引用值x，`x.equals(null)`必须返回`false`。

---
引用：《Effective Java》第三版。



