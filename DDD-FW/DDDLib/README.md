# [DDDLib](https://github.com/dayatang/dddlib)

```md
是一个领域驱动设计（Domain Driven Design，简称DDD）类库
```
## 目标
```md
为基于DDD的开发范式提供基本的接口和抽象，实现一致性。
支持业务代码和技术代码分离。使领域层代码纯粹表达业务概念和业务规则，将具体技术隔离出去。
隔离业务代码对对IoC容器和持久化框架等等基础设施的依赖。
    可以自由切换IoC容器（Spring、Guice、TapestryIoC等）和持久化框架（JPA，Hibernate等）的实现。
减轻开发人员的工作负担，降低开发人员的“概念重量”。绝大多数开发人员只需要了解dddlib-domain模块，
    而且只需要了解dddlib-domain中的几个类：Entity、EntityRepository、InstanceFactory和四种查询对象。
提供程序设计中经常用到的工具，例如Excel导入导出、系统配置、规则引擎封装，等等。
```
