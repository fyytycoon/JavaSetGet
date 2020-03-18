### Tomcat & wildfly

tomcat=>sun=>开源

wildfly(Jboos8.0)=>red hat=>部分开源

### drools

优势:

1. 业务逻辑和数据分离

不足:

1. 复杂性提高
2. spring-boot-devtools冲突

https://blog.csdn.net/pzysoft/article/details/81155607

KieServices:

​      通过服务注册表，提供整体的入口，可以创建Container等。

KieContainer:

​      KieContainer就是一个KieBase的容器，可以根据kmodule.xml 里描述的KieBase信息来获取具体的

KieSession:

​      KieContainer可以承载多个KieModule，也就是装载不同的规则集。

KieBase:  是一个知识仓库，包含了若干的规则、流程、方法等

1. 有状态KieSession

   KieSession会在多次与规则引擎进行交互中，维护会话状态，type属性值是stateful，最后需要清理KieSession维护的状态，调用dispose()方法。

 2.无状态StatelessKieSession

​	StatelessKieSession隔离了每次与规则引擎的交互，不会维护会话状态，无副作用，type属性值是stateless；应用场景：数据校验，运算，数据过滤，任何能被描述成函数或公式的规则。
