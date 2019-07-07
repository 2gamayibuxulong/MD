JSON 服务器与客户端
===



发送数据
---

如果您的数据存储在 JavaScript 对象中，您可以把该对象转换为 JSON，然后将其发送到服务器。

```javascript
var myObj = { name:"Bill Gates",  age:62, city:"Seattle" };
var myJSON =  JSON.stringify(myObj);
```





接收数据
---

如果您以 JSON 格式接收到数据，您能够将其转换为 JavaScript 对象：

```javascript
var myJSON = '{ "name":"Bill Gates",  "age":62, "city":"Seattle" }';
var myObj =  JSON.parse(myJSON);
document.getElementById("demo").innerHTML = myObj.name;
```





存储数据
---

在存储数据时，数据必须是某种具体的格式，并且无论您选择在何处存储它，文本永远是合法格式之一。

JSON 让 JavaScript 对象存储为文本成为可能。

### 实例

把数据存储在本地存储中

```javascript
//存储数据：
myObj = { name:"Bill Gates",  age:62, city:"Seattle" };
myJSON =  JSON.stringify(myObj);
localStorage.setItem("testJSON", myJSON);

//接收数据：
text = localStorage.getItem("testJSON");
obj =  JSON.parse(text);
document.getElementById("demo").innerHTML = obj.name;
```