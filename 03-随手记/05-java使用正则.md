# java中使用正则表达式

**正则  regex **

```java
public static void main(String[] args) {
    String txt = "123ddddccfg";
    Pattern pattern = Pattern.compile("^(123)(fg)$");
    Matcher m = pattern.matcher(txt);

    if (m.find()) {
        System.out.println(m.group());
    }

}
```







