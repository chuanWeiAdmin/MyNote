#### 1.动态添加对象绑定事件

动态添加的元素通过一下方式不行

因为动态生成的元素，是不能够以普通绑定事件的形式来进行操作的

```javascript
$("input[name=xz]").click(function () {

	alert(123);

})
```

动态生成的元素，要以on的方法来触发事件

**语法**：**$(*需要绑定元素的有效的外层元素*).on(*绑定事件的方式*,需要绑定的元素的jquery对象,回调函数)**

```JavaScript
$("#activityBody").on("click",$("input[name=xz]"),function () {    
  ("#qx").prop("checked",$("input[name=xz]").length==$("input[name=xz]:checked").length);
})
```



#### 6.文本域textarea

1. 一定是要以标签对的形式来呈现,正常状态下标签对要紧紧的挨着，**否则中间的都是文本域的内容**
2. textarea虽然是以标签对的形式来呈现的，但是它也是属于表单元素范畴我们所有的对于textarea的取值和赋值操作，**应该统一使用   *val()*   方法**（而不是html()方法）

```html
<textarea class="form-control" rows="3" id="edit-description">123</textarea>
```



#### 7.js中json

```javascript
//在js中json的key要是变量要使用 json[target]来表示
var json ={};
//如果key是变量
json.target//获取不到
json[target] //target为变量的正确获取方式
```

