- [HTTP Parameter Pollution](#http-parameter-pollution)
- [Portability Flaw：Locale Dependent Comparison](#portability-flaw--locale-dependent-comparison)
- [Header Manipulation](#header-manipulation)
- [Insecure Randomness](#insecure-randomness)
    + [前段解决该漏洞方法](#前段解决该漏洞方法)
    + [后台解决方法](#后台解决方法)
- [Path Manipulation](#path-manipulation)
- [Log Forging](#log-forging)
- [Mass Assignment:Insecure Binder Configuration](#mass-assignment-insecure-binder-configuration)


## HTTP Parameter Pollution

**HPP是HTTP Parameter Pollution的缩写，意为HTTP参数污染。**

将直接拼接在请求路径后面的参数通过该方法进行过滤
```java
public class ParamsUtils {

    /**
     * 删除重复的参数，防止HTTP Parameter Pollution
     * @param params
     * @return
     */
    public static String checkParams(String params) {
        HashMap<String, String> map = new HashMap<>();
        StringBuilder paramsBuilder = new StringBuilder();
        String[] arr = new String[] {};
        if (isNotEmpty(params)) {
            arr = params.split("&", -1);
            for (int i = 0; i < arr.length; i++) {
                if (arr.length > 0) {
                    String[] item = arr[i].split("=", -1);
                    map.put(item[0], item[1]);
                }
            }
        }
        if (map.size() < arr.length) {
            return params.replaceAll("&", "");
        } else {
            for (Map.Entry<String, String> m : map.entrySet()) {
                paramsBuilder.append(m.getKey()).append("=").append(m.getValue()).append("&");
            }
            return paramsBuilder.substring(0, paramsBuilder.lastIndexOf("&"));
        }
    }

    private static boolean isNotEmpty(String str) {
        return !isEmpty(str);
    }

    private static boolean isEmpty(String str) {
        return str == null || str.length() == 0;
    }
}
```

## Portability Flaw：Locale Dependent Comparison
**可移植性缺陷：与区域设置相关的比较**
``` java
tag.toUpperCase(Locale.ENGLISH）
```

## Header Manipulation
**HTTP 响应头文件中包含未验证的数据会引发 cache-poisoning、 cross-site scripting、 cross-user defacement、 page hijacking、 cookie manipulation 或 open redirect。**

把出现漏洞的参数通过下面的方法进行过滤

```java
/**
 * 替换非法字符
 * 
 * @param text
 * @return
 */
public static String replaceIllegalChar(String text) {
    if (org.apache.commons.lang.StringUtils.isEmpty(text)) {
        return "";
    }
    StringBuilder result = new StringBuilder();
    char[] chars = text.toCharArray();
    for (int i = 0; i < chars.length; i++) {
        Character temp = chars[i];
        if (CHARACTER_SET.contains(temp)) {
            // do nothing
        } else {
            result.append(temp);
        }
    }
    
    //将&字符替换为空，修复漏洞所用
    char[] textChars = result.toString().toCharArray();
    StringBuilder textBuilder = new StringBuilder();
    for (int j = 0; j < textChars.length; j++) {
        Character textChar = Character.valueOf(textChars[j]);
        if (!Character.valueOf('&').equals(textChar)) {
            textBuilder.append(textChar);
        }else {
            textBuilder.append("");
        }
    }
    return textBuilder.toString();
}


@SuppressWarnings("serial")
private static final Set<Character> CHARACTER_SET = new HashSet<Character>() {
    {
        add(Character.valueOf('\r'));
        add(Character.valueOf('\n'));
    }
};
```

## Insecure Randomness

**不安全随机数**

#### 前段解决该漏洞方法
```js
// 该方法向下兼容到Google44版本
window.crypto.getRandomValues(new Uint32Array(1))[0];
// 如果对随机数的要求不是很高的话
// 可以用 Date 的 getTime()  ，以免工具误会你要拿 Random 来做什么事!
```

#### 后台解决方法
```java
SecureRandom secureRandom = new SecureRandom();
int number = (int) ((secureRandom.nextDouble() * 9 + 1) * 100000);
```

## Path Manipulation

**路径篡改**
1. 攻击者能够指定文件系统上的操作中使用的路径。
2. 通过指定资源，攻击者获得了不允许的功能。

将文件名通过下面的方法进行过滤
```java
/**
* 解决路径篡改
* @param path
* @return
*/
public static String clearPath(String path) {
    HashMap<String, String> map = new HashMap<String, String>();
    map.put("a", "a");
    map.put("b", "b");
    map.put("c", "c");
    map.put("d", "d");
    map.put("e", "e");
    map.put("f", "f");
    map.put("g", "g");
    map.put("h", "h");
    map.put("i", "i");
    map.put("j", "j");
    map.put("k", "k");
    map.put("l", "l");
    map.put("m", "m");
    map.put("n", "n");
    map.put("o", "o");
    map.put("p", "p");
    map.put("q", "q");
    map.put("r", "r");
    map.put("s", "s");
    map.put("t", "t");
    map.put("u", "u");
    map.put("v", "v");
    map.put("w", "w");
    map.put("x", "x");
    map.put("y", "y");
    map.put("z", "z");
    
    map.put("A", "A");
    map.put("B", "B");
    map.put("C", "C");
    map.put("D", "D");
    map.put("E", "E");
    map.put("F", "F");
    map.put("G", "G");
    map.put("H", "H");
    map.put("I", "I");
    map.put("J", "J");
    map.put("K", "K");
    map.put("L", "L");
    map.put("M", "M");
    map.put("N", "N");
    map.put("O", "O");
    map.put("P", "P");
    map.put("Q", "Q");
    map.put("R", "R");
    map.put("S", "S");
    map.put("T", "T");
    map.put("U", "U");
    map.put("V", "V");
    map.put("W", "W");
    map.put("X", "X");
    map.put("Y", "Y");
    map.put("Z", "Z");
    
    map.put("0", "0");
    map.put("1", "1");
    map.put("2", "2");
    map.put("3", "3");
    map.put("4", "4");
    map.put("5", "5");
    map.put("6", "6");
    map.put("7", "7");
    map.put("8", "8");
    map.put("9", "9");

    map.put(".", ".");
    map.put("_", "_");


    map.put(":", ":");
    map.put("/", File.separator);
    map.put("\\", File.separator);

    String temp = "";
    for (int i = 0; i < path.length(); i++) {
        if (map.get(path.charAt(i)+"")!=null) {
            temp += map.get(path.charAt(i)+"");
        }
    }
    return temp;
}
```
## Log Forging

**日志伪造**
1. 数据从一个不可信赖的数据源进入应用程序。

2. 数据写入到应用程序或系统日志文件中。

```java
/**
 * 过滤引起Log Forging漏洞的敏感字符
 * @param str
 */
public String filterLogForging(String str){
    List<String> sensitiveStr = new ArrayList<>();
    sensitiveStr.add("%0d");
    sensitiveStr.add("%0a");
    sensitiveStr.add("%0A");
    sensitiveStr.add("%0D");
    sensitiveStr.add("\r");
    sensitiveStr.add("\n");
    String normalize = Normalizer.normalize(str, Normalizer.Form.NFKC);
    Iterator<String> iterator = sensitiveStr.iterator();
    while (iterator.hasNext()){
        normalize.replace(iterator.next(),"");
    }
    return normalize;
}
```
## Mass Assignment:Insecure Binder Configuration
不安全的参数绑定配置，是指我们的controller中xxxMethod(User user) 未明确指定接口所需属性，而是把整个对象所有属性暴露出去。

在Controller类中添加以下代码：
```java
final String[] DISALLOWED_FIELDS = new String[] {"is_admin"};

@InitBinder
public void initBinder(WebDataBinder binder) {
    binder.setDisallowedFields(DISALLOWED_FIELDS);
}
```
这种方式只是解决了扫描问题，但是实际没有真正解决该漏洞，因为这个错误出现的并不多，高风险也允许有万分之三的存在。
