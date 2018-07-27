* 1\. 方法前不加public, protected, private，为package可见，在包外，即使在子类里面也不可见；
* 2\. 方法的可见性，在子类中重载时，不能更加weaker。public > protected > package >private. 
* 3\. [demo](https://github.com/wangsenyuan/demo/tree/master/java-visibility-demo)