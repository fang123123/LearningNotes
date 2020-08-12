# android基础

#### jdk1.5预置注解

- @Deprecated 方法过时声明
- @SuppressWarning 压制警告
- @Override 方法重写



### XML简介 

Extensible Markup Language可扩展标记语言

#### 作用

- 传递数据
- 保存有关系的数据
- 用作配置文件

#### 组成

- 文档声明

  - 必须写在第一行
  - 声明版本，文件编码（解析时编码，不是保存时编码，默认版本是utf-8）

- 元素

  - 必须有且仅有一个根 标签
  - 标签中换行制表符会保留，占用空间，传输时应去除
  - 命名规范
    - 区分大小写
    - 不能以数字或"_"下划线开头
    - 不能以xml（XML、Xml）开头
    - 不能包含空格
    - 名称中间不能包含冒号

- 属性

  - 也可以用子标签代替

- 注释

- CDATA

  - 里面的内容不会被解析，直接完全输出

  - ```xml
    <![CDATA[<element>this is a content</element>]]>
    结果
    <element>this is a content</element>
    ```

- 特殊字符

  - 需要转义

    - | 特殊字符 | 转移符号 |
      | -------- | -------- |
      | &        | \&amp;   |
      | <        | \&lt;    |
      | >        | \&gt;    |
      | "        | \&quot;  |
      | '        | \&apos;  |

- xml约束
  - 约束出现元素的名称，属性及元素出现的顺序

  - 常用的约束技术

    - XML DTD

      - Document Type Defination 文档类型定义，约束XML的书写规范

        ![image-20200811111900479](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811111900479.png)

      - 引入外部DTD文档

        ![image-20200811112046580](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811112046580.png)

    - XML Schema 

####  DTD定义约束定义规则

定义元素

![image-20200811112425656](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811112425656.png)

 定义属性

![image-20200811114141322](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811114141322.png)

定义实体

 ![image-20200811115545524](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811115545524.png)

![image-20200811115618780](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811115618780.png)

**样例**

![image-20200811114900134](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811114900134.png)





#### XML Schema 

也是用于定义和描述xml文档结构与内容的模式语言，其出现是为了克服DTD的局限性 

![image-20200811120903282](C:\Users\FJ\AppData\Roaming\Typora\typora-user-images\image-20200811120903282.png)