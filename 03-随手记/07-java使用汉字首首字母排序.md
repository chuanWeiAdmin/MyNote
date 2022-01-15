#### 1.不考虑多音字的情况

```java
Comparator<Object> com = Collator.getInstance(java.util.Locale.CHINA);
String[] newArray = {"江苏","海南","重庆"};
List<String> list = Arrays.asList(newArray);
Collections.sort(list, com);
for (String i : list) {
    System.out.print(i + "  ");

}
```



#### 2.考虑多音字的情况

1. 添加 pom 依赖

   ```xml
   <dependency>
     <groupId>com.belerweb</groupId>
     <artifactId>pinyin4j</artifactId>
     <version>2.5.1</version>
   </dependency>
   ```

2. 编写代码

   ```java
   String [] banks = {"江苏","海南","重庆"};
   Arrays.sort(banks, new Comparator<String>() {
   
       HanyuPinyinOutputFormat pinyinOutputFormat = new HanyuPinyinOutputFormat();
       @Override
       public int compare(String bank1, String bank2) {
           try {
   
               bank1 = PinyinHelper.toHanYuPinyinString(bank1, pinyinOutputFormat, " ", true);
               bank2 = PinyinHelper.toHanYuPinyinString(bank2, pinyinOutputFormat, " ", true);
   
           } catch (BadHanyuPinyinOutputFormatCombination badHanyuPinyinOutputFormatCombination) {
               badHanyuPinyinOutputFormatCombination.printStackTrace();
           }
           return bank1.compareTo(bank2);
       }
   });
   System.out.println(Arrays.toString(banks));
   ```

   