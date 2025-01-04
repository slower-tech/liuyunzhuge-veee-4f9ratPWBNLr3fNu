
## 组合模式（Composite Pattern）


组合模式（Composite Pattern）是一种结构型设计模式，它用于将对象组织成`树形结构`，以表示`部分-整体`的层次结构。通过组合模式，客户端可以统一对待单个对象和组合对象，从而简化了客户端代码的复杂性。


### 组合模式的核心思想


1. **统一的接口**：通过抽象类或接口将`单个对象`和`组合对象`统一起来；
2. **递归组合**：`组合对象`中可以包含单个对象或其他组合对象；
3. **透明性**：客户端可以`一致地调用`单个对象和组合对象的方法，而无需区分两者的差异。


### 组合模式的角色


1. **组件（Component）**
定义单个对象和组合对象的公共接口，例如通用操作（`add`、`remove`、`getChild`等）。
2. **叶子节点（Leaf）**
表示树形结构中的基本单元，不能再包含其他对象。它实现了组件接口，但不支持添加或移除操作。
3. **组合对象（Composite）**
表示树形结构中的复杂单元，可以包含叶子节点或其他组合对象。它实现组件接口，并负责管理其子对象的操作。


## 示例代码


组合模式解析XML或 HTML 元素结构的代码示例。我们将 XML/HTML 元素看作“部分\-整体”结构，其中：


* **叶子节点(leaf)**：表示没有子节点的元素（如 `![]()` 或 ）。
* **组合节点(Composite)**：表示可以包含其他子元素的元素（如  或 ）。


两种节点使用同一种的顶层抽象，属于同一类对象，统称为元素（节点）。


### 类图


![image](https://img2024.cnblogs.com/blog/1209017/202501/1209017-20250103163725610-1429970605.png)


### 1\. 抽象组件


HTML顶层元素抽象类。你也可以定义一个顶层接口，然后在抽象类中实现基础功能。



```
public abstract class HTMLElement {
    protected String name;

    public HTMLElement(String name) {
        this.name = name;
    }

    public abstract void render(int level);

    public HTMLElement getChild(int index) {
        throw new UnsupportedOperationException();
    }

    // 默认行为：叶子结点禁止新增元素
    public void addChild(HTMLElement element) {
        throw new UnsupportedOperationException();
    }
    // 默认行为：叶子结点禁止移除子元素
    public void removeChild(HTMLElement element) {
        throw new UnsupportedOperationException();
    }

    // 辅助方法：生成缩进
    protected String generateIndent(int level) {
        StringBuilder indent = new StringBuilder();
        for (int i = 0; i < level * 2; i++) {
            indent.append(" "); // 每层缩进2个空格
        }
        return indent.toString();
    }
}

```

### 2\. 组合结点


表示可以包含子元素的HTML标签



```
public class HTMLComposite extends HTMLElement {
    private List children = new LinkedList<>();

    public HTMLComposite(String name) {
        super(name);
    }

    @Override
    public void addChild(HTMLElement element) {
        children.add(element);
    }

    @Override
    public void removeChild(HTMLElement element) {
        children.remove(element);
    }

    @Override
    public HTMLElement getChild(int index) {
        return children.get(index);
    }

    @Override
    public void render(int level) {
        System.out.println(generateIndent(level) + "<" + name + ">");
        for (HTMLElement child : children) {
            child.render(level + 1); // 子节点递归调用
        }
        System.out.println(generateIndent(level) + " + name + ">");
    }
}

```

### 3\.叶子节点


表示没有子元素的HTML标签



```
public class HTMLLeaf extends HTMLElement {
    public HTMLLeaf(String name) {
        super(name);
    }

    @Override
    public void render(int level) {
        System.out.println(generateIndent(level) + "<" + name + " />");
    }
}

```

### 测试



```
public class CompositePatternHTMLDemo {
    public static void main(String[] args) {
        // 创建HTML结构
        HTMLElement html = new HTMLComposite("html");
        HTMLElement body = new HTMLComposite("body");
        HTMLElement div = new HTMLComposite("div");
        HTMLElement img = new HTMLLeaf("img");
        HTMLElement input = new HTMLLeaf("input");
        HTMLElement p = new HTMLLeaf("p");


        // 组合结构
        html.addChild(body);
        body.addChild(div);
        body.addChild(input);
        div.addChild(img);
        div.addChild(p);

        // 渲染HTML结构
        html.render(0);

        // 去除某个节点
        div.removeChild(p);
        html.render(0);
    }
}

```

测试结果：



```
<html>
  <body>
    <div>
      <img />
      <p />
    div>
    <input />
  body>
html>
<html>
  <body>
    <div>
      <img />
    div>
    <input />
  body>
html>

```

从类图或测试类（使用者）中可以看出，使用者直接依赖于具体的类，属于高耦合的一种编程方式。


### 简单优化（结合其它设计模式）


加入一个工厂类来创建组合节点和叶子结点


**类图结构**变为


![image](https://img2024.cnblogs.com/blog/1209017/202501/1209017-20250103163742394-1654785100.png)


**工厂类代码**



```
public class HTMLElementFactory {
    private static Mapextends HTMLElement>> elementRegistry = new HashMap<>();

    static {
        // 注册类
        try {
            HTMLElementFactory.registerElement("composite", (Class <span class="hljs-keyword"extends HTMLElement>) Class.forName("org.example.composite.htmldemo.HTMLComposite"));
            HTMLElementFactory.registerElement("leaf", (Class <span class="hljs-keyword"extends HTMLElement>) Class.forName("org.example.composite.htmldemo.HTMLLeaf"));
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    // 注册类
    public static void registerElement(String type, Class extends HTMLElement clazz) {
        elementRegistry.put(type, clazz);
    }

    // 创建实例
    public HTMLElement createElement(String type, String name) {
        Class <span class="hljs-keyword"extends HTMLElement> clazz = elementRegistry.get(type);
        if (clazz == null) {
            throw new IllegalArgumentException("未知元素类型: " + type);
        }
        try {
            return clazz.getDeclaredConstructor(String.class).newInstance(name);
        } catch (Exception e) {
            throw new RuntimeException("错误创建元素对象: " + e.getMessage(), e);
        }
    }
}

```

测试代码



```
public class CompositePatternHTMLDemo {
    public static void main(String[] args) {
        // 创建HTML结构
//        HTMLElement html = new HTMLComposite("html");
//        HTMLElement body = new HTMLComposite("body");
//        HTMLElement div = new HTMLComposite("div");
//        HTMLElement img = new HTMLLeaf("img");
//        HTMLElement input = new HTMLLeaf("input");
//        HTMLElement p = new HTMLLeaf("p");

        HTMLElementFactory htmlElementFactory = new HTMLElementFactory();
        // 创建对象
        HTMLElement html = htmlElementFactory.createElement("composite", "html");
        HTMLElement body = htmlElementFactory.createElement("composite", "body");
        HTMLElement div = htmlElementFactory.createElement("composite", "div");
        HTMLElement input = htmlElementFactory.createElement("leaf", "input");
        HTMLElement img = htmlElementFactory.createElement("leaf", "img");
        HTMLElement p = htmlElementFactory.createElement("leaf", "p");

        // 组合结构
        html.addChild(body);
        body.addChild(div);
        body.addChild(input);
        div.addChild(img);
        div.addChild(p);

        // 渲染HTML结构
        html.render(0);

        // 去除某个节点
        div.removeChild(p);
        html.render(0);
    }
}

```

测试结果和前面的一样。


除了结合工厂模式外，还可以结合其它设计模式。比如，结合迭代器模式来递归遍历组合树。


## 总结


组合模式（Composite Pattern）是一种结构型设计模式，它用于将对象组织成`树形结构`，以表示`部分-整体`的层次结构。同时可以结合其它设计模式，使组合模式变得更加灵活和高效。


### 优点


1. **透明性**：客户端无需区分单个对象和组合对象。
2. **灵活性**：可以方便地动态组合对象，拓展系统。
3. **符合开闭原则**：添加新的组件类无需修改现有代码。


**开闭原则**的核心是：


* **对扩展开放**：可以通过扩展系统的功能，而不是修改已有的代码来实现新需求。
* **对修改关闭**：已有的代码逻辑不需要因需求变化而被改动。


关于**开闭原则的“修改关闭”主要是指不修改核心业务逻辑**。对于工厂类这样的“管理型”代码，适当的修改是可以接受的，因为它并不属于核心业务逻辑。


### 缺点


1. **复杂性增加**：系统中的类和对象数量可能会增多。
2. **单一职责原则的可能破坏**：组合对象需要管理其子对象，可能职责过多。


### 使用场景


1. 文件系统（文件和文件夹）；
2. GUI（窗口、按钮、文本框等控件）；
3. 公司组织架构（员工与部门）；
4. XML或HTML的元素结构。


![image](https://img2024.cnblogs.com/blog/1209017/202501/1209017-20250103163759563-1937468555.gif)


[什么是设计模式？](https://github.com)


[单例模式及其思想](https://github.com)


[设计模式\-\-原型模式及其编程思想](https://github.com)


[掌握设计模式之生成器模式](https://github.com)


[掌握设计模式之简单工厂模式](https://github.com)


[掌握设计模式之工厂方法模式](https://github.com)


[掌握设计模式\-\-装饰模式](https://github.com)




---


[超实用的SpringAOP实战之日志记录](https://github.com):[flowercloud花云机场官网](https://lizhenzhen.org)


[2023年下半年软考考试重磅消息](https://github.com)


[通过软考后却领取不到实体证书？](https://github.com)


[计算机算法设计与分析（第5版）](https://github.com)


[Java全栈学习路线、学习资源和面试题一条龙](https://github.com)


[软考证书\=职称证书？](https://github.com)


[软考中级\-\-软件设计师毫无保留的备考分享](https://github.com)


