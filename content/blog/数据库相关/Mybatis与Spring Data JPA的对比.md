---
title: "mybatis与SpringDataJPA的对比"
meta_title: ""
description: "除了面试背诵区别之外的一些对比"
date: 2024-02-15T05:00:00Z
image: "/images/database/labixiaoxin.png"
categories: ["mybatis", "spring data jpa"]
author: "Gavain Juan"
tags: ["mybatis", "spring data jpa"]
draft: false
---
# ORM
Object Relation Mapping
用于在面向对象编程语言和关系型数据库之间建立一种映射关系。
它允许开发人员使用面向对象的方式来操作数据库，而不是直接编写 SQL 语句。

Hibernate是基于ORM实现的，Spring Data JPA是基于Hibernate实现的。
因为mybatis需要编写SQL,但是能够实现查询结果到对象的映射，所以有把mybatis称为半ORM映射的说法。
# JPA
JPA（Java Persistence API） JPA是ORM在java中的一种实现。
它不是一个具体的实现框架，而是定义了一系列的接口和规则，用于在 Java 环境中进行对象关系映射操作。这些规则包括如何定义实体类、实体类之间的关系、如何执行持久化操作等诸多方面。
JPA 作为一种规范，为不同的 ORM 框架提供了统一的接口标准。这意味着不同的 JPA 实现框架（如 Hibernate、EclipseLink 等）都需要遵循 JPA 定义的接口和规则来实现对象关系映射功能。开发人员在使用这些框架时，可以基于相同的 JPA 接口进行编程，而不用担心底层 ORM 框架的具体实现差异。
mybatis并没有遵循JPA的规范。
## 表与类，列与字段的映射
`@Entity`，`@Table`,`@Id`,`@Column` 建立表与实体类的映射
`@OneToOne`、`@OneToMany`、`@ManyToOne`和`@ManyToMany`等注解用于映射一对一、一对多、多对一和多对多的关系。
mybatis构造的返回结果通常都是平铺的，而jpa能体现出对象的包含关系。
```java
@Entity  
@SuperBuilder  
@Table(name = "isp_XX_Item")  
public class User {  
    @Id  
    @GeneratedValue(generator = "uuid_generator")  
    @GenericGenerator(name = "uuid_generator", strategy = "uuid")  
    private String id;  
    @Column(nullable = false)  
    private String name;  
    @Enumerated(EnumType.STRING)  
    @Column(nullable = false)  
    private String username;
    private String password;  
    @OneToOne  
    @JoinColumn(nullable = false, name = "item_key_id", referencedColumnName = "id")  
    private KeyEntity key;  

}
```

>[!INFO] Spring Data JPA要求字段和属性名称存在映射关系，可以配置通用的映射：比如驼峰映射成下划线，也可以通过@Column配置将不相关的属性和字段绑定在一起。

mybatis里面需要自己进行SQL编写，参数映射，结果映射，灵活，但是稍麻烦，以及要考虑SQL注入问题，Spring Data JPA通常情况下通过参数绑定的方式填充查询条件，只要不手写原生SQL，是不会出现SQL注入问题的（mybatis里面#避免SQL注入）。

## 方法名查询&查询 JPQL & Criteria API
### 方法名生成查询

```java
SysSettingEntity  findByCategory(String category);
Optional<SysSettingEntity> findByCategory(String category);
```
### JPQL
```java
// 查询
@Query("select distinct vi.XX.id from XXXXEntity vi where vi.XX.delFlag=:deleteFlag and vi.delFlag=:deleteFlag and vi.XX.id in :XXIds")  
List<String> findListByXXIdListAndDeleteFlag(@Param("XXIds") List<String> XXIds,@Param("deleteFlag") DeleteFlag deleteFlag);
```
### 原生SQL
```java
 // 使用原生SQL实现的方法
    @Query(value = "SELECT DISTINCT v.id " +
            "FROM XX_item vi " +
            "JOIN XX v ON vi.XX_id = v.id " +
            "WHERE v.del_flag = :deleteFlag " +
            "AND vi.del_flag = :deleteFlag " +
            "AND v.id IN (:XXIds)", nativeQuery = true)
    List<String> findListByXXIdListAndDeleteFlag(@Param("XXIds") List<String> XXIds, @Param("deleteFlag") String deleteFlag);
}
```

### Criteria API
复杂查询还是mybatis使用方便。
下面这种纯用java代码写sql,算是正统的复杂查询的编写方式。
```java
 @Override  
    public CustomPage<XXXXEntity> findByPage(XXXXSearchReq XXXXSearchReq) throws ServiceException {  
        Sort sort = Sort.by(StringUtils.isNotBlank(XXXXSearchReq.getOrder())  
                        && Objects.equals(XXXXSearchReq.getOrder(), "descending") ? Sort.Direction.DESC : Sort.Direction.ASC,  
                StringUtils.isBlank(XXXXSearchReq.getOrderByColumn()) ? "name" : XXXXSearchReq.getOrderByColumn());  
        Pageable pageable = PageRequest.of(XXXXSearchReq.getPageNum(), XXXXSearchReq.getPageSize(), sort);  
        Page<XXXXEntity> XXPage = XXXXRepository.findAll(buildSpecification(XXXXSearchReq), pageable);   
        return XXCustomPage;  
    }  
  
    public Specification<XXXXEntity> buildSpecification(XXXXSearchReq XXXXSearchReq) {  
        return new Specification<XXXXEntity>() {  
            @Override  
            public Predicate toPredicate(Root<XXXXEntity> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {  
                List<Predicate> predicates = new ArrayList<>();  
                if (StringUtils.isNotBlank(XXXXSearchReq.getAll())) {  
                    String keyword = "%" + XXXXSearchReq.getAll() + "%";  
                    Predicate nameCondition = criteriaBuilder.like(root.get("name"), keyword);  
                    Predicate websiteCondition = criteriaBuilder.like(root.get("website"), keyword);  
                    Predicate usernameCondition = criteriaBuilder.like(root.get("username"), keyword);  
                    predicates.add(criteriaBuilder.or(nameCondition, websiteCondition, usernameCondition));  
                } else if (StringUtils.isNotBlank(XXXXSearchReq.getName())) {  
                    String keyword = "%" + XXXXSearchReq.getName() + "%";  
                    Predicate nameCondition = criteriaBuilder.like(root.get("name"), keyword);  
                    predicates.add(nameCondition);  
                } else if (StringUtils.isNotBlank(XXXXSearchReq.getWebsite())) {  
                    String keyword = "%" + XXXXSearchReq.getWebsite() + "%";  
                    Predicate nameCondition = criteriaBuilder.like(root.get("website"), keyword);  
                    predicates.add(nameCondition);  
                }  
                LoginUser loginUser = SecurityUtils.getLoginUser();  
                predicates.add(criteriaBuilder.equal(root.get("XX").get("id"), XXXXSearchReq.getXXId()));  
                predicates.add(criteriaBuilder.equal(root.get("delFlag"), DeleteFlag.NORMAL));  
                predicates.add(criteriaBuilder.equal(root.get("updateBy"), loginUser.getUserId()));  
                return criteriaBuilder.and(predicates.toArray(new Predicate[predicates.size()]));  
            }  
        };  
    }
```
对于不太复杂的场景中的if判断可以尝试下这种方式
```java
@Query("from XXXX vi where vi.XX.id=:XXId and(:keyword is  null OR(:keyword is not null and vi.name=:keyword)) and vi.delFlag=:deleteFlag")  
Page<XXXXEntity> findByPage(@Param("XXId") String XXId, @Param("keyword") String keyword, @Param("deleteFlag") DeleteFlag deleteFlag, Pageable pageable);
```
第三方扩展QueryDSL
```java
  QEmployee employee = QEmployee.employee;
  BooleanBuilder builder = new BooleanBuilder();
  for (String name : names) {
      builder.or(employee.name.equalsIgnoreCase(name));
  }
  if (id != null) {
      builder.and(employee.id.equals(id))
  }
  queryFactory.selectFrom(employee).where(builder).fetch();
```
## 缓存关联
- **一级缓存（L1 Cache）**：JPA 的`EntityManager`自带一级缓存，在一个`EntityManager`的生命周期内，它会缓存已经加载的实体对象。例如，在同一次数据库会话中，多次查询同一个主键的实体对象，第一次查询后，后续查询可以直接从一级缓存中获取，提高了查询效率。
- **二级缓存（L2 Cache）**：Spring Data JPA支持二级缓存，它是跨`EntityManager`的缓存。对于一些经常查询且数据不经常变化的实体对象，可以通过配置二级缓存来进一步提高性能。例如，对于系统配置信息这样的实体对象，使用二级缓存可以减少数据库的访问次数。
mybatis也有类似的功能。

# 数据库的兼容性问题
## 不同数据库的特殊的语法
**增删改查中**差异大且常用的只要就是分页，排序，函数，主键。
### 分页，排序
```SQL
-- MySQL 
SELECT * FROM users ORDER BY id LIMIT 20, 10; 
-- PostgreSQL 
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;
-- PostgreSQL 10
SELECT * FROM orders ORDER BY order_date OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```
mybatis 集成 page helper
mybatis plus的 MybatisPlusInterceptor
Spring Data jpa的 Pageable  
都是相同的原理：通过不同数据库的方言得到不同的语法
### 函数
SQL标准规定的函数比如聚合函数，字符串函数，现在主流的数据库都有支持，可能一些早期的数据库版本存在支持问题。
```SQL
-- 拼接
SELECT CONCAT(column1, column2) FROM table_name;
--早期oracle
SELECT column1 || column2 FROM table_name;
```
JPQL提供了自己的函数，根据不同的方言可以转换为不同数据库的实现，目前来看mybatis的方言没有包括数据库函数的转换。

### 主键
不同的数据库自生成主键的实现方式不同，MySQL自增主键，Oracle，PG是序列。
Spring Data Jpa 通过在实体类的ID字段上配置`@GeneratedValue`可以做到不同数据库适配不同的主键，同时Hibernate提供了UUID的实现，也可以在这里配置。
mybatis依靠useGeneratedKeys="true"，可以在不同数据库生成不同类型的主键。
### 其他 
如果要做数据库兼容，使用mybatis的话，注意使用标准的SQL语法，尽量不要使用数据库自有的语法，加上数据库各自的方言也可以做到数据库兼容。 
 要考虑兼容不同数据库的话，解决这种不同数据库差异的问题,通常可以：
 - 数据库层面的配置。
 - 重写特定数据库方言中的排序方法。
 - 最后再考虑在SQL中添加自有语法，然后配置不同数据库调用不同的bean或者方法。
**因为现在mybatis也做了很多去兼容不同数据库的改进，所以实际上SpringData JPA跟mybatis差别不大**。
使用JPA编写持久层通常不使用原生SQL编写，更多的是使用方法名生成查询，JPQL，Criteria API来实现，这几种方式将编写的SQL限定在了标准SQL语法内，不会使用数据库特有的语法。对于需要特殊语法处理的操作，Spring Data JPA能够更早的发现。
 
# 切换数据库举例
Spring Data  JPA支持主流的关系型数据库MySQL，Mariadb,Oracle,SqlServer,PG,H2,DB2等。
比如，目前iSensePassword查询方法是基于PG的，只要引入相关的驱动进行相应的配置调整就可以切换到另外的数据库，以下为MariaDB
```yaml
spring:  
  datasource:  
    url: jdbc:mysql://127.0.0.1:3308/isensepassword  #数据库链接
    driver-class-name: com.mysql.cj.jdbc.Driver  
    username: root  
    password: my-secret-pw
  jpa:  
    database-platform: org.hibernate.dialect.MySQL8Dialect #数据库方言
```
SpringData JPA虽然不支持国产数据库，但是国产数据库通常都是基于某种开源的数据库或者存在某种数据库的兼容模式，通过对代码进行适量的调整，也可以让Spring Data JPA支持国产数据库。