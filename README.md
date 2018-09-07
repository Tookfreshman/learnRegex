# Learn Regex
原作者：老姚  
链接：https://juejin.im/post/5965943ff265da6c30653879  
来源：掘金  

## 目录
- [第一章 正则表达式字符匹配](#chapter1)
- [第二章 正则表达式位置匹配](#chapter2)
- [第三章 正则表达式括号的作用](#chapter3)
- [第四章 正则表达式回溯法原理](#chapter4)
- [第五章 正则表达式的拆分](#chapter5)
- [第六章 正则表达式的构建](#chapter6)
- [第七章 正则表达式编程](#chapter7)


## 引言
- 正则表达式是一种匹配模式，它要么匹配字符，要么匹配位置
- 在正则中可以使用括号补货数据，要么在api中进行分组引用，要么在正则里进行反向引用

## <span id="chapter1">第一章 正则表达式字符匹配</span>

### 1.精确匹配和模糊匹配
#### 1.1 精确匹配
```
    const reg = /hello/;
    console.log(reg.test("hello"));
    //=> true
```
#### 1.2 两种模糊匹配
##### 1.2.1 横向模糊匹配
如
```
    const reg = /[\d]{1,3}/;
    //匹配1-3位数字
```
##### 1.2.2 横向模糊匹配
如
```
    const reg = /a[123]b/;
    //匹配a1b、a2b、a3b
```

### 2.字符组
虽然叫字符组，但[abc]其实只表示abc三个字母中的一个字符。

#### 2.1 范围表示法
[12345abcEFG] 可以表示为 [1-5a-cE-F]  
如所有字母可表示为 [a-zA-Z]

#### 2.2 排除字符组
[^abc]表示某个位置的字符可以是任何字符，但是就是不能是"a"、"b"、"c"中的任何一个，^（脱字符），表示求反的概念。

#### 2.3常见的简写形式
\d就是[0-9]。表示是一位数字（digit）。  
\D就是[^0-9]。表示除数字以外的任何字符。  
\w[0-9a-zA-Z_]。表示数字、大小写字母和下划线（word）。也称单词字符。  
\W[^0-9a-zA-Z_]。非单词字符。  
\s[ \t\v\n\r\f]。表示空白符，包括空格、水平制表符、垂直制表符、换行符、回车符、换页符（space）。
\S[^ \t\v\n\r\f]。非空白符。
.[^\n\r\u2028\u2029]。通配符，表示几乎任意字符。换行符、回车符、行分隔符和段分隔符除外。

### 3.量词
#### 3.1 简写形式
{m,} 表示至少出现m次。  
{m} 等价于{m,m}，表示出现m次。  
? 等价于{0,1}，表示出现或者不出现。  
\+ 等价于{1,}，表示出现至少一次。记忆方式：加号是追加的意思，得先有一个，然后才考虑追加。  
\* 等价于{0,}，表示出现任意次，有可能不出现。

#### 3.2 贪婪匹配和惰性匹配
看如下例子：
```
    const regex = /\d{2,5}/g;
    const string = "123 1234 12345 123456";
    console.log( string.match(regex) );
    // => ["123", "1234", "12345", "12345"]
```
其中正则/\d{2,5}/，表示数字连续出现2到5次。会匹配2位、3位、4位、5位连续数字。  
但是其是贪婪的，它会尽可能多的匹配。你能给我6个，我就要5个。你能给我3个，我就3要个。反正只要在能力范围内，越多越好。  

我们知道有时贪婪不是一件好事。而惰性匹配，就是尽可能少的匹配：
```
    const regex = /\d{2,5}?/g;
    const string = "123 1234 12345 123456";
    console.log( string.match(regex) );
    // => ["12", "12", "34", "12", "34", "12", "34", "56"]
```
其中/\d{2,5}?/表示，虽然2到5次都行，当2个就够的时候，就不在往下尝试了。

通过在量词后面加个问号就能实现惰性匹配，因此所有惰性匹配情形如下：
1. {m,n}?
2. {m,}?
3. ??
4. +?
5. \*?

### 4. 多选分支
一个模式可以实现横向和纵向模糊匹配。而多选分支可以支持多个子模式任选其一。  
具体形式如下：(p1|p2|p3)，其中p1、p2和p3是子模式，用|（管道符）分隔，表示其中任何之一。  
例如要匹配"good"和"nice"可以使用/good|nice/。测试如下：  
```
    const regex = /good|nice/g;
    const string = "good idea, nice try.";
    console.log( string.match(regex) );
    // => ["good", "nice"]
```
但有个事实我们应该注意，比如我用/good|goodbye/，去匹配"goodbye"字符串时，结果是"good"：  
```
    const regex = /good|goodbye/g;
    const string = "goodbye";
    console.log( string.match(regex) );
    // => ["good"]
```
而把正则改成/goodbye|good/，结果是：  
```
    const regex = /goodbye|good/g;
    const string = "goodbye";
    console.log( string.match(regex) );
    // => ["goodbye"]
```
也就是说，分支结构也是惰性的，即当前面的匹配上了，后面的就不再尝试了。  

### 5. 例子
#### 5.1 匹配16进制颜色
示例：
\#ffbbad
\#FFF  
正则：
```
    const reg = /#([0-9a-fA-F]{6}|[0-9a-fA-F]{3})/;
```

#### 5.2 匹配时间
示例：  23:59  02:07   
```
    const reg = /^([01][0-9]|[2][0-3]):[0-5][0-9]$/;
```

#### 5.3 匹配日期
示例： 2017-08-09  
```
    const regex = /^[0-9]{4}-(0[1-9]|1[0-2])-(0[1-9]|[12][0-9]|3[01])$/;
```

## <span id="chapter2">第二章 正则表达式位置匹配</span>

### 1. 什么是位置
位置是相邻字符之间的位置。比如，下图中箭头所指的地方：  
![位置](https://user-gold-cdn.xitu.io/2017/7/19/95d0faf6b21f9414d24c8281b3046746?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 "位置")

### 2. 如何匹配位置
在es5 中共有6个锚字符：
^ $ \b \B (?=p) (?!p)  
#### 2.1 ^和$
^（脱字符）匹配开头，在多行匹配中匹配行开头。  
$（美元符号）匹配结尾，在多行匹配中匹配行结尾。  
多行匹配模式时，二者是行的概念，这个需要我们的注意。
#### 2.2 \b和\B
\b是单词边界，具体就是\w和\W之间的位置，也包括\w和^之间的位置，也包括\w和$之间的位置。  
比如一个文件名是"[JS] Lesson_01.mp4"中的\b，如下：
```
    const result = "[JS] Lesson_01.mp4".replace(/\b/g, '#');
    console.log(result);
    // => "[#JS#] #Lesson_01#.#mp4#"
```
为什么是这样呢？这需要仔细看看。  
首先，我们知道，\w是字符组[0-9a-zA-Z_]的简写形式，即\w是字母数字或者下划线的中任何一个字符。而\W是排除字符组[^0-9a-zA-Z_]的简写形式，即\W是\w以外的任何一个字符。  
此时我们可以看看"[#JS#] #Lesson_01#.#mp4#"中的每一个"#"，是怎么来的。  
* 第一个"#"，两边是"["与"J"，是\W和\w之间的位置。
* 第二个"#"，两边是"S"与"]"，也就是\w和\W之间的位置。
* 第三个"#"，两边是空格与"L"，也就是\W和\w之间的位置。
* 第四个"#"，两边是"1"与"."，也就是\w和\W之间的位置。
* 第五个"#"，两边是"."与"m"，也就是\W和\w之间的位置。
* 第六个"#"，其对应的位置是结尾，但其前面的字符"4"是\w，即\w和$之间的位置。

知道了\b的概念后，那么\B也就相对好理解了。  
\B就是\b的反面的意思，非单词边界。例如在字符串中所有位置中，扣掉\b，剩下的都是\B的。  
具体说来就是\w与\w、\W与\W、^与\W，\W与$之间的位置。  
比如上面的例子，把所有\B替换成"#"：
```
    const result = "[JS] Lesson_01.mp4".replace(/\B/g, '#');
    console.log(result);
    // => "#[J#S]# L#e#s#s#o#n#_#0#1.m#p#4"
```
#### 2.3 (?=p)和(?!p)
(?=p)，其中p是一个子模式，即p前面的位置。  
比如(?=l)，表示'l'字符前面的位置，例如：  
```
    const result = "hello".replace(/(?=l)/g, '#');
    console.log(result);
    // => "he#l#lo"
```
而(?!p)就是(?=p)的反面意思，比如：  
```
    const result = "hello".replace(/(?!l)/g, '#');
    console.log(result);
    // => "#h#ell#o#"
```
二者的学名分别是positive lookahead和negative lookahead。  
中文翻译分别是正向先行断言和负向先行断言。  
ES6中，还支持positive lookbehind和negative lookbehind。  
具体是(?<=p)和(?<!p)。  
也有书上把这四个东西，翻译成环视，即看看右边或看看左边。  
但一般书上，没有很好强调这四者是个位置。  
比如(?=p)，一般都理解成：要求接下来的字符与p匹配，但不能包括p的那些字符。  
而简单来说(?=p)就与^一样好理解，就是p前面的那个位置。

### 3. 位置的特性
对于位置的理解，我们可以理解成空字符''。  
比如"hello"字符串等价于如下的形式：  
`"hello" == "" + "h" + "" + "e" + "" + "l" + "" + "l" + "o" + "";`  
也等价于：  
`"hello" == "" + "" + "hello";`  
因此，把/^hello$/写成/^^hello$$$/，是没有任何问题的：  
```
    const result = /^^hello$$$/.test("hello");
    console.log(result);
    // => true
```
甚至可以写成更复杂的:  
```
    const result = /(?=he)^^he(?=\w)llo$\b\b$/.test("hello");
    console.log(result);
    // => true
```
也就是说字符之间的位置，可以写成多个。  
把位置理解空字符，是对位置非常有效的理解方式。  

### 4. 相关案例
#### 4.1 不匹配任何东西的正则
/.^/

#### 4.2 数字的千位分隔符表示法
##### 4.2.1 弄出最后一个逗号
使用(?=\d{3}$)就可以做到：  
```
    const result = "12345678".replace(/(?=\d{3}$)/g, ',')
    console.log(result);
    // => "12345,678"
```
##### 4.2.2 弄出所有的逗号  
因为逗号出现的位置，要求后面3个数字一组，也就是\d{3}至少出现一次。  
此时可以使用量词+：  
```
    const result = "12345678".replace(/(?=(\d{3})+$)/g, ',')
    console.log(result);
    // => "12,345,678"
```
##### 4.2.3 匹配其余案例
```
    const result = "123456789".replace(/(?=(\d{3})+$)/g, ',')
    console.log(result);
    // => ",123,456,789"
```
<span id="aa"></span>
因为上面的正则，仅仅表示把从结尾向前数，一但是3的倍数，就把其前面的位置替换成逗号。因此才会出现这个问题。  
怎么解决呢？我们要求匹配的到这个位置不能是开头。  
我们知道匹配开头可以使用^，但要求这个位置不是开头怎么办？  
easy，(?!^)，你想到了吗？测试如下：  
```
    const string1 = "12345678",
    string2 = "123456789";
    reg = /(?!^)(?=(\d{3})+$)/g;

    const result = string1.replace(reg, ',')
    console.log(result);
    // => "12,345,678"

    result = string2.replace(reg, ',');
    console.log(result);
    // => "123,456,789"
```
##### 4.2.4 支持其他形式
如果要把"12345678 123456789"替换成"12,345,678 123,456,789"。  
此时我们需要修改正则，把里面的开头^和结尾$，替换成\b：  
```
    const string = "12345678 123456789",
    reg = /(?!\b)(?=(\d{3})+\b)/g;

    const result = string.replace(reg, ',')
    console.log(result);
    // => "12,345,678 123,456,789"

```
其中(?!\b)怎么理解呢？   
要求当前是一个位置，但不是\b前面的位置，其实(?!\b)说的就是\B。   
因此最终正则变成了：/\B(?=(\d{3})+\b)/g。  

#### 4.3 验证密码问题
密码长度6-12位，由数字、小写字符和大写字母组成，但必须至少包括2种字符。   

##### 4.3.1 简化
不考虑“但必须至少包括2种字符”这一条件。我们可以容易写出：   
`const reg = /^[0-9a-zA-Z]{6,12}$/;`

##### 4.3.2 判断是否包含有某一种字符
假设，要求的必须包含数字，怎么办？此时我们可以使用(?=.\*[0-9])来做。  
因此正则变成：   
`const reg = /(?=.*[0-9])^[0-9A-Za-z]{6,12}$/;`

##### 4.3.3 同时包含具体两种字符
比如同时包含数字和小写字母，可以用(?=.\*[0-9])(?=.\*[a-z])来做。  
因此正则变成：   
`const reg = /(?=.*[0-9])(?=.*[a-z])^[0-9A-Za-z]{6,12}$/;`

##### 4.3.4 解惑
上面的正则看起来比较复杂，只要理解了第二步，其余就全部理解了。  
`/(?=.*[0-9])^[0-9A-Za-z]{6,12}$/`
对于这个正则，我们只需要弄明白(?=.\*[0-9])^即可。  
分开来看就是(?=.\*[0-9])和^。  
(?=.\*[0-9])表示该位置后面的字符匹配.\*[0-9]，即，有任何多个任意字符，后面再跟个数字。  
翻译成大白话，就是接下来的字符，必须包含个数字。

##### 4.3.5 另外一种解法
“至少包含两种字符”的意思就是说，不能全部都是数字，也不能全部都是小写字母，也不能全部都是大写字母。  
那么要求“不能全部都是数字”，怎么做呢？(?!p)出马！  
对应的正则是：  
`const reg = /(?!^[0-9]{6,12}$)^[0-9A-Za-z]{6,12}$/;`
三种“都不能”呢？  
最终答案是：  
`const reg = /(?!^[0-9]{6,12}$)(?!^[a-z]{6,12}$)(?!^[A-Z]{6,12}$)^[0-9A-Za-z]{6,12}$/;`

## <span id="chapter3">第三章 正则表达式括号的作用</span>
不管哪门语言中都有括号。正则表达式也是一门语言，而括号的存在使这门语言更为强大。  
对括号的使用是否得心应手，是衡量对正则的掌握水平的一个侧面标准。  
括号的作用，其实三言两语就能说明白，括号提供了分组，便于我们引用它。  
引用某个分组，会有两种情形：在JavaScript里引用它，在正则表达式里引用它。  

### 1. 分组和分支结构
这二者是括号最直觉的作用，也是最原始的功能。  

#### 1.1 分组
我们知道/a+/匹配连续出现的“a”，而要匹配连续出现的“ab”时，需要使用/(ab)+/。   
其中括号是提供分组功能，使量词+作用于“ab”这个整体，测试如下：   
```
    const regex = /(ab)+/g;
    const string = "ababa abbb ababab";
    console.log( string.match(regex) );
    // => ["abab", "ab", "ababab"]
```

#### 1.2 分支结构
而在多选分支结构(p1|p2)中，此处括号的作用也是不言而喻的，提供了子表达式的所有可能。  
比如，要匹配如下的字符串：  
I love JavaScript  
I love Regular Expression
可以使用正则：  
```
    const regex = /^I love (JavaScript|Regular Expression)$/;
    console.log( regex.test("I love JavaScript") );
    console.log( regex.test("I love Regular Expression") );
    // => true
    // => true
```
如果去掉正则中的括号，即/^I love JavaScript|Regular Expression$/，匹配字符串是"I love JavaScript"和"Regular Expression"，当然这不是我们想要的。  

### 2. 引用分组
这是括号一个重要的作用，有了它，我们就可以进行数据提取，以及更强大的替换操作。   
而要使用它带来的好处，必须配合使用实现环境的API。   
以日期为例。假设格式是yyyy-mm-dd的，我们可以先写一个简单的正则：   
`const regex = /\d{4}-\d{2}-\d{2}/;`
然后再修改成括号版的：
`const regex = /(\d{4})-(\d{2})-(\d{2})/;`

#### 2.1 提取数据
比如提取出年、月、日，可以这么做：  
```
    const regex = /(\d{4})-(\d{2})-(\d{2})/;
    const string = "2017-06-12";
    console.log( string.match(regex) );
    // => ["2017-06-12", "2017", "06", "12", index: 0, input: "2017-06-12"]
```
同时，也可以使用构造函数的全局属性$1至$9来获取：   
```
    const regex = /(\d{4})-(\d{2})-(\d{2})/;
    const string = "2017-06-12";

    regex.test(string); // 正则操作即可，例如
    //regex.exec(string);
    //string.match(regex);

    console.log(RegExp.$1); // "2017"
    console.log(RegExp.$2); // "06"
    console.log(RegExp.$3); // "12"
```

#### 2.2 替换
```
    const regex = /(\d{4})-(\d{2})-(\d{2})/;
    const string = "2017-06-12";
    const result = string.replace(regex, "$2/$3/$1");
    console.log(result);
    // => "06/12/2017"
```
其中replace中的，第二个参数里用$1、$2、$3指代相应的分组。等价于如下的形式：  
```
    const regex = /(\d{4})-(\d{2})-(\d{2})/;
    const string = "2017-06-12";
    const result = string.replace(regex, function(match, year, month, day) {
    	return month + "/" + day + "/" + year;
    });
    console.log(result);
    // => "06/12/2017"
```
也等价于：   
```
    const regex = /(\d{4})-(\d{2})-(\d{2})/;
    const string = "2017-06-12";
    const result = string.replace(regex, function(match, year, month, day) {
    	return month + "/" + day + "/" + year;
    });
    console.log(result);
    // => "06/12/2017"
```

### 3. 反向引用
除了使用相应API来引用分组，也可以在正则本身里引用分组。但只能引用之前出现的分组，即反向引用。   
还是以日期为例。   
比如要写一个正则支持匹配如下三种格式：   
2016-06-12   
2016/06/12   
2016.06.12  
最先可能想到的正则是:   
```
    const regex = /\d{4}(-|\/|\.)\d{2}(-|\/|\.)\d{2}/;
    const string1 = "2017-06-12";
    const string2 = "2017/06/12";
    const string3 = "2017.06.12";
    const string4 = "2016-06/12";
    console.log( regex.test(string1) ); // true
    console.log( regex.test(string2) ); // true
    console.log( regex.test(string3) ); // true
    console.log( regex.test(string4) ); // true
```
其中/和.需要转义。虽然匹配了要求的情况，但也匹配"2016-06/12"这样的数据。  
假设我们想要求分割符前后一致怎么办？此时需要使用反向引用：   
```
    const regex = /\d{4}(-|\/|\.)\d{2}\1\d{2}/;
    const string1 = "2017-06-12";
    const string2 = "2017/06/12";
    const string3 = "2017.06.12";
    const string4 = "2016-06/12";
    console.log( regex.test(string1) ); // true
    console.log( regex.test(string2) ); // true
    console.log( regex.test(string3) ); // true
    console.log( regex.test(string4) ); // false
```
注意里面的\1，表示的引用之前的那个分组(-|\/|\.)。不管它匹配到什么（比如-），\1都匹配那个同样的具体某个字符。  
我们知道了\1的含义后，那么\2和\3的概念也就理解了，即分别指代第二个和第三个分组。看到这里，此时，恐怕你会有三个问题。  

#### 3.1 括号嵌套怎么办？
以左括号（开括号）为准。比如：  
```
    const regex = /^((\d)(\d(\d)))\1\2\3\4$/;
    const string = "1231231233";
    console.log( regex.test(string) ); // true
    console.log( RegExp.$1 ); // 123
    console.log( RegExp.$2 ); // 1
    console.log( RegExp.$3 ); // 23
    console.log( RegExp.$4 ); // 3
```
我们可以看看这个正则匹配模式：  
- 第一个字符是数字，比如说1，
- 第二个字符是数字，比如说2，
- 第三个字符是数字，比如说3，
- 接下来的是\1，是第一个分组内容，那么看第一个开括号对应的分组是什么，是123，
- 接下来的是\2，找到第2个开括号，对应的分组，匹配的内容是1，
- 接下来的是\3，找到第3个开括号，对应的分组，匹配的内容是23，
- 最后的是\4，找到第3个开括号，对应的分组，匹配的内容是3。
这个问题，估计仔细看一下，就该明白了。  

#### 3.2 \10表示什么呢？
另外一个疑问可能是，即\10是表示第10个分组，还是\1和0呢？  
答案是前者，虽然一个正则里出现\10比较罕见。测试如下：  
```
    const regex = /(1)(2)(3)(4)(5)(6)(7)(8)(9)(#) \10+/;
    const string = "123456789# ######"
    console.log( regex.test(string) );
    // => true
```

#### 3.3 引用不存在的分组会怎样？
因为反向引用，是引用前面的分组，但我们在正则里引用了不存在的分组时，此时正则不会报错，只是匹配反向引用的字符本身。例如\2，就匹配"\2"。注意"\2"表示对"2"进行了转意。  
```
    const regex = /\1\2\3\4\5\6\7\8\9/;
    console.log( regex.test("\1\2\3\4\5\6\7\8\9") );
    console.log( "\1\2\3\4\5\6\7\8\9".split("") );
```
chrome浏览器打印的结果：   
![chrome浏览器打印的结果](https://user-gold-cdn.xitu.io/2017/7/19/f75ad2642625466dd5adcad3e2a4c51a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 "chrome浏览器打印的结果")

### 4. 非捕获分组
之前文中出现的分组，都会捕获它们匹配到的数据，以便后续引用，因此也称他们是捕获型分组。  
如果只想要括号最原始的功能，但不会引用它，即，既不在API里引用，也不在正则里反向引用。此时可以使用非捕获分组(?:p)，例如本文第一个例子可以修改为：  
```
    const regex = /(?:ab)+/g;
    const string = "ababa abbb ababab";
    console.log( string.match(regex) );
    // => ["abab", "ab", "ababab"]
```
### 5. 相关案例
至此括号的作用已经讲完了，总结一句话，就是提供了可供我们使用的分组，如何用就看我们的了。   

#### 5.1 字符串trim方法模拟   
trim方法是去掉字符串的开头和结尾的空白符。有两种思路去做。   
第一种，匹配到开头和结尾的空白符，然后替换成空字符。如：   
```
    function trim(str) {
    	return str.replace(/^\s+|\s+$/g, '');
    }
    console.log( trim("  foobar   ") );
    // => "foobar"
```
第二种，匹配整个字符串，然后用引用来提取出相应的数据：  
```
    function trim(str) {
    	return str.replace(/^\s*(.*?)\s*$/g, "$1");
    }
    console.log( trim("  foobar   ") );
    // => "foobar"
```
这里使用了惰性匹配*?，不然也会匹配最后一个空格之前的所有空格的。  
当然，前者效率高。  

#### 5.2 将每个单词的首字母转换为大写
```
    function titleize(str) {
        return str.toLowerCase().replace(/(?:^|\s)\w/g, function(c) {
            return c.toUpperCase();
        });
    }
    console.log( titleize('my name is epeli') );
```
思路是找到每个单词的首字母，当然这里不使用非捕获匹配也是可以的。  

#### 5.3 驼峰化
```
    function camelize(str) {
    	return str.replace(/[-_\s]+(.)?/g, function(match, c) {
    		return c ? c.toUpperCase() : '';
    	});
    }
    console.log( camelize('-moz-transform') );
    // => "MozTransform"
```
其中分组(.)表示首字母。单词的界定是，前面的字符可以是多个连字符、下划线以及空白符。正则后面的?的目的，是为了应对str尾部的字符可能不是单词字符，比如str是'-moz-transform'。  

#### 5.4 匹配成对标签
要求匹配：  
```
<title>regular expression</title>
<p>bye bye</p>
```
不匹配：  
`<title>wrong!</p>`
匹配一个开标签，可以使用正则<[^>]+>，  
匹配一个闭标签，可以使用<\/[^>]+>，  
但是要求匹配成对标签，那就需要使用反向引用，如：  
```
    const regex = /<([^>]+)>[\d\D]*<\/\1>/;
    const string1 = "<title>regular expression</title>";
    const string2 = "<p>laoyao bye bye</p>";
    const string3 = "<title>wrong!</p>";
    console.log( regex.test(string1) ); // true
    console.log( regex.test(string2) ); // true
    console.log( regex.test(string3) ); // false
```
其中开标签<[^>]+>改成<([^>]+)>，使用括号的目的是为了后面使用反向引用，而提供分组。闭标签使用了反向引用，<\/\1>。  
另外[\d\D]的意思是，这个字符是数字或者不是数字，因此，也就是匹配任意字符的意思。  

## <span id="chapter4">第四章 正则表达式回溯法原理</span>

### 1. 没有回溯的匹配
假设我们的正则是/ab{1,3}c/，其可视化形式是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/d92077b25d4faf8073d38999294f746c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
而当目标字符串是"abbbc"时，就没有所谓的“回溯”。其匹配过程是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/3f6e829c62fca181d818205e0e08bf73?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
其中子表达式b{1,3}表示“b”字符连续出现1到3次。  

### 2. 有回溯的匹配
如果目标字符串是"abbc"，中间就有回溯。  
![error](https://user-gold-cdn.xitu.io/2017/7/19/e58356622f6087437f33cdce7ce7bd3d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
图中第5步有红颜色，表示匹配不成功。此时b{1,3}已经匹配到了2个字符“b”，准备尝试第三个时，结果发现接下来的字符是“c”。那么就认为b{1,3}就已经匹配完毕。然后状态又回到之前的状态（即第6步，与第4步一样），最后再用子表达式c，去匹配字符“c”。当然，此时整个表达式匹配成功了。  
图中的第6步，就是“回溯”。  
你可能对此没有感觉，这里我们再举一个例子。正则是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/12da8829af2cb1d67ea78631d58be6ce?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
目标字符串是"abbbc"，匹配过程是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/dddfffaf633dd14c4eefba488f64400f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
其中第7步和第10步是回溯。第7步与第4步一样，此时b{1,3}匹配了两个"b"，而第10步与第3步一样，此时b{1,3}只匹配了一个"b"，这也是b{1,3}的最终匹配结果。   
这里再看一个清晰的回溯，正则是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/a9e420776eb01c07979f1599e4060775?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
目标字符串是："acd"ef，匹配过程是：  

### 3. 常见的回溯形式
正则表达式匹配字符串的这种方式，有个学名，叫回溯法。  
回溯法也称试探法，它的基本思想是：从问题的某一种状态（初始状态）出发，搜索从这种状态出发所能达到的所有“状态”，当一条路走到“尽头”的时候（不能再前进），再后退一步或若干步，从另一种可能“状态”出发，继续搜索，直到所有的“路径”（状态）都试探过。这种不断“前进”、不断“回溯”寻找解的方法，就称作“回溯法”。（copy于百度百科）。  
本质上就是深度优先搜索算法。其中退到之前的某一步这一过程，我们称为“回溯”。从上面的描述过程中，可以看出，路走不通时，就会发生“回溯”。即，尝试匹配失败时，接下来的一步通常就是回溯。  
道理，我们是懂了。那么JS中正则表达式会产生回溯的地方都有哪些呢？  

#### 3.1 贪婪量词
之前的例子都是贪婪量词相关的。比如b{1,3}，因为其是贪婪的，尝试可能的顺序是从多往少的方向去尝试。首先会尝试"bbb"，然后再看整个正则是否能匹配。不能匹配时，吐出一个"b"，即在"bb"的基础上，再继续尝试。如果还不行，再吐出一个，再试。如果还不行呢？只能说明匹配失败了。  
虽然局部匹配是贪婪的，但也要满足整体能正确匹配。否则，皮之不存，毛将焉附？  
此时我们不禁会问，如果当多个贪婪量词挨着存在，并相互有冲突时，此时会是怎样？  
答案是，先下手为强！因为深度优先搜索。测试如下：  
```
    const string = "12345";
    const regex = /(\d{1,3})(\d{1,3})/;
    console.log( string.match(regex) );
    // => ["12345", "123", "45", index: 0, input: "12345"]
```

#### 3.2 惰性量词
惰性量词就是在贪婪量词后面加个问号。表示尽可能少的匹配，比如：  
```
    const string = "12345";
    const regex = /(\d{1,3}?)(\d{1,3})/;
    console.log( string.match(regex) );
    // => ["1234", "1", "234", index: 0, input: "12345"]
```
其中\d{1,3}?只匹配到一个字符"1"，而后面的\d{1,3}匹配了"234"。  
虽然惰性量词不贪，但也会有回溯的现象。比如正则是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/0e29c26dd50349760d05935c5e93f07b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
目标字符串是"12345"，匹配过程是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/a2af73fc275cddf7c9c5fb5a786861c0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
知道你不贪、很知足，但是为了整体匹配成，没办法，也只能给你多塞点了。因此最后\d{1,3}?匹配的字符是"12"，是两个数字，而不是一个。  
#### 3.3 分支结构  
我们知道分支也是惰性的，比如/can|candy/，去匹配字符串"candy"，得到的结果是"can"，因为分支会一个一个尝试，如果前面的满足了，后面就不会再试验了。  
分支结构，可能前面的子模式会形成了局部匹配，如果接下来表达式整体不匹配时，仍会继续尝试剩下的分支。这种尝试也可以看成一种回溯。  
比如正则：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/4aedaeda72a6d291b9a8685cc0170347?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
目标字符串是"candy"，匹配过程：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/d69d02dfe0712ee3d22d5bb1afcda0a2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
上面第5步，虽然没有回到之前的状态，但仍然回到了分支结构，尝试下一种可能。所以，可以认为它是一种回溯的。  

### 4. 小结
其实回溯法，很容易掌握的。  
简单总结就是，正因为有多种可能，所以要一个一个试。直到，要么到某一步时，整体匹配成功了；要么最后都试完后，发现整体匹配不成功。  
1. 贪婪量词“试”的策略是：买衣服砍价。价钱太高了，便宜点，不行，再便宜点。
2. 惰性量词“试”的策略是：卖东西加价。给少了，再多给点行不，还有点少啊，再给点。
3. 分支结构“试”的策略是：货比三家。这家不行，换一家吧，还不行，再换。
既然有回溯的过程，那么匹配效率肯定低一些。相对谁呢？相对那些DFA引擎。  
而JS的正则引擎是NFA，NFA是“非确定型有限自动机”的简写。  
大部分语言中的正则都是NFA，为啥它这么流行呢？  
答：你别看我匹配慢，但是我编译快啊，而且我还有趣哦。  

## <span id="chapter5">第五章 正则表达式的拆分</span>
对于一门语言的掌握程度怎么样，可以有两个角度来衡量：读和写。  
不仅要求自己能解决问题，还要看懂别人的解决方案。代码是这样，正则表达式也是这样。  
正则这门语言跟其他语言有一点不同，它通常就是一大堆字符，而没有所谓“语句”的概念。  
如何能正确地把一大串正则拆分成一块一块的，成为了破解“天书”的关键。  

### 1. 结构和操作符
编程语言一般都有操作符。只要有操作符，就会出现一个问题。当一大堆操作在一起时，先操作谁，又后操作谁呢？为了不产生歧义，就需要语言本身定义好操作顺序，即所谓的优先级。  
而在正则表达式中，操作符都体现在结构中，即由特殊字符和普通字符所代表的一个个特殊整体。  
JS正则表达式中，都有哪些结构呢？  
>字符字面量、字符组、量词、锚字符、分组、选择分支、反向引用。  

具体含义简要回顾如下（如懂，可以略去不看）：
>字面量，匹配一个具体字符，包括不用转义的和需要转义的。比如a匹配字符"a"，又比如\n匹配换行符，又比如\.匹配小数点。  
>字符组，匹配一个字符，可以是多种可能之一，比如[0-9]，表示匹配一个数字。也有\d的简写形式。另外还有反义字符组，表示可以是除了特定字符之外任何一个字符，比如[^0-9]，表示一个非数字字符，也有\D的简写形式。  
>量词，表示一个字符连续出现，比如a{1,3}表示“a”字符连续出现3次。另外还有常见的简写形式，比如a+表示“a”字符连续出现至少一次。  
>锚点，匹配一个位置，而不是字符。比如^匹配字符串的开头，又比如\b匹配单词边界，又比如(?=\d)表示数字前面的位置。  
>分组，用括号表示一个整体，比如(ab)+，表示"ab"两个字符连续出现多次，也可以使用非捕获分组(?:ab)+。  
>分支，多个子表达式多选一，比如abc|bcd，表达式匹配"abc"或者"bcd"字符子串。反向引用，比如\2，表示引用第2个分组。


其中涉及到的操作符有：  
>1.转义符 \  
2.括号和方括号 (...)、(?:...)、(?=...)、(?!...)、[...]  
3.量词限定符 {m}、{m,n}、{m,}、?、\*、+  
4.位置和序列 ^ 、$、 \元字符、 一般字符  
5.管道符（竖杠）|  

上面操作符的优先级从上至下，由高到低。
这里，我们来分析一个正则：  
`/ab?(c|de*)+|fg/`
1.由于括号的存在，所以，(c|de*)是一个整体结构。  
2.在(c|de*)中，注意其中的量词*，因此e*是一个整体结构。  
3.又因为分支结构“|”优先级最低，因此c是一个整体、而de*是另一个整体。  
4.同理，整个正则分成了 a、b?、(...)+、f、g。而由于分支的原因，又可以分成ab?(c|de*)+和fg这两部分。  
希望你没被我绕晕，上面的分析可用其可视化形式描述如下：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/4bb44a11e383047a027a234ee15663ad?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  

### 2. 注意要点
关于结构和操作符，还是有几点需要强调：  

#### 2.1 匹配字符串整体问题
因为是要匹配整个字符串，我们经常会在正则前后中加上锚字符^和$。  
比如要匹配目标字符串"abc"或者"bcd"时，如果一不小心，就会写成/^abc|bcd$/。  
而位置字符和字符序列优先级要比竖杠高，故其匹配的结构是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/173e8c288f0da4ed89df597551fa80db?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')
应该修改成:  
![error](https://user-gold-cdn.xitu.io/2017/7/19/c3bcb26f1b87e43bab1497322f5107e5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')

#### 2.2 量词连缀问题
假设，要匹配这样的字符串：  
>1.每个字符为a、b、c任选其一  
2.字符串的长度是3的倍数  

此时正则不能想当然地写成/^[abc]{3}+$/，这样会报错，说+前面没什么可重复的：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/678d0621feb4bdcea899a6f61628a521?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')
#### 2.3 元字符转义问题
所谓元字符，就是正则中有特殊含义的字符。  
所有结构里，用到的元字符总结如下：  
> ^ $ . * + ? | \ / ( ) [ ] { } = ! : - ,

当匹配上面的字符本身时，可以一律转义：
```
    const string = "^$.*+?|\\/[]{}=!:-,";
    const regex = /\^\$\.\*\+\?\|\\\/\[\]\{\}\=\!\:\-\,/;
    console.log( regex.test(string) );
    // => true
```
其中string中的\字符也要转义的。  
另外，在string中，也可以把每个字符转义，当然，转义后的结果仍是本身：  
```
    const string = "^$.*+?|\\/[]{}=!:-,";
    const string2 = "\^\$\.\*\+\?\|\\\/\[\]\{\}\=\!\:\-\,";
    console.log( string === string2 );
    // => true
```
现在的问题是，是不是每个字符都需要转义呢？否，看情况。  

##### 2.3.1 字符组中的元字符
跟字符组相关的元字符有[]、^、-。因此在会引起歧义的地方进行转义。例如开头的^必须转义，不然会把整个字符组，看成反义字符组。   
```
    const string = "^$.*+?|\\/[]{}=!:-,";
    const regex = /[\^$.*+?|\\/\[\]{}=!:\-,]/g;
    console.log( string.match(regex) );
    // => ["^", "$", ".", "*", "+", "?", "|", "\", "/", "[", "]", "{", "}", "=", "!", ":", "-", ","]
```

##### 2.3.2 匹配“[abc]”和“{3,5}”
我们知道[abc]，是个字符组。如果要匹配字符串"[abc]"时，该怎么办？  
可以写成/\[abc\]/，也可以写成/\[abc]/，测试如下：  
```
    const string = "[abc]";
    const regex = /\[abc]/g;
    console.log( string.match(regex)[0] );
    // => "[abc]"
```
只需要在第一个方括号转义即可，因为后面的方括号构不成字符组，正则不会引发歧义，自然不需要转义。  
同理，要匹配字符串"{3,5}"，只需要把正则写成/\{3,5}/即可。  
另外，我们知道量词有简写形式{m,}，却没有{,n}的情况。虽然后者不构成量词的形式，但此时并不会报错。当然，匹配的字符串也是"{,n}"，测试如下：  
```
    const string = "{,3}";
    const regex = /{,3}/g;
    console.log( string.match(regex)[0] );
    // => "{,3}"
```
##### 2.3.3 其余情况
比如= ! : - ,等符号，只要不在特殊结构中，也不需要转义。   
但是，括号需要前后都转义的，如/\(123\)/。  
至于剩下的^ $ . * + ? | \ /等字符，只要不在字符组内，都需要转义的。  

### 3. 案例分析
接下来分析两个例子，一个简单的，一个复杂的。  

#### 3.1 身份证
正则表达式是：  
`/^(\d{15}|\d{17}[\dxX])$/`
因为竖杠“|”,的优先级最低，所以正则分成了两部分\d{15}和\d{17}[\dxX]。   
- \d{15}表示15位连续数字。
- \d{17}[\dxX]表示17位连续数字，最后一位可以是数字可以大小写字母"x"。  
可视化如下：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/e47d20e904f9b5665942a01fc9d6111d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')

## <span id="chapter6">第六章 正则表达式的构建</span>
对于一门语言的掌握程度怎么样，可以有两个角度来衡量：读和写。   
不仅要看懂别人的解决方案，也要能独立地解决问题。代码是这样，正则表达式也是这样。   
与“读”相比，“写”往往更为重要，这个道理是不言而喻的。   
对正则的运用，首重就是：如何针对问题，构建一个合适的正则表达式？   

### 1. 平衡法则
构建正则有一点非常重要，需要做到下面几点的平衡：  
1. 匹配预期的字符串
2. 不匹配非预期的字符串
3. 可读性和可维护性
4. 效率

### 2. 构建正则前提

#### 2.1 是否能使用正则
正则太强大了，以至于我们随便遇到一个操作字符串问题时，都会下意识地去想，用正则该怎么做。但我们始终要提醒自己，正则虽然强大，但不是万能的，很多看似很简单的事情，还是做不到的。  
比如匹配这样的字符串：1010010001....  
虽然很有规律，但是只靠正则就是无能为力。  

#### 2.2 是否有必要使用正则
要认识到正则的局限，不要去研究根本无法完成的任务。同时，也不能走入另一个极端：无所不用正则。能用字符串API解决的简单问题，就不该正则出马。   
- 比如，从日期中提取出年月日，虽然可以使用正则：  
```
    const string = "2017-07-01";
    const regex = /^(\d{4})-(\d{2})-(\d{2})/;
    console.log( string.match(regex) );
    // => ["2017-07-01", "2017", "07", "01", index: 0, input: "2017-07-01"]
```
其实，可以使用字符串的split方法来做，即可：   
```
    const string = "2017-07-01";
    const result = string.split("-");
    console.log( result );
    // => ["2017", "07", "01"]
```
- 比如，判断是否有问号，虽然可以使用：  
```
    const string = "?id=xx&act=search";
    console.log( string.search(/\?/) );
    // => 0
```
其实，可以使用字符串的indexOf方法：   
```
    const string = "?id=xx&act=search";
    console.log( string.indexOf("?") );
    // => 0
```
- 比如获取子串，虽然可以使用正则：
```
    const string = "JavaScript";
    console.log( string.match(/.{4}(.+)/)[1] );
    // => Script
```
其实，可以直接使用字符串的substring或substr方法来做：  
```
    const string = "JavaScript";
    console.log( string.substring(4) );
    // => Script
```

#### 2.3 是否有必要构建一个复杂的正则
比如密码匹配问题，要求密码长度6-12位，由数字、小写字符和大写字母组成，但必须至少包括2种字符。   
在第2章里，我们写出了正则是：   
`/(?!^[0-9]{6,12}$)(?!^[a-z]{6,12}$)(?!^[A-Z]{6,12}$)^[0-9A-Za-z]{6,12}$/`
其实可以使用多个小正则来做：   
```
    const regex1 = /^[0-9A-Za-z]{6,12}$/;
    const regex2 = /^[0-9]{6,12}$/;
    const regex3 = /^[A-Z]{6,12}$/;
    const regex4 = /^[a-z]{6,12}$/;
    function checkPassword(string) {
    	if (!regex1.test(string)) return false;
    	if (regex2.test(string)) return false;
    	if (regex3.test(string)) return false;
    	if (regex4.test(string)) return false;
    	return true;
    }
```

### 3. 准确性
所谓准确性，就是能匹配预期的目标，并且不匹配非预期的目标。   
这里提到了“预期”二字，那么我们就需要知道目标的组成规则。  
不然没法界定什么样的目标字符串是符合预期的，什么样的又不是符合预期的。  
下面将举例说明，当目标字符串构成比较复杂时，该如何构建正则，并考虑到哪些平衡。  

#### 3.1 匹配固定电话
比如要匹配如下格式的固定电话号码：   
>055188888888
0551-88888888
(0551)88888888

第一步，了解各部分的模式规则。  
上面的电话，总体上分为区号和号码两部分（不考虑分机号和+86的情形）。  
区号是0开头的3到4位数字，对应的正则是：0\d{2,3}  
号码是非0开头的7到8位数字，对应的正则是：[1-9]\d{6,7}  
因此，匹配055188888888的正则是：/^0\d{2,3}[1-9]\d{6,7}$/  
匹配0551-88888888的正则是：/^0\d{2,3}-[1-9]\d{6,7}$/  
匹配(0551)88888888的正则是：/^\(0\d{2,3}\)[1-9]\d{6,7}$/  
第二步，明确形式关系。  
这三者情形是或的关系，可以构建分支：  
`/^0\d{2,3}[1-9]\d{6,7}$|^0\d{2,3}-[1-9]\d{6,7}$|^\(0\d{2,3}\)[1-9]\d{6,7}$/`  
提取公共部分：  
`/^(0\d{2,3}|0\d{2,3}-|\(0\d{2,3}\))[1-9]\d{6,7}$/`  
进一步简写：  
`/^(0\d{2,3}-?|\(0\d{2,3}\))[1-9]\d{6,7}$/`  
其可视化形式：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/225f104c7293c5a174f7c9394d755f13?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')
上面的正则构建过程略显罗嗦，但是这样做，能保证正则是准确的。  
上述三种情形是或的关系，这一点很重要，不然很容易按字符是否出现的情形把正则写成：  
`/^\(?0\d{2,3}\)?-?[1-9]\d{6,7}$/`  
虽然也能匹配上述目标字符串，但也会匹配(0551-88888888这样的字符串。当然，这不是我们想要的。  
其实这个正则也不是完美的，因为现实中，并不是每个3位数和4位数都是一个真实的区号。  
这就是一个平衡取舍问题，一般够用就行。  

#### 3.2 匹配浮点数
要求匹配如下的格式：   
>1.23、+1.23、-1.23
10、+10、-10
.2、+.2、-.2

可以看出正则分为三部分。  
符号部分：[+-]  
整数部分：\d+  
小数部分：\.\d+  
上述三个部分，并不是全部都出现。如果此时很容易写出如下的正则：  
`/^[+-]?(\d+)?(\.\d+)?$/`  
此正则看似没问题，但这个正则也会匹配空字符""。  
因为目标字符串的形式关系不是要求每部分都是可选的。  
要匹配1.23、+1.23、-1.23，可以用/^[+-]?\d+\.\d+$/  
要匹配10、+10、-10，可以用/^[+-]?\d+$/  
要匹配.2、+.2、-.2，可以用/^[+-]?\.\d+$/  
因此整个正则是这三者的或的关系，提取公共部分后是：  
/^[+-]?(\d+\.\d+|\d+|\.\d+)$/  
其可视化形式是：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/f48457e329e8598e113b42d699744c9b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
如果要求不匹配+.2和-.2，此时正则变成：  
![error](https://user-gold-cdn.xitu.io/2017/7/19/8c7263b0465bc26e5062d9978906f578?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')  
当然，/^[+-]?(\d+\.\d+|\d+|\.\d+)$/也不是完美的，我们也是做了些取舍，比如：  
- 它也会匹配012这样以0开头的整数。如果要求不匹配的话，需要修改整数部分的正则。
- 一般进行验证操作之前，都要经过trim和判空。那样的话，也许那个错误正则也就够用了。
- 也可以进一步改写成：/^[+-]?(\d+)?(\.)?\d+$/，这样我们就需要考虑可读性和可维护性了。

### 4. 效率
保证了准确性后，才需要是否要考虑要优化。大多数情形是不需要优化的，除非运行的非常慢。什么情形正则表达式运行才慢呢？我们需要考察正则表达式的运行过程（原理）。  
正则表达式的运行分为如下的阶段：  
1. 编译
2. 设定起始位置
3. 尝试匹配
4. 匹配失败的话，从下一位开始继续第3步
5. 最终结果：匹配成功或失败
下面以代码为例，来看看这几个阶段都做了什么：  
```
    const regex = /\d+/g;
    console.log( regex.lastIndex, regex.exec("123abc34def") );
    console.log( regex.lastIndex, regex.exec("123abc34def") );
    console.log( regex.lastIndex, regex.exec("123abc34def") );
    console.log( regex.lastIndex, regex.exec("123abc34def") );
    // => 0 ["123", index: 0, input: "123abc34def"]
    // => 3 ["34", index: 6, input: "123abc34def"]
    // => 8 null
    // => 0 ["123", index: 0, input: "123abc34def"]
```
具体分析如下：  
`const regex = /\d+/g;`
当生成一个正则时，引擎会对其进行编译。报错与否出现这这个阶段。  
`regex.exec("123abc34def")`  
当尝试匹配时，需要确定从哪一位置开始匹配。一般情形都是字符串的开头，即第0位。  
但当使用test和exec方法，且正则有g时，起始位置是从正则对象的lastIndex属性开始。  
因此第一次exec是从第0位开始，而第二次是从3开始的。  
设定好起始位置后，就开始尝试匹配了。  
比如第一次exec，从0开始，去尝试匹配，并且成功地匹配到3个数字。此时结束时的下标是2，因此下一次的起始位置是3。  
而第二次，起始下标是3，但第3个字符是“a”，并不是数字。但此时并不会直接报匹配失败，而是移动到下一位置，即从第4位开始继续尝试匹配，但该字符是b，也不是数字。再移动到下一位，是c仍不行，再移动一位是数字3，此时匹配到了两位数字34。此时，下一次匹配的位置是d的位置，即第8位。  
第三次，是从第8位开始匹配，直到试到最后一位，也没发现匹配的，因此匹配失败，返回null。同时设置lastIndex为0，即，如要再尝试匹配的话，需从头开始。  
从上面可以看出，匹配会出现效率问题，主要出现在上面的第3阶段和第4阶段。  
因此，主要优化手法也是针对这两阶段的。  

#### 4.1 使用具体型字符组来代替通配符，来消除回溯
而在第三阶段，最大的问题就是回溯。  
例如，匹配双引用号之间的字符。如，匹配字符串123"abc"456中的"abc"。
如果正则用的是：/".\*"/，，会在第3阶段产生4次回溯（粉色表示.\*匹配的内容）：   
![error](https://user-gold-cdn.xitu.io/2017/7/19/5b677f04f2b8d5d776060cea3b045863?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')   

如果正则用的是：/".\*?"/，会产生2次回溯（粉色表示.\*?匹配的内容）：
![error](https://user-gold-cdn.xitu.io/2017/7/19/a064ff019c3c6cdde6005b6c83b60e7b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 'image')   
因为回溯的存在，需要引擎保存多种可能中未尝试过的状态，以便后续回溯时使用。注定要占用一定的内存。  
此时要使用具体化的字符组，来代替通配符.，以便消除不必要的字符，此时使用正则/"[^"]\*"/，即可。  

#### 4.2 使用非捕获型分组
因为括号的作用之一是，可以捕获分组和分支里的数据。那么就需要内存来保存它们。  
当我们不需要使用分组引用和反向引用时，此时可以使用非捕获分组。例如：  
`/^[+-]?(\d+\.\d+|\d+|\.\d+)$/`
可以修改成：  
`/^[+-]?(?:\d+\.\d+|\d+|\.\d+)$/`  

#### 4.3 独立出确定字符
例如/a+/，可以修改成/aa*/。  
因为后者能比前者多确定了字符a。这样会在第四步中，加快判断是否匹配失败，进而加快移位的速度。  

#### 4.4 提取分支公共部分
比如/^abc|^def/，修改成/^(?:abc|def)/。  
又比如/this|that/，修改成/th(?:is|at)/。  
这样做，可以减少匹配过程中可消除的重复。  

#### 4.5 减少分支的数量，缩小它们的范围
/red|read/，可以修改成/rea?d/。此时分支和量词产生的回溯的成本是不一样的。但这样优化后，可读性会降低的。  

## <span id="chapter7">第七章 正则表达式编程</span>
什么叫知识，能指导我们实践的东西才叫知识。  
学习一样东西，如果不能使用，最多只能算作纸上谈兵。正则表达式的学习，也不例外。  

### 1. 正则表达式的四种操作  
正则表达式是匹配模式，不管如何使用正则表达式，万变不离其宗，都需要先“匹配”。  
有了匹配这一基本操作后，才有其他的操作：验证、切分、提取、替换。  
进行任何相关操作，也需要宿主引擎相关API的配合使用。当然，在JS中，相关API也不多。  

#### 1.1 验证  
验证是正则表达式最直接的应用，比如表单验证。  
在说验证之前，先要说清楚匹配是什么概念。  
所谓匹配，就是看目标字符串里是否有满足匹配的子串。因此，“匹配”的本质就是“查找”。  
有没有匹配，是不是匹配上，判断是否的操作，即称为“验证”。  

- 使用search
```
    const regex = /\d/;
    const string = "abc123";
    console.log( !!~string.search(regex) );
    // => true
```
- 使用test
```
    const regex = /\d/;
    const string = "abc123";
    console.log( regex.test(string) );
    // => true
```
- 使用match  
```
    const regex = /\d/;
    const string = "abc123";
    console.log( !!string.match(regex) );
    // => true
```
- 使用exec
```
    const regex = /\d/;
    const string = "abc123";
    console.log( !!regex.exec(string) );
    // => true
```

#### 1.2 切分
匹配上了，我们就可以进行一些操作，比如切分。  
所谓“切分”，就是把目标字符串，切成一段一段的。在JS中使用的是split。  
比如，目标字符串是"html,css,javascript"，按逗号来切分：  
```
    const regex = /,/;
    const string = "html,css,javascript";
    console.log( string.split(regex) );
    // => ["html", "css", "javascript"]
```
又比如，如下的日期格式：  
>2017/06/26
2017.06.26
2017-06-26

可以使用split“切出”年月日：  
```
    const regex = /\D/;
    console.log( "2017/06/26".split(regex) );
    console.log( "2017.06.26".split(regex) );
    console.log( "2017-06-26".split(regex) );
    // => ["2017", "06", "26"]
    // => ["2017", "06", "26"]
    // => ["2017", "06", "26"]
```

#### 1.3 提取
虽然整体匹配上了，但有时需要提取部分匹配的数据。  
此时正则通常要使用分组引用（分组捕获）功能，还需要配合使用相关API。  
这里，还是以日期为例，提取出年月日。注意下面正则中的括号：  
- match
```
    const regex = /^(\d{4})\D(\d{2})\D(\d{2})$/;
    const string = "2017-06-26";
    console.log( string.match(regex) );
    // =>["2017-06-26", "2017", "06", "26", index: 0, input: "2017-06-26"]
```
- exec
```
    const regex = /^(\d{4})\D(\d{2})\D(\d{2})$/;
    const string = "2017-06-26";
    console.log( regex.exec(string) );
    // =>["2017-06-26", "2017", "06", "26", index: 0, input: "2017-06-26"]
```
- test
```
    const regex = /^(\d{4})\D(\d{2})\D(\d{2})$/;
    const string = "2017-06-26";
    regex.test(string);
    console.log( RegExp.$1, RegExp.$2, RegExp.$3 );
    // => "2017" "06" "26"
```
- search
```
    const regex = /^(\d{4})\D(\d{2})\D(\d{2})$/;
    const string = "2017-06-26";
    string.search(regex);
    console.log( RegExp.$1, RegExp.$2, RegExp.$3 );
    // => "2017" "06" "26"
```
- replace
```
    const regex = /^(\d{4})\D(\d{2})\D(\d{2})$/;
    const string = "2017-06-26";
    const date = [];
    string.replace(regex, function(match, year, month, day) {
    	date.push(year, month, day);
    });
    console.log(date);
    // => ["2017", "06", "26"]
```
其中，最常用的是match。  
#### 1.4 替换
找，往往不是目的，通常下一步是为了替换。在JS中，使用replace进行替换。  
比如把日期格式，从yyyy-mm-dd替换成yyyy/mm/dd：  
```
    const string = "2017-06-26";
    const today = new Date( string.replace(/-/g, "/") );
    console.log( today );
    // => Mon Jun 26 2017 00:00:00 GMT+0800 (中国标准时间)
```
这里只是简单地应用了一下replace。但，replace方法是强大的，是需要重点掌握的。  

### 2. 相关API注意要点
从上面可以看出用于正则操作的方法，共有6个，字符串实例4个，正则实例2个：  
>String#search
String#split
String#match
String#replace
RegExp#test
RegExp#exec

#### 2.1 search和match的参数问题
我们知道字符串实例的那4个方法参数都支持正则和字符串。  
但search和match，会把字符串转换为正则的。  
```
    const string = "2017.06.27";

    console.log( string.search(".") );
    // => 0
    //需要修改成下列形式之一
    console.log( string.search("\\.") );
    console.log( string.search(/\./) );
    // => 4
    // => 4

    console.log( string.match(".") );
    // => ["2", index: 0, input: "2017.06.27"]
    //需要修改成下列形式之一
    console.log( string.match("\\.") );
    console.log( string.match(/\./) );
    // => [".", index: 4, input: "2017.06.27"]
    // => [".", index: 4, input: "2017.06.27"]

    console.log( string.split(".") );
    // => ["2017", "06", "27"]

    console.log( string.replace(".", "/") );
    // => "2017/06.27"
```

#### 2.2 match返回结果的格式问题
match返回结果的格式，与正则对象是否有修饰符g有关。  
```
    const string = "2017.06.27";
    const regex1 = /\b(\d+)\b/;
    const regex2 = /\b(\d+)\b/g;
    console.log( string.match(regex1) );
    console.log( string.match(regex2) );
    // => ["2017", "2017", index: 0, input: "2017.06.27"]
    // => ["2017", "06", "27"]
```
没有g，返回的是标准匹配格式，即，数组的第一个元素是整体匹配的内容，接下来是分组捕获的内容，然后是整体匹配的第一个下标，最后是输入的目标字符串。  
有g，返回的是所有匹配的内容。  
当没有匹配时，不管有无g，都返回null。  

#### 2.3 exec比match更强大
当正则没有g时，使用match返回的信息比较多。但是有g后，就没有关键的信息index了。  
而exec方法就能解决这个问题，它能接着上一次匹配后继续匹配：  
```
    const string = "2017.06.27";
    const regex2 = /\b(\d+)\b/g;
    console.log( regex2.exec(string) );
    console.log( regex2.lastIndex);
    console.log( regex2.exec(string) );
    console.log( regex2.lastIndex);
    console.log( regex2.exec(string) );
    console.log( regex2.lastIndex);
    console.log( regex2.exec(string) );
    console.log( regex2.lastIndex);
    // => ["2017", "2017", index: 0, input: "2017.06.27"]
    // => 4
    // => ["06", "06", index: 5, input: "2017.06.27"]
    // => 7
    // => ["27", "27", index: 8, input: "2017.06.27"]
    // => 10
    // => null
    // => 0
```
其中正则实例lastIndex属性，表示下一次匹配开始的位置。   
比如第一次匹配了“2017”，开始下标是0，共4个字符，因此这次匹配结束的位置是3，下一次开始匹配的位置是4。  
从上述代码看出，在使用exec时，经常需要配合使用while循环：  
```
    const string = "2017.06.27";
    const regex2 = /\b(\d+)\b/g;
    const result;
    while ( result = regex2.exec(string) ) {
    	console.log( result, regex2.lastIndex );
    }
    // => ["2017", "2017", index: 0, input: "2017.06.27"] 4
    // => ["06", "06", index: 5, input: "2017.06.27"] 7
    // => ["27", "27", index: 8, input: "2017.06.27"] 10
```

#### 2.4 修饰符g，对exec和test的影响
上面提到了正则实例的lastIndex属性，表示尝试匹配时，从字符串的lastIndex位开始去匹配。  
字符串的四个方法，每次匹配时，都是从0开始的，即lastIndex属性始终不变。  
而正则实例的两个方法exec、test，当正则是全局匹配时，每一次匹配完成后，都会修改lastIndex。下面让我们以test为例，看看你是否会迷糊：  
```
    const regex = /a/g;
    console.log( regex.test("a"), regex.lastIndex );
    console.log( regex.test("aba"), regex.lastIndex );
    console.log( regex.test("ababc"), regex.lastIndex );
    // => true 1
    // => true 3
    // => false 0
```
注意上面代码中的第三次调用test，因为这一次尝试匹配，开始从下标lastIndex即3位置处开始查找，自然就找不到了。  
如果没有g，自然都是从字符串第0个字符处开始尝试匹配：  
```
    const regex = /a/;
    console.log( regex.test("a"), regex.lastIndex );
    console.log( regex.test("aba"), regex.lastIndex );
    console.log( regex.test("ababc"), regex.lastIndex );
    // => true 0
    // => true 0
    // => true 0
```

#### 2.5 test整体匹配时需要使用^和$
这个相对容易理解，因为test是看目标字符串中是否有子串匹配正则，即有部分匹配即可。  
```
    console.log( /123/.test("a123b") );
    // => true
    console.log( /^123$/.test("a123b") );
    // => false
    console.log( /^123$/.test("123") );
    // => true
```

#### 2.6 split相关注意事项
split方法看起来不起眼，但要注意的地方有两个的。  
第一，它可以有第二个参数，表示结果数组的最大长度：  
```
    const string = "html,css,javascript";
    console.log( string.split(/,/, 2) );
    // =>["html", "css"]
```
第二，正则使用分组时，结果数组中是包含分隔符的：  
```
    const string = "html,css,javascript";
    console.log( string.split(/(,)/) );
    // =>["html", ",", "css", ",", "javascript"]
```

#### 2.7 replace是很强大的
《JavaScript权威指南》认为exec是这6个API中最强大的，而我始终认为replace才是最强大的。因为它也能拿到该拿到的信息，然后可以假借替换之名，做些其他事情。   
总体来说replace有两种使用形式，这是因为它的第二个参数，可以是字符串，也可以是函数。  
当第二个参数是字符串时，如下的字符有特殊的含义：  
>$1,$2,...,$99 匹配第1~99个分组里捕获的文本
$& 匹配到的子串文本
$\` 匹配到的子串的左边文本
$' 匹配到的子串的右边文本
$$ 美元符号

例如，把"2,3,5"，变成"5=2+3"：  
```
    const result = "2,3,5".replace(/(\d+),(\d+),(\d+)/, "$3=$1+$2");
    console.log(result);
    // => "5=2+3"
```
又例如，把"2,3,5"，变成"222,333,555":   
```
    const result = "2,3,5".replace(/(\d+)/g, "$&$&$&");
    console.log(result);
    // => "222,333,555"
```
再例如，把"2+3=5"，变成"2+3=2+3=5=5":   
```
    const result = "2+3=5".replace(/=/, "$&$`$&$'$&");
    console.log(result);
    // => "2+3=2+3=5=5"
```
当第二个参数是函数时，我们需要注意该回调函数的参数具体是什么：  
```
    "1234 2345 3456".replace(/(\d)\d{2}(\d)/g, function(match, $1, $2, index, input) {
    	console.log([match, $1, $2, index, input]);
    });
    // => ["1234", "1", "4", 0, "1234 2345 3456"]
    // => ["2345", "2", "5", 5, "1234 2345 3456"]
    // => ["3456", "3", "6", 10, "1234 2345 3456"]
```
此时我们可以看到replace拿到的信息，并不比exec少。   

#### 2.8 使用构造函数需要注意的问题
一般不推荐使用构造函数生成正则，而应该优先使用字面量。因为用构造函数会多写很多\。  
```
    const string = "2017-06-27 2017.06.27 2017/06/27";
    const regex = /\d{4}(-|\.|\/)\d{2}\1\d{2}/g;
    console.log( string.match(regex) );
    // => ["2017-06-27", "2017.06.27", "2017/06/27"]

    regex = new RegExp("\\d{4}(-|\\.|\\/)\\d{2}\\1\\d{2}", "g");
    console.log( string.match(regex) );
    // => ["2017-06-27", "2017.06.27", "2017/06/27"]
```
#### 2.9 修饰符
ES5中修饰符，共3个：   
>g 全局匹配，即找到所有匹配的，单词是global
i 忽略字母大小写，单词ingoreCase
m 多行匹配，只影响^和$，二者变成行的概念，即行开头和行结尾。单词是multiline

当然正则对象也有相应的只读属性：  
```
    const regex = /\w/img;
    console.log( regex.global );
    console.log( regex.ignoreCase );
    console.log( regex.multiline );
    // => true
    // => true
    // => true
```

#### 2.10 source属性
正则实例对象属性，除了global、ingnoreCase、multiline、lastIndex属性之外，还有一个source属性。   
它什么时候有用呢？   
比如，在构建动态的正则表达式时，可以通过查看该属性，来确认构建出的正则到底是什么：  
```
    const className = "high";
    const regex = new RegExp("(^|\\s)" + className + "(\\s|$)");
    console.log( regex.source )
    // => (^|\s)high(\s|$) 即字符串"(^|\\s)high(\\s|$)"
```
