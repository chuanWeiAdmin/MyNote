#### html

**可以直接使用表单提交到对应url,或者使用ajax提交**

<!--表单提交记得申明enctype为multipart/form-data 否则后面会报错-->

```html
<!--表单提交记得申明enctype为multipart/form-data 否则后面会报错-->
<form id="myform" action="/file/uploadFile" enctype="multipart/form-data" method="post">
    <input type="text" id="username" name="username"/><br/>
    <input type="file" name="file1" id="file_id"/><br/>
    <input type="button" id="test_button" style="height: 50px; width: 100px" value="测试按钮">
</form>
```



#### js代码

```javascript
<script>
    
    $(function () {
        $("#test_button").click(function () {
            //获取表单中的内容
            var file1 = $("#file_id")[0].files[0];
            var userName = $("#username").val();

            var formData = new FormData();
            formData.append("file1", file1);
            formData.append("userName", userName);
            //'/some.do',

            $.ajax({
                url:'/some.do',
                type: "POST",
                contentType: false,
                datatype: "text",
                data: formData,
                processData: false, // 告诉jQuery不要去处理发送的数据
                success: function (data) {
                    alert(data);
                }
            });

        });
    })
</script>
```



#### 后端代码

```java
@PostMapping("some.do")
@ResponseBody
public String  doSome(HttpServletRequest request, HttpServletResponse response) throws IOException {
    String result = "成功";

    MultipartHttpServletRequest multipartHttpServletRequest = (MultipartHttpServletRequest) request;
    List<MultipartFile> files = multipartHttpServletRequest.getFiles("file1");
    System.out.println("文件:" + files);
    System.out.println(files.size());

    String name = request.getParameter("userName");
    System.out.println(name);

    //将MultipartFile保存
    for (MultipartFile file : files) {
        String filePath = "C:\\Users\\0\\Desktop\\新建文件夹\\" + file.getOriginalFilename();
        File desFile = new File(filePath);
        if (!desFile.getParentFile().exists()) {
            desFile.mkdirs();
        }
        
        try {
            file.transferTo(desFile);
        } catch (IllegalStateException | IOException e) {
            e.printStackTrace();
            result = "失败";
        }
    }

    return result;
}
```

