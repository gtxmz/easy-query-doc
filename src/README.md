---
home: true
icon: home
title: 项目主页
heroImage: /logo.svg
heroText: easy-query
tagline: 🚀 java下唯一一款同时支持强类型对象关系查询和强类型SQL语法查询的ORM,拥有对象模型筛选、隐式子查询、隐式join、显式子查询、显式join,支持Java/Kotlin
actions:
  - text: 开始使用 →
    link: /startup/
    type: primary

  - text: 爱心支持💡
    link: /support

# features:

# - title: 零依赖
#   icon: async
#   details: 核心包无任何依赖,没有历史包袱,全部自行实现
#   # link: /easy-query-doc/query/relation

# - title: 零调用
#   icon: copy
#   details: 使用lambda表达式缓存实现bean对象的”零“调用耗时赋值和获取值,而不是普通的高频反射
#   # link: https://theme-hope.vuejs.press/zh/layout/

# - title: 零SQL
#   icon: object
#   details: ORM 框架可以屏蔽 SQL 语句的复杂性，通过提供面向对象的查询语言、方法，简化数据查询和操作强类型更加安全
#   # link: https://theme-hope.vuejs.press/zh/layout/

# - title: 零配置
#   icon: tool
#   details: ORM 框架可以通过自动扫描实体类和数据库表之间的映射关系，无需繁琐的配置文件。
#   # link: https://theme-hope.vuejs.press/zh/layout/

# - title: 多语言
#   icon: language
#   details: 支持java、kotlin两种语言,并且提供了两种语言相似的api，维护同一套内部接口保证两边api仅仅是针对核心功能的扩展

# - title: 强类型
#   icon: structure
#   details: 如果一款orm不包含泛型约束,那么这个orm就没有必要存在,因为他和手写sql没有任何区别,无法在编译时为您提供错误信息,帮您做到强类型语言该有的提示

# - title: 分库分表
#   icon: condition
#   details: 一款自带分库分表读写分离的orm,拥有和市面上大部分分库分表框架抗衡的能力,并且抽象了业务逻辑可以让用户完全自定义自己的业务逻辑来实现

# - title: 列加密
#   icon: lock
#   details: 自带数据库列加密,并且支持模糊查询实现高性能而不是单纯的数据库函数调用,并且用户可以自定义自己的加密函数

# - title: VO查询
#   icon: search
#   details: 框架让VO的能得到了进一步的提升,而不是单纯的数据交换对象,用户可以针对VO的字段进行自动化列选择查询,并且支持自定义VO对象让其更加丰富
#   link: /query/select-column

# - title: 差异更新
#   icon: change
#   details: 市面上基本上大部分java orm仅支持全量更新或者null列非null列更新,而不支持差异更新,框架提供差异更新追踪数据变化情况,提高更新sql的强壮性
#   link: /basic/update#_3-差异更新

# - title: 原子列更新
#   icon: react
#   details: 实体对象更新如updateById是一个用户方便但是无差别更新的方法,但是框架提供了差异更新让其上升到了一个纬度并且在没有乐观锁的情况下支持库存数量级别的原子更新

# - title: 关联查询
#   icon: style
#   details: 框架不仅支持多表原始sql的join模式,也支持数据库对象模型的一对一、一对多、多对一、多对多模式,并且支持关联查询的自定义过滤,逻辑删除等一系列特性
#   link: /query/relation

copyright: false
footer: 使用 <a href="https://theme-hope.vuejs.press/" target="_blank">VuePress Theme Hope</a> 主题 | MIT 协议, 版权所有 © 2019-present Mr.Hope
---

<!-- <video src="/videos/EQ 插件支持 DTO 实体 Column 比对.mp4" muted autoplay id='v' width="800"></video> -->

::: code-tabs
@tab 单表查询

```java
//筛选用户名称包含小明的
List<SysUser> users = easyEntityQuery.queryable(SysUser.class)
    .where(s -> s.name().like("小明")).toList();
//筛选用户名称包含小明并且是2020年以前创建的
List<SysUser> users = easyEntityQuery.queryable(SysUser.class)
    .where(s -> {
            s.name().like("小明");
            s.createTime().lt(LocalDateTime.of(2020,1,1,0,0));
    }).toList();
//筛选用户名称包含小明的或者名称包含小红的
List<SysUser> users = easyEntityQuery.queryable(SysUser.class)
    .where(s -> {
        s.or(() -> {
            s.name().like("小明");
            s.name().like("小红");
        });
    }).toList();
```

@tab 隐式 join 筛选 🔥

```java
//user和address一对一
//查询杭州或绍兴的用户
List<SysUser> userInHz = easyEntityQuery.queryable(SysUser.class)
                .where(u -> {
                    //隐式子查询会自动join用户表和地址表
                    u.or(()->{
                      //根据条件是否生效自动添加address表的join
                        u.address().city().eq("杭州市");
                        u.address().city().eq("绍兴市");
                    });
                }).toList();


//查询用户名叫小明并且家住杭州的
List<SysUser> userInHz = easyEntityQuery.queryable(SysUser.class)
                .where(u -> {
                    u.name().eq("小明");
                    //隐式子查询会自动join地址表
                    //根据条件是否生效自动添加address表的join
                    //比如eq("")和eq("杭州生成的表不存在address和city的区别")
                    u.address().city().eq("杭州市");
                }).toList();
```

@tab 隐式子查询 🔥

```java
//user和role多对多
//筛选用户角色是管理员的
List<SysUser> adminUsers = easyEntityQuery.queryable(SysUser.class)
            .where(s -> {
                //筛选条件为角色集合里面有角色名称叫做管理员的
                s.roles().where(role -> {
                    role.name().eq("管理员");
                }).any();
            }).toList();

//匿名返回用户id和用户所拥有的角色数量
List<Draft2<String, Long>> userIdAndRoleCount = easyEntityQuery.queryable(SysUser.class)
        .where(user -> user.name().like("小明"))
        .select(user -> Select.DRAFT.of(
                user.id(),
                user.roles().count()
        )).toList();
```

@tab join 多表

```java
//匿名对象
List<Draft3<String, LocalDateTime, String>> userInfo = easyEntityQuery.queryable(SysUser.class)
        .leftJoin(SysUserAddress.class, (user, addr) -> user.id().eq(addr.userId()))
        .where((user, addr) -> {
            user.name().like("小明");
            addr.city().eq("杭州");
        })
        .select((user, addr) -> Select.DRAFT.of(
                user.id(),
                user.createTime(),
                addr.area()
        )).toList();


List<UserDTO> userInfo = easyEntityQuery.queryable(SysUser.class)
        .leftJoin(SysUserAddress.class, (user, addr) -> user.id().eq(addr.userId()))
        .where((user, addr) -> {
            user.name().like("小明");
            addr.city().eq("杭州");
        })
        .select(UserDTO.class,(user, addr) -> Select.DRAFT.of(
                user.FETCHER.id().createTime(),
                addr.area()
        )).toList();
```

@tab 显式子查询

```java

List<SysUser> userIn = easyEntityQuery.queryable(SysUser.class)
        .where(user -> {
            user.id().in(
                    easyEntityQuery.queryable(SysRole.class)
                            .select(s -> s.id())
            );
        }).toList();


List<SysUser> userExists = easyEntityQuery.queryable(SysUser.class)
        .where(user -> {

            user.expression().exists(()->{
                return easyEntityQuery.queryable(SysRole.class)
                        .where(role -> {
                            role.name().eq("管理员");
                            role.id().eq(user.id());
                        });
                    });
        }).toList();


//匿名返回用户id和用户所拥有的角色数量
List<Draft2<String, Long>> userIdAndRoleCount1 = easyEntityQuery.queryable(SysUser.class)
        .where(user -> user.name().like("小明"))
        .select(user -> Select.DRAFT.of(
                user.id(),
                user.expression().subQuery(() -> {
                    return easyEntityQuery.queryable(SysRole.class)
                            .where(r -> {
                                r.expression().exists(()->{
                                    return easyEntityQuery.queryable(UserRole.class)
                                            .where(u -> {
                                                u.roleId().eq(r.id());
                                                u.userId().eq(user.id());
                                            });
                                });
                            })
                            .selectCount();
                })
        )).toList();
```

@tab 结构化数据返回 🔥

```java

//可以直接筛选出结构化DTO
List<StructSysUserDTO> users = easyEntityQuery.queryable(SysUser.class)
        .where(s -> s.name().like("小明"))
        .selectAutoInclude(StructSysUserDTO.class).toList();


{
	"id": "...",
	"createTime": "...",
	"roles": [{
		"id": "...",
		"name": "",
		"menus": [{
			"id": "....",
			"name": "...."
		}, {
			"id": "....",
			"name": "...."
		}]
	}]
}

//返回用户角色和菜单

@Data
public class StructSysUserDTO {

    private String id;
    private String name;
    private LocalDateTime createTime;
    @Navigate(value = RelationTypeEnum.ManyToMany)
    private List<InternalRoles> roles;
    @Data
    public static class InternalRoles {
        private String id;
        //....
        @Navigate(value = RelationTypeEnum.ManyToMany)
        private List<InternalMenus> menus;
    }
    @Data
    public static class InternalMenus {
        private String id;
        private String name;
        //....
    }
}
```

@tab 结构化穿透 🔥

```java

快速返回用户拥有的菜单,因为用户和菜单中间由角色进行关联并且两者都是多对多所以如果需要自行实现那么是非常麻烦的一件事情

用户和菜单之间隔着角色的多对多所以如果想要获取用户的菜单id直接可以通过这种方式快速筛选

方式1 仅获取用户拥有的菜单id

List<String> menuIds = easyEntityQuery.queryable(SysUser.class)
        .where(s -> s.name().like("小明"))
        .toList(s -> s.roles().flatElement().menus().flatElement().id());


方式2 仅获取用户拥有的菜单id和菜单名称


List<SysMenu> menuIdNames = easyEntityQuery.queryable(SysUser.class)
        .where(s -> s.name().like("小明"))
        .toList(s -> s.roles().flatElement().menus().flatElement(x->x.FETCHER.id().name()));

List<SysUserFlatDTO> users = easyEntityQuery.queryable(SysUser.class)
        .where(s -> s.name().like("小明"))
        .selectAutoInclude(SysUserFlatDTO.class).toList();

@Data
public class SysUserFlatDTO {
    private String id;
    private String name;
    private LocalDateTime createTime;

    //穿透获取用户下的roles下的menus下的id 如果穿透获取的是非基本类型那么对象只能是数据库对象而不是dto对象
    @NavigateFlat(value = RelationMappingTypeEnum.ToMany,mappingPath = {
            SysUser.Fields.roles,
            SysRole.Fields.menus,
            SysMenu.Fields.id
    })
    private List<String> menuIds;

//非基本对象也可以直接返回数据库对象
//    @NavigateFlat(value = RelationMappingTypeEnum.ToMany,mappingPath = {
//            SysUser.Fields.roles,
//            SysRole.Fields.menus
//    })
//    private List<SysMenu> menu;

    @NavigateFlat(value = RelationMappingTypeEnum.ToMany,mappingPath = {
            SysUser.Fields.roles,
            SysMenu.Fields.id
    })
    private List<String> roleIds;
}


```

@tab 原生 sql

```java
//原生sql执行

List<SysUser> filterTime = easyEntityQuery.queryable(SysUser.class)
        .where(user -> {
            user.expression().sql("{0} > {1}", c -> {
                c.value(LocalDateTime.now()).expression(user.createTime());
            });
        }).toList();

//原生sql片段


List<Draft2<String, String>> idAndName = easyEntityQuery.queryable(SysUser.class)
        .where(user -> {
            user.createTime().gt(LocalDateTime.now());
        }).select(user -> Select.DRAFT.of(
                user.id(),
                user.expression().sqlSegment("IFNULL({0},'')", c -> c.expression(user.name())).setPropertyType(String.class)
        )).toList();

```

:::

<div align="center">
    <a target="_blank" href="https://central.sonatype.com/search?q=easy-query">
        <img src="https://img.shields.io/maven-central/v/com.easy-query/easy-query-all?label=Maven%20Central" alt="Maven" />
    </a>
    <a target="_blank" href="https://www.apache.org/licenses/LICENSE-2.0.txt">
		<img src="https://img.shields.io/:license-Apache2-blue.svg" alt="Apache 2" />
	</a>
    <a target="_blank" href="https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html">
		<img src="https://img.shields.io/badge/JDK-8-green.svg" alt="jdk-8" />
	</a>
    <a target="_blank" href="https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html">
		<img src="https://img.shields.io/badge/JDK-11-green.svg" alt="jdk-11" />
	</a>
    <a target="_blank" href="https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html">
		<img src="https://img.shields.io/badge/JDK-17-green.svg" alt="jdk-17" />
	</a>
    <br />
        <img src="https://img.shields.io/badge/SpringBoot-v2.x-blue">
        <img src="https://img.shields.io/badge/SpringBoot-v3.x-blue">
        <a target="_blank" href='https://gitee.com/noear/solon'><img src="https://img.shields.io/badge/Solon-v2.x-blue"></a>
    <br />
    <a target="_blank" href='https://gitee.com/xuejm/easy-query'>
		<img src='https://gitee.com/xuejm/easy-query/badge/star.svg' alt='Gitee star'/>
	</a>
    <a target="_blank" href='https://github.com/xuejmnet/easy-query'>
		<img src="https://img.shields.io/github/stars/xuejmnet/easy-query.svg?logo=github" alt="Github star"/>
	</a>
    
</div>

## 🔔 QQ 群: 170029046

<br/>

## 🔔 交流 QQ 群

::: center
<img src="/qrcode.jpg" alt="群号: 170029046" class="no-zoom" style="width:200px;">

#### EasyQuery 官方 QQ 群: 170029046

## github 仓库

[easy-query](https://github.com/xuejmnet/easy-query)

## gitee 仓库

[easy-query](https://gitee.com/xuejm/easy-query)

## 许可证

[Apache-2.0 License](https://github.com/xuejmnet/easy-query/blob/main/LICENSE)

## 文档主题

[vuepress-theme-hope](https://vuepress-theme-hope.github.io/)

<link rel="stylesheet" href="/index.css">



<!-- ```mermaid
erDiagram
    CUSTOMER {
        int customer_id
        string name
        string email
    }

    ORDER {
        int order_id
        date order_date
        float amount
    }

    CUSTOMER ||--o{ ORDER : places
    ORDER }o--|{ PRODUCT : contains
    PRODUCT {
        int product_id
        string name
        float price
    }
``` -->
