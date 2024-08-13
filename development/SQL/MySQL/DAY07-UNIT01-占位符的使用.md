# MyBatis的占位符

# 1 学习目标

1. **重点掌握**#{}占位符的使用

# 2 MyBatis的占位符介绍

-  **#{}占位符**: 相当于JDBC中的问号(?)占位符，是为SQL语句中的值进行占位，如果参数值是字符串或者日期类型，会进行转义处理

-  我们使用#{}写的SQL语句:

```xml
<update id="deleteStudent">
    DELETE FROM student WHERE id = #{id}
</update>
```

- 实际MyBatis执行的SQL

```sql
DELETE FROM student WHERE id = ?
```

- #{}占位符用来设置参数，参数的类型能够有3种：基本类型，自定义类型，Map
- 如果只传递单个参数,只需要使用#{参数名}即可将参数取出,其中参数名可以起任意值
- 如果传递多个参数,那么#{}的名字需要互不相同,所以一般情况我们都使用传入的字段名

# 3 XML方式使用#{}

## 3.1 入门案例

- 查询指定id的信息

①**`StudentMapper.xml`**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="tedu.mapper.StudentMapper">
    <select id="getStudentAll" resultType="tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
    </select>
    <insert id="insertStudent">
        INSERT INTO student
        values ('7777', '猪八戒', 3000, '男', '分家大队长', NULL, 0, 0, 0)
    </insert>
    <update id="updateStudent">
        UPDATE student
        SET job='净坛使者'
        WHERE name = '猪八戒'
    </update>
    <delete id="deleteStudent">
        DELETE
        FROM student
        WHERE name = '猪八戒'
    </delete>
    <select id="getStudentById" resultType="tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE id = #{id}
    </select>
</mapper>
```

②**`StudentMapper接口`**

```java
package tedu.mapper;

import tedu.pojo.Student;

import java.util.List;


public interface StudentMapper {
    public Student getStudentById(Long id);

    public List<Student> getStudentAll();

    public int insertStudent();

    public int updateStudent();

    public int deleteStudent();
}
```

③**`TestPlaceHolder`**

```java
package tedu;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tedu.mapper.StudentMapper;
import tedu.pojo.Student;

@SpringBootTest
class TestPlaceHolder {
    @Autowired
    private StudentMapper studentMapper;

    @Test
    public void testGetStudentById(){
        Student student = studentMapper.getStudentById(1L);
        System.out.println(student);
    }
}
```

## 3.2 基本类型

### 3.3.1 单个参数

- 如果只传入单个参数时,#{}占位符的参数名可以起任意名
- 查询指定name的信息

①**`StudentMapper.xml`**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.tedu.mapper.StudentMapper">
    <select id="getStudentAll" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
    </select>
    <insert id="insertStudent">
        INSERT INTO student
        values ('7777', '猪八戒', 3000, '男', '分家大队长', NULL, 0, 0, 0)
    </insert>
    <update id="updateStudent">
        UPDATE student
        SET job='净坛使者'
        WHERE name = '猪八戒'
    </update>
    <delete id="deleteStudent">
        DELETE
        FROM student
        WHERE name = '猪八戒'
    </delete>
    <select id="getStudentById" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE id = #{id}
    </select>
    <select id="getStudentByName" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE name = #{name}
    </select>
</mapper>
```

②**`StudentMapper接口`**

```java
package tedu.mapper;

import tedu.pojo.Student;

import java.util.List;


public interface StudentMapper {
    public Student getStudentByName(String name);

    public Student getStudentById(Long id);

    public List<Student> getStudentAll();

    public int insertStudent();

    public int updateStudent();

    public int deleteStudent();
}
```

③**`TestPlaceHolder`**

```java
package tedu;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tedu.mapper.StudentMapper;
import tedu.pojo.Student;

@SpringBootTest
class TestPlaceHolder {
    @Autowired
    private StudentMapper studentMapper;

    @Test
    public void testGetStudentById(){
        Student student = studentMapper.getStudentById(1L);
        System.out.println(student);
    }
    @Test
    public void testGetStudentByName(){
        Student student = studentMapper.getStudentByName("齐邵时");
        System.out.println(student);
    }
}
```

### 3.3.2 多个参数

- MyBatis如果传入多个参数,会对参数做特殊处理,会自动将多个参数封装成一个map,#{}会根据参数名作为key,去map中取对应的值,而MyBatis在封装时,会自动以对应的属性名来匹配对应的值

- 查询指定的出生日期范围内的记录

①**`StudentMapper.xml`**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.tedu.mapper.StudentMapper">
    <select id="getStudentAll" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
    </select>
    <insert id="insertStudent">
        INSERT INTO student
        values ('7777', '猪八戒', 3000, '男', '分家大队长', NULL, 0, 0, 0)
    </insert>
    <update id="updateStudent">
        UPDATE student
        SET job='净坛使者'
        WHERE name = '猪八戒'
    </update>
    <delete id="deleteStudent">
        DELETE
        FROM student
        WHERE name = '猪八戒'
    </delete>
    <select id="getStudentById" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE id = #{id}
    </select>
    <select id="getStudentByName" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE name = #{name}
    </select>
    <select id="getStudentByBirth" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE birth > #{param1}
          AND birth &lt; #{param2}
    </select>
</mapper>
```

②**`StudentMapper接口`**

```java
package tedu.mapper;

import tedu.pojo.Student;

import java.util.Date;
import java.util.List;


public interface StudentMapper {
    public List<Student> getStudentByBirth(String minBirth, String maxBirth);

    public Student getStudentByName(String name);

    public Student getStudentById(Long id);

    public List<Student> getStudentAll();

    public int insertStudent();

    public int updateStudent();

    public int deleteStudent();
}
```

③**`TestPlaceHolder`**

```java
package tedu;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tedu.mapper.StudentMapper;
import tedu.pojo.Student;

import java.util.List;

@SpringBootTest
class TestPlaceHolder {
    @Autowired
    private StudentMapper studentMapper;

    @Test
    public void testGetStudentById() {
        Student student = studentMapper.getStudentById(1L);
        System.out.println(student);
    }

    @Test
    public void testGetStudentByName() {
        Student student = studentMapper.getStudentByName("齐邵时");
        System.out.println(student);
    }

    @Test
    public void testGetStudentByBirth() {
        List<Student> students = studentMapper.getStudentByBirth("2015-5-1", "2015-5-31");
        for (Student student : students) {
            System.out.println(student);
        }
    }
}
```

## 3.4 自定义类型

- 如果在传入多个参数时,正好这些参数都是属于业务中数据模型,比如Jobs这个实体类的内容,那么可以直接把POJO作为参数即可,那此时使用#{}只需要用属性名即可取出参数

- 向student表中插入一条记录

①**`StudentMapper.xml`**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="tedu.mapper.StudentMapper">
    <select id="getStudentAll" resultType="tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
    </select>
    <insert id="insertStudent">
        INSERT INTO student
        values ('7777', '猪八戒', 3000, '男', '分家大队长', NULL, 0, 0, 0)
    </insert>
    <update id="updateStudent">
        UPDATE student
        SET job='净坛使者'
        WHERE name = '猪八戒'
    </update>
    <delete id="deleteStudent">
        DELETE
        FROM student
        WHERE name = '猪八戒'
    </delete>
    <select id="getStudentById" resultType="tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE id = #{id}
    </select>
    <select id="getStudentByName" resultType="tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE name = #{name}
    </select>
    <select id="getStudentByBirth" resultType="tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE birth > #{param1}
          AND birth &lt; #{param2}
    </select>
    <insert id="addStudent">
        INSERT INTO student
        values (#{id}, #{name}, #{age}, #{gender}, #{job}, #{birth}, #{locationId}, #{teamLeader}, #{classId})
    </insert>
</mapper>
```

②**`StudentMapper接口`**

```java
package tedu.mapper;

import tedu.pojo.Student;

import java.util.List;


public interface StudentMapper {
    public int addStudent(Student student);

    public List<Student> getStudentByBirth(String minBirth, String maxBirth);

    public Student getStudentByName(String name);

    public Student getStudentById(Long id);

    public List<Student> getStudentAll();

    public int insertStudent();

    public int updateStudent();

    public int deleteStudent();
}
```

③**`TestPlaceHolder`**

```java
package tedu;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tedu.mapper.StudentMapper;
import tedu.pojo.Student;

import java.util.List;

@SpringBootTest
class TestPlaceHolder {
    @Autowired
    private StudentMapper studentMapper;

    @Test
    public void testGetStudentById() {
        Student student = studentMapper.getStudentById(1L);
        System.out.println(student);
    }

    @Test
    public void testGetStudentByName() {
        Student student = studentMapper.getStudentByName("齐邵时");
        System.out.println(student);
    }

    @Test
    public void testGetStudentByBirth() {
        List<Student> students = studentMapper.getStudentByBirth("2015-5-1", "2015-5-31");
        for (Student student : students) {
            System.out.println(student);
        }
    }

    @Test
    public void testAddStudent() {
        Student student = new Student();
        student.setId(8888);
        student.setName("沙和尚");
        student.setAge(2800);
        student.setGender("男");
        student.setJob("搬家大队长");
        student.setBirth(null);
        student.setLocationId(0);
        student.setTeamLeader(0);
        student.setClassId(0);
        int rows = studentMapper.addStudent(student);
        System.out.println(rows > 0 ? "新增成功!" : "新增失败!");
    }
}
```

## 3.5 Map类型

- 如果在传入多个参数时,这些参数不属于业务模型中的数据,没有对应的实体类,那么此时为了方便,也可以将这些参数封装到一个Map中
- 但是这种方式只适合用于这个Map不经常使用,临时需要传入多个数据的情况

- 修改student表中的一条记录

①**`StudentMapper.xml`**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="tedu.mapper.StudentMapper">
    <select id="getStudentAll" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
    </select>
    <insert id="insertStudent">
        INSERT INTO student
        values ('7777', '猪八戒', 3000, '男', '分家大队长', NULL, 0, 0, 0)
    </insert>
    <update id="updateStudent">
        UPDATE student
        SET job='净坛使者'
        WHERE name = '猪八戒'
    </update>
    <delete id="deleteStudent">
        DELETE
        FROM student
        WHERE name = '猪八戒'
    </delete>
    <select id="getStudentById" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE id = #{id}
    </select>
    <select id="getStudentByName" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE name = #{name}
    </select>
    <select id="getStudentByBirth" resultType="cn.tedu.pojo.Student">
        SELECT id,
               name,
               age,
               gender,
               job,
               birth,
               location_id,
               team_leader,
               class_id
        FROM student
        WHERE birth > #{param1}
          AND birth &lt; #{param2}
    </select>
    <insert id="addStudent">
        INSERT INTO student
        values (#{id}, #{name}, #{age}, #{gender}, #{job}, #{birth}, #{locationId}, #{teamLeader}, #{classId})
    </insert>
    <update id="updateStudentById">
        UPDATE student SET name = #{name}, job = #{job} WHERE id = #{id}
    </update>
</mapper>
```

②**`StudentMapper接口`**

```java
package tedu.mapper;

import tedu.pojo.Student;

import java.util.List;
import java.util.Map;


public interface StudentMapper {
    public int addStudent(Student student);

    public List<Student> getStudentByBirth(String minBirth, String maxBirth);

    public Student getStudentByName(String name);

    public Student getStudentById(Long id);

    public List<Student> getStudentAll();

    public int insertStudent();

    public int updateStudent();

    public int deleteStudent();

    public int updateStudentById(Map<String, Object> map);
}
```

③**`TestPlaceHolder`**

```java
package tedu;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tedu.mapper.StudentMapper;
import tedu.pojo.Student;

import java.util.HashMap;
import java.util.List;

@SpringBootTest
class TestPlaceHolder {
    @Autowired
    private StudentMapper studentMapper;

    @Test
    public void testGetStudentById() {
        Student student = studentMapper.getStudentById(1L);
        System.out.println(student);
    }

    @Test
    public void testGetStudentByName() {
        Student student = studentMapper.getStudentByName("齐邵时");
        System.out.println(student);
    }

    @Test
    public void testGetStudentByBirth() {
        List<Student> students = studentMapper.getStudentByBirth("2015-5-1", "2015-5-31");
        for (Student student : students) {
            System.out.println(student);
        }
    }

    @Test
    public void testAddStudent() {
        Student student = new Student();
        student.setId(8888);
        student.setName("沙和尚");
        student.setAge(2800);
        student.setGender("男");
        student.setJob("搬家大队长");
        student.setBirth(null);
        student.setLocationId(0);
        student.setTeamLeader(0);
        student.setClassId(0);
        int rows = studentMapper.addStudent(student);
        System.out.println(rows > 0 ? "新增成功!" : "新增失败!");
    }
    @Test
    public void testUpdateStudentById() {
        HashMap<String, Object> map = new HashMap<>();
        map.put("id", "8888");
        map.put("name", "孙悟空");
        map.put("job", "除妖大队长");
        int rows = studentMapper.updateStudentById(map);
        System.out.println(rows > 0 ? "修改成功!!" : "修改失败!!");
    }
}
```

# 4 注解方式使用#{}

## 4.1 入门案例: 查询指定id的信息

①**`StudentMapper接口`**

```java
package tedu.mapper;

import org.apache.ibatis.annotations.Select;
import tedu.pojo.Student;

import java.util.List;
import java.util.Map;


public interface StudentMapper {
    public int addStudent(Student student);

    public List<Student> getStudentByBirth(String minBirth, String maxBirth);

    public Student getStudentByName(String name);

    public Student getStudentById(Long id);

    @Select("SELECT * FROM student WHERE id = #{id}")
    public Student getStudentById2(Long id);

    public List<Student> getStudentAll();

    public int insertStudent();

    public int updateStudent();

    public int deleteStudent();

    public int updateStudentById(Map<String, Object> map);

}
```

②**`TestPlaceHolder`**

```java
package tedu;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tedu.mapper.StudentMapper;
import tedu.pojo.Student;

import java.util.HashMap;
import java.util.List;

@SpringBootTest
class TestPlaceHolder {
    @Autowired
    private StudentMapper studentMapper;

    @Test
    public void testGetStudentById() {
        Student student = studentMapper.getStudentById(1L);
        System.out.println(student);
    }

    @Test
    public void testGetStudentByName() {
        Student student = studentMapper.getStudentByName("齐邵时");
        System.out.println(student);
    }

    @Test
    public void testGetStudentByBirth() {
        List<Student> students = studentMapper.getStudentByBirth("2015-5-1", "2015-5-31");
        for (Student student : students) {
            System.out.println(student);
        }
    }

    @Test
    public void testAddStudent() {
        Student student = new Student();
        student.setId(8888);
        student.setName("沙和尚");
        student.setAge(2800);
        student.setGender("男");
        student.setJob("搬家大队长");
        student.setBirth(null);
        student.setLocationId(0);
        student.setTeamLeader(0);
        student.setClassId(0);
        int rows = studentMapper.addStudent(student);
        System.out.println(rows > 0 ? "新增成功!" : "新增失败!");
    }
    @Test
    public void testUpdateStudentById() {
        HashMap<String, Object> map = new HashMap<>();
        map.put("id", "8888");
        map.put("name", "孙悟空");
        map.put("job", "除妖大队长");
        int rows = studentMapper.updateStudentById(map);
        System.out.println(rows > 0 ? "修改成功!!" : "修改失败!!");
    }
    @Test
    public void testGetStudentById02() {
        Student student = studentMapper.getStudentById2(8888L);
        System.out.println(student);
    }
}
```

# 5 日志配置

## 5.1 什么是日志

- 日志的作用是用来追踪和记录我们的程序运行中的信息，我们可以利用日志很快定位问题，追踪分析。
- 如果没有日志，程序一旦出现问题，很难一下子就能定位问题。尤其是访问第三方接口、随机或偶尔出现的问题、很难再现的问题。

## 5.2 日志级别

SpringBoot的日志框架提供了如下几种日志级别（从高到低）：

- ERROR：错误级别，指出发生错误事件，需要立即处理；

- WARN：警告级别，表明可能发生错误的情况，需要注意和防止；

- INFO：信息级别，描述应用程序的正常运行信息，但不会打印太多的信息；(SpringBoot默认级别)

- DEBUG：调试级别，用于调试应用程序，打印详细的调试信息；

- TRACE：跟踪级别，记录应用程序中每个步骤的详细信息。

## 5.3 设置日志级别

- 为了方便查看打印的SQL日志,所以可以为我们项目添加日志配置,只需要在application.yml中添加如下内容:

  **注意**: cn.tedu为我们项目的根包

```yaml
#日志设置
logging:
  level:
    cn:
      tedu: debug
```
