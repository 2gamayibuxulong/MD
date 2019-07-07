JSON 转换
===

主要格式

{"age":"22","name":"李四"}

[{"age":"21","name":"张三"}]


---

- ## String——>>>JSONArray

```java
String st = "[{name:Tim,age:25,sex:male},{name:Tom,age:28,sex:male},{name:Lily,age:15,sex:female}]";

JSONArray tableData = JSONArray.parseArray(st);
```



- ## JSONArray——>>>JSONObject

```java
JSONObject rowData = new JSONObject();
for(int i;i<tableData.length();i++){
    rowData = tableData.getJSONObject[i];
}
```



- ## String——>>>JSONObject

```java
String st = "{name:Tim,age:25,sex:male}";
JSONObject rowData = JSONObject.parseObject(st);
```



- ## JSONObject——>>>JSONArray

  ```java
  JSONObject rowData = {info:
                              [
                                  {
                                      name:Tim,
                                      age:25,
                                      sex:male
                                  },{
                                      name:Tom,
                                      age:28,
                                      sex:male
                                  },{
                                      name:Lily,
                                      age:15,
                                      sex:female
                                  }
                              ]
                          };
  JSONArry tableData = rowData.get("info");
  ```





## 对象 转 JSON

---

```java
    Student stu1 = new Student();
	stu1.setName("张三");
	stu1.setAge("21");
 
	String stu1Json = JSONObject.toJSONString(stu1);
```


## JSON 转 对象

```java
    Student stu1to = JSON.parseObject(stu1Json, Student.class);
	System.out.println("json 转对象：");
	System.out.println(stu1to);
	System.out.println(stu1to.getName());
	System.out.println(stu1to.getAge());
```




## 对象数组 转 JSON

```java
    Student stu2 = new Student();
	stu2.setName("李四");
	stu2.setAge("22");
	List<Student> list = new ArrayList<Student>();
	list.add(stu1);
	list.add(stu2);
 
	String listJson = JSONObject.toJSONString(list);
	System.out.println(listJson);
```




## JSON 转 对象数组

```java
    List<Student> studentList = JSON.parseArray(listJson, Student.class);
	for (Student student : studentList) {
		System.out.println(student.getName());
	}
```


## JSON多级组合，适用于请求文档传输参数

```java
    JSONObject jsona = new JSONObject();
	jsona.put("number", "1");
	JSONObject jsonb = new JSONObject();
	jsonb.put("listMap", list);
 
	JSONObject jsonAll = new JSONObject();
	jsonAll.put("jsona", jsona);
	jsonAll.put("jsonb", jsonb);
	String jsonAllStr =JSONObject.toJSONString(jsonAll);
	System.out.println(jsonAllStr);
```


## 多级 JSON 组合

```java
String getJsona = JSON.parseObject(jsonAllStr).getString("jsona");
String strjsona = JSON.parseObject(getJsona, String.class); //指定获取 字段名对象信息，如果为单个String可不指定，这里作为实例写出
System.out.println("只拿jsona信息");
System.out.println(strjsona);
```



```java
String getJsonb = JSON.parseObject(jsonAllStr).getString("jsonb");
String getJsonbb = JSON.parseObject(getJsonb).getString("listMap");    //这里被二级包裹，所以要获取2次才能转换对象数组
List<Student> strjsonb = JSON.parseArray(getJsonbb, Student.class);
System.out.println("只拿jsonbb信息");
System.out.println(strjsonb);
```