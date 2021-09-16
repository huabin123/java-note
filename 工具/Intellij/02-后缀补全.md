```java
package com.huabin.idea;

import com.sun.org.apache.xpath.internal.functions.FuncDoclocation;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

/**
 * @author: huabin
 * @date: 2021/9/7 5:48 下午
 */
public class PostfixCompletionTests {

    private Object object;

    @Test
    public void testPostfix(){
        // for 循环 list.for list.fori list.forr
        List<String> list = new ArrayList<>();

        for (String s : list) {

        }

        for (int i = 0; i < list.size(); i++) {

        }

        for (int i = list.size() - 1; i >= 0; i--) {

        }

        // 判空（非空）,null .notnull/.nn
        Object o = new Object();
        if (o == null) {

        }
        if (o != null) {

        }
        if (o != null) {

        }

        // boolean取反  flag.if  flag.not.if
        boolean flag = false;
        if (flag) {

        }
        if (!flag) {

        }

        // 定义变量 .new .var .val .field
        new PostfixCompletionTests();
        Object o1 = new Object();
//        lombok.val o2 = new Object();
        object = new Object();

        // 格式化字符串 "hello %s".format
        String.format("hello %s", "world");

        // 同步锁 .synchronized
        Object lock = new Object();
        synchronized (lock) {

        }

        // 异常捕获 .try
        try {
            shitMayHappen();
        } catch (Exception e) {
            e.printStackTrace();
        }

        // 强制转换 .cast castvar(用这个)
        Object fund = new Object();
        FuncDoclocation fund1 = (FuncDoclocation) fund;
        FuncDoclocation funcDoclocation = (FuncDoclocation) fund;

        // 输出 .sout .soutv .souf .serr
        String out = "this is msg";
        System.out.println(out);
        System.out.println("out = " + out);
        System.out.printf("", out);
        System.err.println(out);

        // 抛出异常与返回 .throw .return
        return out;
        throw new RuntimeException();

        /*
        自定义的方法
        */
        List<fund1> fund1s =new ArrayList();


    }


    public void shitMayHappen() throws Exception{
        throw new RuntimeException("with error");
    }
}

```

> 自定义postfix的方法

![image-20210907191523335](/Users/apple/Library/Application Support/typora-user-images/image-20210907191523335.png)

