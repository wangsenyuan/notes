1. 单例模式
   a. 目的是为了在同一个jvm中只有一个该类的实例；
   b. 适用于无状态，功能性，需多次被调用的对象；
   c. 实现方式主要有以下两种方式：
      1. private constructor， 
         static intance,
         static method to return the instance
      2. lazy intialization
         double instance null check
         sychronized when new
         volatile is used to avoid java optimize code
2. 工厂模式
   a. 主要有两个作用，隐藏初始化的细节，后续可以替换实现；或者初始化的过程很繁杂，统一处理，保证对象正确；
   b. 简单的实现，只需要一个工厂方法，具体的实现在该方法内实现；工厂本身也具有层次结构，Abstract Factory只声明工厂方法，Concret Factory构造具体的实例。这样要替换实现，只需要替换工厂即可；也可以用于单元测试；
