[TOC]



# 前言

MyBatis-Plus 是 MyBatis 为简化开发而提供的强大增强工具包。该工具包为 MyBatis 提供了一些高效、实用、开箱即用的功能，使用它可以有效节省您的开发时间。简单来说 MyBatis-Plus 提供了比 MyBatis 更方便的对数据库的增删改查，代码更简洁，完全兼容 MyBatis 



关于 MyBatis-Plus 你可以参考：

[MyBatis-Plus官网](https://baomidou.com/introduce/)

[MyBatis-Plus的 github 地址](https://github.com/baomidou/mybatis-plus)

# 一、依赖
对于 Spring Boot2
```java
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.7</version>
</dependency>
```

对于 Spring Boot3
```java
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.7</version>
</dependency>
```

# 二、配置
配置文件里配置
```java
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  configuration:
    #解决Mybatis 查询返回类型为Map 空值字段不显示
    call-setters-on-nulls: true

```

# 三、注解

| 注解        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| @TableName  | 用于当数据库表名和实体类名不一致的时候注解在类名上，以明确实体类对应的表名 |
| @TableId    | 用于标记实体类中对应的数据库表中的主键字段                   |
| @TableField | 用于标记实体类中的字段对应的数据库表中的名称                 |

关于注解详细信息可参考官网 [MyBatis-Plus注解](https://baomidou.com/reference/annotation/)


# 四、数据库建立 teacher 表测试
```java
CREATE TABLE teacher (
    id INT AUTO_INCREMENT PRIMARY KEY, -- 教师ID
    name VARCHAR(100) NOT NULL,        -- 姓名
    gender ENUM('男', '女') NOT NULL,  -- 性别
    birth_date DATE NOT NULL,          -- 出生日期
    age INT,                           -- 年龄
    school_name VARCHAR(100) NOT NULL, -- 学校
    class VARCHAR(50),            -- 班级
    course VARCHAR(100)           -- 课程
);

-- 插入测试数据
INSERT INTO teacher (name, gender, birth_date, age, school_name, class, course) VALUES 
('张三', '男', '1980-05-12', 44, '北京第一中学', '初三1班', '数学'),
('李四', '女', '1985-08-20', 39, '上海第二中学', '高二3班', '英语'),
('王五', '男', '1990-11-15', 33, '广州第三中学', '初二4班', '物理'),
('赵六', '女', '1975-09-30', 48, '深圳第四中学', '高三5班', '化学'),
('孙七', '男', '1983-02-25', 41, '杭州第五中学', '初一2班', '历史');
```



# 五、实体类

```java
public class Teacher {
    @TableId(value = "id", type = IdType.AUTO)
    Integer id;            // 教师ID
    String name;          // 姓名
    String gender;        // 性别
    LocalDate birthDate;  // 出生日期
    Integer age;          // 年龄
    String schoolName;   // 学校
    @TableField("class")
    String className;    // 班级
    String course;      //课程
 
    //省略get、set方法
}
```

> 说明：
>
> 上面的实体类 Teacher 对应了 teacher 表，及表中的字段，因为 mybatisplus 会将表名和表字段里的下划线去除转为驼峰命名，所以上面的 Teacher 类上可以省略 @TableName 注解，birthDate 对应了表中的 birth_date 字段所以可以省略 @TableField 字段，className 使用了 @TableField("class") 表名其对应表中的 class 字段。

虽然上面的实体类没问题，但建议 在实体类上使用 @TableName 来明确对应的表名，在类变量上使用 @TableField 来明确对应的表字段名，所以上面的实体类调整为：

```java
@TableName(value = "teacher")
public class Teacher {
    @TableId(value = "id", type = IdType.AUTO)
    Integer id;            // 教师ID
    @TableField("name")
    String name;          // 姓名
    @TableField("gender")
    String gender;        // 性别
    @TableField("birth_date")
    LocalDate birthDate;  // 出生日期
    @TableField("age")
    Integer age;          // 年龄
    @TableField("school_name")
    String schoolName;   // 学校
    @TableField("class")
    String className;    // 班级
    @TableField("course")
    String course;      //课程
    
    //省略get、set方法
}    
```





#  六、Mapper类

新建 Mapper 接口继承 BaseMapper

```java
@Mapper
@Repository
public interface MybatisPlusTestMapper extends BaseMapper<Teacher> {

}
```


#  七、使用

## select查询

查询表中的所有记录
```java
        LambdaQueryWrapper<Teacher> wrapper = new LambdaQueryWrapper<>();
        List<Teacher> teachers = mybatisPlusTestMapper.selectList(wrapper);
```

根据条件查询指定字段
```java
 LambdaQueryWrapper<Teacher> wrapper = new LambdaQueryWrapper<>();
 //这里只查询 id、姓名、年龄，因为没查询其他字段所以其他字段为null
        wrapper.select(Teacher::getId, Teacher::getSchoolName,Teacher::getAge)
                .lt(Teacher::getAge, 49 )
                .ge(Teacher::getAge, 37)
                .eq(Teacher::getGender, "女");
        List<Teacher> teachers = mybatisPlusTestMapper.selectList(wrapper);
```
根据条件查询全部字段
```java
 LambdaQueryWrapper<Teacher> wrapper = new LambdaQueryWrapper<>();
        wrapper.lt(Teacher::getAge, 49 )
                .ge(Teacher::getAge, 37)
                .eq(Teacher::getGender, "女");
        List<Teacher> teachers = mybatisPlusTestMapper.selectList(wrapper);
```

## insert插入

```java
		//因为id是自增的所以可以不需要set
        Teacher teacher = new Teacher();
        teacher.setName("刘梅");
        teacher.setGender("女");
        teacher.setBirthDate(LocalDate.of(1996, 10, 21));
        teacher.setAge(25);
        teacher.setSchoolName("济南一中");
        teacher.setClassName("1班");
        teacher.setCourse("数学");

        int rows = mybatisPlusTestMapper.insert(teacher); 
        if (rows > 0) {
            System.out.println("插入成功.");
        } else {
            System.out.println("插入失败.");
        }

```
## delete删除
```java
        LambdaQueryWrapper<Teacher> lambdaQueryWrapper = new LambdaQueryWrapper<>();
        lambdaQueryWrapper.eq(Teacher::getAge, 25);
        int delete = mybatisPlusTestMapper.delete(lambdaQueryWrapper);
        if (delete != 1) {
            System.out.println("未找到要删除的记录.");;
        }else {
            System.out.println("已删除"+delete+"条记录");;
        }

```
## update更新
查询年龄大于40并且性别是女的记录，将班级修改为"2班"，课程修改为"体育"
```java
        LambdaUpdateWrapper<Teacher> updateWrapper = new LambdaUpdateWrapper<>();
        updateWrapper.gt(Teacher::getAge, 40).eq(Teacher::getGender, "女");
        Teacher updateUser = new Teacher();
        updateUser.setClassName("2班");
        updateUser.setCourse("体育");
        int rows = mybatisPlusTestMapper.update(updateUser, updateWrapper); 
        if (rows > 0) {
            System.out.println("更新成功.");
        } else {
            System.out.println("没有找到要更新的数据.");
        }
```

## 其他
MyBatis-Plus 还提供了批量插入、流失查询、分页查询等等操作，可参考官网 [持久层接口](https://baomidou.com/guides/data-interface/) 。

另外 MyBatis-Plus 完全兼容 MyBatis ，因此你可以在 Mapper 类里增加方法，然后在 xml 里写你自己的 sql 来执行。

---

参考：
[MyBatisPlus - 润物无声、效率至上、丰富功能](https://blog.csdn.net/CYK_byte/article/details/136114368)