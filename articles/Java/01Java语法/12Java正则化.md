# 概念

正则表达式（`regex`）可以用来匹配、替换、分割字符串。

正则表达式由常用字符+特殊字符组成，称为模式Pattern。

示例1：典型用法

```
public class RegexTest {
    public static void main(String[] args) {
        String content = "Hello Word!";
        // 1. 创建一个表达式对象
        Pattern pattern = Pattern.compile("[a|e]+");
        // 2. 创建一个匹配对象，即，按照Pattern去匹配字符串。
        Matcher matcher = pattern.matcher(content);
        // 3. 开始循环匹配
        while (matcher.find()) {
            System.out.println("content has a or e: " + matcher.group(0));
        }
    }
}
```

示例2：字符串方法（实际用的是Pattern类的静态方法）

```
public class RegexTest {
    public static void main(String[] args) {
        // 匹配：手机号
        boolean matches = "18820203030".matches("\\d{11}");
        System.out.println(matches);
        // 分割：按空格分割
        String[] split = "Hello World".split("\\s");
        System.out.println(split.length);
        // 替换
        String s = "password:1882141".replaceAll("\\d", "*");
        System.out.println(s);
    }
}
```



# 正则表达式的语法

正则表达式支持的字符

| 字符   | 含义                                    |
| ------ | --------------------------------------- |
| X      | 任何合法的字符                          |
| \0mnn  | 八进制数 0mnn所表示的字符               |
| \xhh   | 十六进制数 0xhh所表示的字符             |
| \uhhhh | 十六进制值 0xhhhh 所表示的 Unicode 字符 |

特殊字符（转移字符）：

| 字符  | 含义                       |
| ----- | -------------------------- |
| .     | 任意一个字符，除换行符外   |
| ?     | 零次或一次                 |
| ^     | 开头 （在[]中表示非）      |
| $     | 结尾                       |
| +     | 一次或多次                 |
| *     | 零次或多次                 |
| {n}   | 正好n次                    |
| {n,}  | 至少n次                    |
| {n,m} | 至少n次，至多m次           |
| []    | 表示[]中的任意一个字符     |
| -     | 在[]中使用，表示字符范围   |
| \|    | 在[]中使用，表示或         |
| \     | 表示后面一个字符是转移字符 |
| \d    | 数字                       |
| \D    | 非数字                     |
| \w    | 单词                       |
| \W    | 非单词                     |
| \s    | 空格                       |
| \S    | 非空格                     |
| \f    | 换页符                     |
| \n    | 换行符                     |

示例

| 正则表达式      | 含义         |
| --------------- | ------------ |
| [\u4e00-\u9fa5] | 表示中文字符 |
| ^\d{m,n}$       | m-n位的数字  |





# 分组&Matcher类

Matcher 类提供了几个常用方法：

| 名称        | 说明                                                        |
| ----------- | ----------------------------------------------------------- |
| find()      | 返回目标字符串中是否包含与 Pattern 匹配的子串               |
| group()     | 返回上一次与 Pattern 匹配的子串                             |
| start()     | 返回上一次与 Pattern 匹配的子串在目标字符串中的开始位置     |
| end()       | 返回上一次与 Pattern 匹配的子串在目标字符串中的结束位置加 1 |
| lookingAt() | 返回目标字符串前面部分与 Pattern 是否匹配                   |
| matches()   | 返回整个目标字符串与 Pattern 是否匹配                       |
| reset()     | 将现有的 Matcher 对象应用于一个新的字符序列。               |



分组和Matcher类用法：

```
public class TestReg {
    public static void main(String[] args) {
        // 按指定模式在字符串查找
        String line = "phone:1889999";
        // () 表示分组，这里有两组
        // \\d+ 表示至少一个数字
        // ? 表示勉强模式，尽可能少的匹配
        String pattern = "(\\d+?)(\\d+?)";

        // 创建 Pattern 对象
        Pattern r = Pattern.compile(pattern);

        // 现在创建 matcher 对象
        Matcher m = r.matcher(line);
        while (m.find( )) {
            System.out.println("Found value: " + m.group(0));
            System.out.println("Found value: " + m.group(1));
            System.out.println("Found value: " + m.group(2));
        }
    }
}

// 输出
// m.group(0)表示整个匹配到的字符串 m.group(i)表示第几个分组(i>0)
Found value: 18
Found value: 1
Found value: 8
Found value: 89
Found value: 8
Found value: 9
Found value: 99
Found value: 9
Found value: 9
```



分组的反向引用

```
String s = "the quick brown fox jumps over the lazy dog.";
// $1表示分组1的内容
String r = s.replaceAll("\\s([a-z]{4})\\s", " <b>$1</b> ");
System.out.println(r); 
// 输出
// the quick brown fox jumps <b>over</b> the <b>lazy</b> dog.
```



# 匹配模式

当使用数量表示符`?+*`时，正则表达式有以下三种模式：

Greedy（贪婪模式）：默认。一直匹配下去，直到无法匹配为止。

Reluctant（勉强模式）：只会匹配最少的字符。用?作为后缀。

Possessive（占有模式）：目标只有Java支持。用+作为后缀。

举例：

| 贪婪模式 | 勉强模式 | 占用模式 | 含义         |
| -------- | -------- | -------- | ------------ |
| X?       | X??      | X?+      | 零次或一次   |
| X*       | X*?      | X*+      | 零次或多次   |
| X+       | X+?      | X++      | 一次或多次   |
| X{n,m}   | X{n,m}?  | X{n,m}+  | 最少n，最多m |



示例：

```
System.out.println("hello,java!".replaceFirst("\\w*", "x")); # x,java!
System.out.println("hello,java!".replaceFirst("\\w*?", "x")); # xhello,java!

因为下面是勉强模式，能按0次匹配上。
```



尽量用勉强模式，贪婪模式容易内存溢出！



# ?:和?!



`(?:pattern)`

非获取匹配，匹配pattern但不获取匹配结果，不进行存储供以后使用。这在使用或字符“(|)”来组合一个模式的各个部分是很有用。例如`“industr(?:y|ies)”`就是一个比`“industry|industries”`更简略的表达式。

`(?=pattern)`

非获取匹配，正向肯定预查，在任何匹配pattern的字符串开始处匹配查找字符串，该匹配不需要获取供以后使用。例如，`“Windows(?=95|98|NT|2000)”`能匹配`“Windows2000”`中的`“Windows”`，但不能匹配`“Windows3.1”`中的`“Windows”`。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始。

`(?!pattern)`

非获取匹配，正向否定预查，在任何不匹配pattern的字符串开始处匹配查找字符串，该匹配不需要获取供以后使用。例如`“Windows(?!95|98|NT|2000)”`能匹配`“Windows3.1”`中的`“Windows”`，但不能匹配`“Windows2000”`中的`“Windows”`。

`(?<=pattern)`

非获取匹配，反向肯定预查，与正向肯定预查类似，只是方向相反。例如，`“(?<=95|98|NT|2000)Windows”`能匹配`“2000Windows”`中的`“Windows”`，但不能匹配`“3.1Windows”`中的`“Windows”`。

`(?<!pattern)`

非获取匹配，反向否定预查，与正向否定预查类似，只是方向相反。例如`“(?<!95|98|NT|2000)Windows”`能匹配`“3.1Windows”`中的`“Windows”`，但不能匹配`“2000Windows”`中的`“Windows”`。



JDK文档中的描述

| Special constructs (non-capturing) |                                           |
| ---------------------------------- | ----------------------------------------- |
| (?:X)                              | X, as a non-capturing group               |
| (?=X)                              | X, via zero-width positive look ahead     |
| (?!X)                              | X, via zero-width negative look ahead     |
| (?<=X)                             | X, via zero-width positive look behind    |
| (?<!X)                             | X, via zero-width negative look behind    |
| (?>X)                              | X, as an independent, non-capturing group |



non-capturing group非捕获组：因为`()`表示组，但是在这里只匹配不存至组。

zero-with零宽度：断言前后不会改变匹配引擎的指针位置。

positive & negative：匹配 & 不匹配

look ahead & look behind 前瞻和后顾：匹配引擎的前后，和常识中的文本前后正好相反，因为匹配指针一直往前走的。

示例1：`(?:X)`

```
(?:b)b不可以匹配bc
(?:b)c可以匹配bc
 * 因为(?:X)不可以回溯，不是零宽度
```

示例2：`(?=X)`

```
a(?=b)c不能匹配abc，也不能匹配ac,ab，不能匹配任何字符串。
 * 因为a后面跟的是c，但又要预测a之后是b。
(?=b)b可以匹配b
(?=b)bc可以匹配bc
 * 因为(?=b)前面没有字符，表示从任意位置都可以开始匹配。所以(?=X)一般放在后面
```

示例3：`(?<=X)`

```
(?<=b)c可以匹配bc，匹配结果是c
```

示例4：`(?>X)`

```
(?>b)b不能匹配bc
(?>b)c可以匹配bc
 * (?>X)预测其后是X，但是不进行字符位置回溯。有点类似于(?:X)
```

