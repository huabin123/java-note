###### Spring-IOC<br />**huabin**<br />*(c)2021. 版权所有*

[TOC]



# 什么是IOC和DI

IOC（inversion of control）意思是反转控制，什么是反转控制，常见说法是把`对象的创建都交给IOC`了，程序员从我们平常代码中大量的NEW代码解脱出来。

用了IOC之后就伴随着DI（depend injection）意思是依赖注入，什么是依赖注入呢，就是Spring会把我们需要用到的依赖打包准备好，我们开箱即用。

# 如何实现自己的IOC

目标：`把用户需要用到的对象实例化封装到一个容器中，提供容器公共的访问方法供开发人员使用`

步骤：

1. 创建自定义的注解（带有我们自定义注解的类才进行对象的实例化和装配）；
2. 需用读取用户配置文件（用户可自定义那些包需要进行扫描bean的装配）；
3. 扫描文件进行类加载（根据用户的配置扫描对应的包加载对应包里面的calss放到一个容器里）；
4. 把扫描出来的class中带有自定义注解的类和属性分别进行实例化和属性注入；
5. 把实例化好的对象放到一个容器里并提供调用方法；

根据以上思路，我们整理出来需要使用的技术有注解相关技术、配置文件加载技术、类加载技术、反射技术。

# 简单实现一个IOC

## 前期准备

### 新建application.properties配置ioc.bean.scan属性，指定扫描那个文件目录下的类

![img-202109231443960](https://gitee.com/huawesome/my-picture/raw/master/img/202109251256687.png)

### 定义MyComponent注解，标注哪些类需要进行IOC管理

```java
/**
 * 自定义注解，标注哪些类需要进行IOC管理
 *
 * @author: huabin
 * @date: 2021/9/23 上午10:06
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface MyComponent {

    String value() default "";

}
```



### 定义MyAutowrite注解，标注哪些属性进行依赖注入

```java
/**
 * 自定义注解，标注哪些属性进行依赖注入
 *
 * @author: huabin
 * @date: 2021/9/23 上午10:10
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
public @interface MyAutowried {

    String value() default "";

}
```



### 读取Properties文件（读取ioc.bean.scan的值）、类名首字母转小写工具方法（bean的名称默认是类名首字母小写）

```java
public class IocUtil {

    /**
     * 功能简述：根据配置文件名加载配置文件
     * @param fileName
     * @return
     */
    public static Properties getPropertyByName(String fileName) throws IOException {
        InputStream is = null;
        Properties pro = null;

        try {
            is = IocUtil.class.getClassLoader().getResourceAsStream(fileName);
            pro = new Properties();
            pro.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            return pro;
        }

    }

    /**
     * 功能简述：首字母转小写
     * @param name
     * @return
     */
    public static String toLowercaseIndex(String name){
        if (!StringUtils.isEmpty(name)) {
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append(name.substring(0, 1).toLowerCase());
            stringBuilder.append(name.substring(1, name.length()));
            return stringBuilder.toString();
        }
        return null;
    }
}
```



### 定义需要进行IOC关联的类UserDao、UserService

```java
@MyComponent
public class UserDao {
    public void findUser(String userName){
        System.out.println("找到一个名字叫："+userName);
    }
}
```

```java
@MyComponent
public class UserService {

    @MyAutowried
    private UserDao userDao;

    public void findUser(String userName){
        userDao.findUser(userName);
    }

}

```



## 定位资源

读取application.properties文件的ioc.bean.scan属性，得到需要扫描的文件夹

```java
/**
     * 功能简述：定位需要进行实例化Java类
     * @param key
     * @return
     */
    private String getBeanScanPath(String key) throws IOException {
        Properties properties = IocUtil.getPropertyByName("application.properties");
        return properties.get(key).toString();
    }
```



## 加载类

根据ioc加载需要实例化的对象，保存到classNameSet集合里

```java
/**
     * 功能简述：加载JavaClass，后期根据class的全路径实例化bean
     * @param packageName 包名
     * @return
     */
    public void loadBeanClass(String packageName){
        // 路径替换
        String filePath = packageName.replace(".", "/");
        URL url = this.getClass().getClassLoader().getResource(filePath);

        // 得到根文件夹
        File rootFile = new File(url.getFile());

        // 遍历所有文件夹
        if (rootFile != null) {
            for (File file:
                 rootFile.listFiles()) {
                if (file.isDirectory()) {
                    // 如果是文件夹则递归继续往下扫描
                    loadBeanClass(packageName+"."+file.getName());
                } else {
                    if (file.getName().indexOf(".class")>0) {
                        // 保存class类名
                        classNameSet.add(packageName + "." + file.getName().replace(".class", ""));
                    }
                }
            }
        }


    }
```



## 实例化

实例化对象保存到beanMap容器

```java
private void registerBean() throws Exception {
        if (!CollectionUtils.isEmpty(classNameSet)) {
            for (String className : classNameSet) {
                // 实例化对象放入beanMap
                Class<?> clazz = Class.forName(className);
                MyComponent myComponent = clazz.getAnnotation(MyComponent.class);
                if (myComponent == null) {
                    continue;
                }
                // 定义bean key名称
                String beanname = (StringUtils.isEmpty(myComponent.value())) ? IocUtil.toLowercaseIndex(clazz.getSimpleName()) : myComponent.value();
                beanMap.put(beanname, clazz.newInstance());
            }
        }
    }
```



## 依赖注入

对已经实例化的对象进行属性注入

```java
/**
     * 功能简述：对已经实例化的对象进行属性注入
     * @param o
     * @return
     */
    private void doInjection(Object o) throws Exception {
        Field[] fields = o.getClass().getDeclaredFields();
        if (ArrayUtil.isNotEmpty(fields)) {
            for (Field file : fields) {
                MyAutowried autowried = file.getAnnotation(MyAutowried.class);
                if (autowried != null) {
                    // 得到beanName 验证该Bean是否已经实例化了
                    String beanName = (StringUtils.isEmpty(autowried.value())) ? IocUtil.toLowercaseIndex(file.getType().getSimpleName()) : autowried.value();

                    if (!beanMap.containsKey(beanName)) {
                        Class<?> clazz = file.getType();
                        beanMap.put(beanName, clazz.newInstance());
                    }

                    // 调用对象set方法注入属性
                    file.setAccessible(true);
                    file.set(o, beanMap.get(beanName));

                    // 递归当前实例化的对象的属性注入
                    doInjection(beanMap.get(beanName));
                }
            }
        }
    }
```



## 代码总览

```java
package com.hua;

import cn.hutool.core.map.MapUtil;
import cn.hutool.core.util.ArrayUtil;
import com.hua.anotation.MyAutowried;
import com.hua.anotation.MyComponent;
import com.hua.exception.NotFountBeanException;
import com.hua.util.IocUtil;
import com.sun.tools.javac.util.ArrayUtils;
import org.springframework.util.CollectionUtils;
import org.springframework.util.StringUtils;

import java.io.File;
import java.io.IOException;
import java.lang.reflect.Field;
import java.net.URL;
import java.util.HashSet;
import java.util.Map;
import java.util.Properties;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 自己实现一个简单地IOC容器
 *
 * @author: huabin
 * @date: 2021/9/23 上午10:03
 */
public class MyApplicationContext {

    // 存储需要实例化的对象class全路径名
    private Set<String> classNameSet = new HashSet<String>();

    // 存储实例化的对象Class全路径名
    private Map<String,Object> beanMap = new ConcurrentHashMap<String, Object>();

    public MyApplicationContext(){
        try {
            init();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public Object getBean(String name) throws NotFountBeanException {
        return beanMap.get(name);
    }

    public void init() throws Exception {
        //1、定位资源
        String beanScanPath = getBeanScanPath("ioc.bean.scan");
        //2、加载需要实例化的Class
        loadBeanClass(beanScanPath);
        //3、实例化bean
        registerBean();
        //4、注入bean的属性
        dependenceInjection();

    }

    private void dependenceInjection() throws Exception {
        if (MapUtil.isEmpty(beanMap)) return;
        for (Object o : beanMap.values()) {
            doInjection(o);
        }
    }

    /**
     * 功能简述：定位需要进行实例化Java类
     * @param key
     * @return
     */
    private String getBeanScanPath(String key) throws IOException {
        Properties properties = IocUtil.getPropertyByName("application.properties");
        return properties.get(key).toString();
    }


    /**
     * 功能简述：加载JavaClass，后期根据class的全路径实例化bean
     * @param packageName 包名
     * @return
     */
    public void loadBeanClass(String packageName){
        // 路径替换
        String filePath = packageName.replace(".", "/");
        URL url = this.getClass().getClassLoader().getResource(filePath);

        // 得到根文件夹
        File rootFile = new File(url.getFile());

        // 遍历所有文件夹
        if (rootFile != null) {
            for (File file:
                 rootFile.listFiles()) {
                if (file.isDirectory()) {
                    // 如果是文件夹则递归继续往下扫描
                    loadBeanClass(packageName+"."+file.getName());
                } else {
                    if (file.getName().indexOf(".class")>0) {
                        // 保存class类名
                        classNameSet.add(packageName + "." + file.getName().replace(".class", ""));
                    }
                }
            }
        }


    }

    private void registerBean() throws Exception {
        if (!CollectionUtils.isEmpty(classNameSet)) {
            for (String className : classNameSet) {
                // 实例化对象放入beanMap
                Class<?> clazz = Class.forName(className);
                MyComponent myComponent = clazz.getAnnotation(MyComponent.class);
                if (myComponent == null) {
                    continue;
                }
                // 定义bean key名称
                String beanname = (StringUtils.isEmpty(myComponent.value())) ? IocUtil.toLowercaseIndex(clazz.getSimpleName()) : myComponent.value();
                beanMap.put(beanname, clazz.newInstance());
            }
        }
    }

    /**
     * 功能简述：对已经实例化的对象进行属性注入
     * @param o
     * @return
     */
    private void doInjection(Object o) throws Exception {
        Field[] fields = o.getClass().getDeclaredFields();
        if (ArrayUtil.isNotEmpty(fields)) {
            for (Field file : fields) {
                MyAutowried autowried = file.getAnnotation(MyAutowried.class);
                if (autowried != null) {
                    // 得到beanName 验证该Bean是否已经实例化了
                    String beanName = (StringUtils.isEmpty(autowried.value())) ? IocUtil.toLowercaseIndex(file.getType().getSimpleName()) : autowried.value();

                    if (!beanMap.containsKey(beanName)) {
                        Class<?> clazz = file.getType();
                        beanMap.put(beanName, clazz.newInstance());
                    }

                    // 调用对象set方法注入属性
                    file.setAccessible(true);
                    file.set(o, beanMap.get(beanName));

                    // 递归当前实例化的对象的属性注入
                    doInjection(beanMap.get(beanName));
                }
            }
        }
    }
}

```

# 测试

```java
/**
 * 测试自定义IOC
 *
 * @author: huabin
 * @date: 2021/9/23 下午2:17
 * @Version 1.0
 */
public class IocTest {

    public static void main(String[] args) throws Exception {
        MyApplicationContext myApplicationContext = new MyApplicationContext();
        UserService userService = (UserService)myApplicationContext.getBean("userService");
        userService.findUser("张三");
    }
    
}
```







# 源码地址

https://github.com/huabin123/java-code/tree/master/java-springboot

# 参考

[什么是IOC，如何用代码实现Spring IOC](https://zhuanlan.zhihu.com/p/58622371)

###### 
