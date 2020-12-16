## Scope

### Scope范围

| 作用域名称  | 作用域范围                             |
| ----------- | -------------------------------------- |
| singleton   | 在一个BeanFactory中是唯一的一个        |
| prototype   | 每次获取SpringBean都会生成一个新的对象 |
| request     | 将SpringBean存储在ServletRequest中     |
| session     | 将SpringBean存储在HttpSession中        |
| application | 将SpringBean存储在ServletContext中     |





### Prototype

Singleton的生命周期之前已经分析的很彻底了，本小节主要分析Prototype的生命周期。

















