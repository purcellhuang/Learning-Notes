# FastJson的简单使用

**mvnrepository： [https://mvnrepository.com/artifact/com.alibaba/fastjson](https://mvnrepository.com/artifact/com.alibaba/fastjson)**

## FastJosn特性
* 提供服务器端、安卓客户端两种解析工具，性能表现较好。
* 提供了 toJSONString() 和 parseObject() 方法来将 Java 对象与 JSON 相互转换。调用toJSONString方 法即可将对象转换成 JSON 字符串，parseObject 方法则反过来将 JSON 字符串转换成对象。
* 允许转换预先存在的无法修改的对象（只有class、无源代码）。
* Java泛型的广泛支持。
* 允许对象的自定义表示、允许自定义序列化类。
* 支持任意复杂对象（具有深厚的继承层次和广泛使用的泛型类型）。

## 简单使用
**创建Bean实体类对象**
```
public class Person {

    //serialize/deserialize 指定字段不序列化
    @JSONField(name = "name")    //// name 指定字段的名字
    private String name;
    @JSONField(name = "age",ordinal = 1)    // ordinal 指定字段的顺序
    private int age;
    @JSONField(name="dateOfBirth", format="dd/MM/yyyy", ordinal = 2)    //format配置日期格式化
    private Date dateOfBirth;


    public Person() {
    }
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Date getDateOfBirth() {
        return dateOfBirth;
    }

    public void setDateOfBirth(Date dateOfBirth) {
        this.dateOfBirth = dateOfBirth;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", dateOfBirth=" + dateOfBirth +
                '}';
    }
}

```
----
**测试案例**
```
        Person person = new Person("PurcellHuang",18);
        //Bean  -->  JsonString
        String jsonString = JSON.toJSONString(person);

        //JsonString  -->  Bean
        Person p1 = JSON.parseObject(jsonString,Person.class);

        //Bean  <-->  JsonObject
        JSONObject jsonObject = (JSONObject)JSON.toJSON(person);
        Person p2 = JSON.toJavaObject(jsonObject, Person.class);

        //JsonString  <-->  List
        List<Person> mList = new ArrayList<>();
        mList.add(person);
        String mlist = JSON.toJSONString(mList);
        List<Person> list =JSON.parseArray(mlist,Person.class);
```