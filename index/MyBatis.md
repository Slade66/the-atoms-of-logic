
### @Param 

**作用：** 给 SQL 插入的变量“起名字”。

比如有个方法 `int countByUsername(@Param("username") String username);`
MyBatis 需要把这个参数传到 SQL 里，比如：
```xml
<select id="countByUsername" resultType="int">
    SELECT COUNT(*) FROM user WHERE username = #{username}
</select>
```
这里的 `#{username}` 就是用的你 @Param("username") 定义的名字。

如果没有@Param，MyBatis 默认用：arg0 / param1，你在 SQL 里就只能用这两个东西插变量，可读性极差。

### id 

对应 Mapper 接口方法名

必须一一对应，不然报错。

### insert

MyBatis 执行 insert 时，其实做了 3 件事：
1️⃣ 把 Java 对象 → SQL 参数
2️⃣ 执行 INSERT
3️⃣ （可选）把数据库生成的 ID 回填到对象

### resultType

SQL 返回结果的类型

### resultMap

在别处定义一个 `<resultMap id="UserResultMap" type="User">`，把 SQL 的返回字段映射到这个类型对象上，解决：字段名 ≠ 属性名 的问题。

### parameterType

参数的类型。

用于在 SQL 语句中直接插入参数类型的字段。

### useGeneratedKeys

开启“获取自增主键”

获取数据库生成的主键。

### keyProperty

插入后，把数据库生成的主键回填到 Java 对象哪个字段

### keyColumn

数据库表中的列名

当数据库主键列名和对象字段名不一致时使用。


