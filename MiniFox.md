---
title: MiniFox-模拟远程接口返回的测试桩
date: 2016-09-28 23:36
tags: 
- Java
- Tool
- 测试
---

*这两天应测试要求写了一个模拟远程接口返回的测试桩。*
代码在这里：https://github.com/Yealove/MiniFox

## 为什么做
原来有一个测试桩，功能是：根据访问的URL返回一个固定报文（配置在某文件中）。
但当一个URL业务对应多个返回的测试场景时需要手动修改文件，测试觉得太麻烦，浪费很多时间在替换文件上面。
故需要一个能按请求报文内容返回不同报文的测试桩。

## 做什么
与测试讨论后得出需求如下：
1. 根据请求报文中节点的值返回不同的固定报文
2. 规则可以配置一个或多个节点
3. 值可以配置枚举

## 怎么做
基本需求确认后，就开始考虑怎么实现了。

### 请求报文获取
请求是用Socket监听端口，只处理POST请求，当有请求时，通过Socket的输入流获取请求信息。
```Java
ServerSocket server = new ServerSocket(port);
while (true) {
    Fox fox = new Fox(server.accept());
    fox.start();
}
```

因为是通过流获取信息，所以这里需要我们自己解析HTTP请求信息：
```
Http请求格式:
<request-line>
<headers>
<blank line>
[<request-body>]
```
开始还不知道怎么判断请求结束，纠结了一阵，后来才发现POST请求的`headers`中有`Content-Length`表示`request-body`的长度，只要按照这个值获取就行了。这个也就是我需要的请求报文了。


### 配置解析与匹配
之前那个测试桩的配置是用json写的，如果还是用json写现在的复杂配置的话，则看起来和写起来不是特别的方便，所以按照需求自己定制了配置规则如下：
```
#url配置
/busi/sub
#值的匹配方式 word-整词匹配(默认)|contain-包含匹配(配置值包含实际值)|regex-正则匹配
_match                   ^contain
#默认返回文件名
_default                 ^default.txt
#规则(节点=值,多个以英文都会分隔)^返回文件名
node1=123,node2=234      ^file1.txt
node2=121                ^file2.xml
#url配置结束标志
_end
```
一行表示一条规则，对应一个固定返回。
匹配方式为三种，全匹配、包含和正则。
匹配方式中包含和正则都能实现值的枚举。

规则定义后开始考虑文件解析与存储，最后存储实现为：
一个URL对应一个URL配置对象，URL配置对象包含多个规则和对应返回文件名，一个规则对应多个节点和值的对应关系，即一个URL配置按关系解析为三层。
```Java
/**
 * Url配置
 */
private static class UrlConfig {
	List<Rule> rules = new ArrayList<Rule>();
	String defaultFileName;
	Match match = Match.WORD;
}

/**
 * 规则
 */
private static class Rule {
	List<Relation> relations = new ArrayList<Relation>();
	String fileName;
}

/**
 * 对应关系
 */
private static class Relation {
	String key;
	String value;
}
```

匹配实现为：根据访问URL获取配置对象，循环规则，根据规则中的节点获取请求报文值，如果对应关系值都匹配则返回当前规则文件名，否则继续循环规则。
```Java
    /**
     * 根据url和请求内容获取配置的文件名
     *
     * @param url
     * @param xml
     * @return
     */
    public static String getResultFileName(String url, String xml) {

        UrlConfig urlConfig = urlConfigMap.get(url);
        if (urlConfig == null) {
            throw new ConfigCheckedException("[" + url + "]未配置规则，请检查配置！");
        }

        //规则校验
        List<Rule> rules = urlConfig.rules;
        for (Rule rule : rules) {
            boolean isThisRule = true;
            //配置节点不在请求报文中的个数
            int noMatchCount = 0;

            //规则校验
            for (Relation relation : rule.relations) {
                //值不相等则当前规则校验失败，进行下个规则校验
                boolean isBreakFor = false;

                int startIndex = xml.indexOf("<" + relation.key + ">") + relation.key.length() + 2;
                int endIndex = xml.indexOf("</" + relation.key + ">");

                //有节点，且结束位置在开始位置之后，则有值
                if (startIndex > -1 && endIndex > -1 && endIndex > startIndex) {
                    //获取请求报文中节点的值
                    String value = xml.substring(startIndex, endIndex);
                    
                    //根据匹配规则进行值的校验
                    switch (urlConfig.match) {
                        case CONTAIN: {
                            if (!relation.value.contains(value)) {
                                isThisRule = false;
                                isBreakFor = true;
                            }
                            break;
                        }

                        case REGEX: {
                            if (!value.matches(relation.value)) {
                                isThisRule = false;
                                isBreakFor = true;
                            }
                            break;
                        }

                        default: {
                            if (!value.equals(relation.value)) {
                                isThisRule = false;
                                isBreakFor = true;
                            }
                            break;
                        }
                    }

                    //值不相等则当前规则校验失败，进行下个规则校验
                    if(isBreakFor) {
                        break;
                    }
                } else {
                    //如果没有此节点则计数加1
                    noMatchCount++;
                }
            }

            //配置节点存在且值匹配，则返回对应的文件名称
            if (noMatchCount < rule.relations.size() && isThisRule) {
                return rule.fileName;
            }
        }

        if (urlConfig.defaultFileName == null || "".equals(urlConfig.defaultFileName.trim())) {
            throw new ConfigCheckedException("[" + url + "]未配置默认返回");
        }

        return urlConfig.defaultFileName;
    }

```

### 固定报文返回
这个比较容易，就是根据获取到文件名读取文件，将文件内容通过Socket的输出流返回回去。
这里需要注意的是，返回也是需要按HTTP规范来返回的。
```
Http响应格式：
<status-line>
<headers>
<blank line>
[<response-body>]
```





