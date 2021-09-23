### 文件注释

![image-20210923110527569](https://gitee.com/huawesome/my-picture/raw/master/img/202109231105624.png)

![image-20210923110559808](https://gitee.com/huawesome/my-picture/raw/master/img/202109231105839.png)



### 方法注释

![image-20210923113546678](https://gitee.com/huawesome/my-picture/raw/master/img/202109231135714.png)

![image-20210923113603853](https://gitee.com/huawesome/my-picture/raw/master/img/202109231136891.png)

```sh
groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {if(i == 0) result += '* @param ' + params[i] + ' ' + ((i < params.size() - 1) ? '\\n' : '');else result += ' * @param ' + params[i] + ' ' + ((i < params.size() - 1) ? '\\n' : '')}; return result", methodParameters())
```

### 参考

- [IDEA自定义类注释和方法注释（自定义groovyScript方法实现多行参数注释）](https://www.cnblogs.com/Neil-learning/p/13169717.html)

