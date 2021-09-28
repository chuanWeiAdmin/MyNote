```java
public static String creatUUID() {
    //生成的是36位的UUID
    //通过replace("-", ""); 变成32位的UUID
	return UUID.randomUUID().toString().replace("-", "");
}
```

