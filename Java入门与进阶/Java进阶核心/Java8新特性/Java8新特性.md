## Lambda表达式

### Lambda表达式解决的问题

Lambda表达式的本质是作为函数式接口的实例。

用匿名实现类写的都可以换成Lambda表达式。

 Lambda 是一个匿名函数，我们可以把 Lambda 表达式理解为是一段可以传递的代码（将代码 像数据一样进行传递）。可以写出更简洁、更 灵活的代码。作为一种更紧凑的代码风格，使 Java的语言表达能力得到了提升。

- 从匿名类到 Lambda 的转换

  ```java
  Runnable r1 = new Runnable() {
    @Override
    public void run() {
      System.out.println("我爱北京天安门");
    }
  };
  ```

  ```java
  Runnable r2 = () -> System.out.println("我爱北京故宫");
  ```

- 匿名内部类传参到Lambda表达式的转换

  ```java
  Comparator<Integer> com1 = new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
          return Integer.compare(o1,o2);
      }
  };
  
  int compare1 = com1.compare(12,21);
  System.out.println(compare1);
  ```

  ```java
  Comparator<Integer> com2 = (o1,o2) -> Integer.compare(o1,o2);
  
  int compare2 = com2.compare(32,21);
  System.out.println(compare2);
  ```

  





### 语法

- 箭头操作符左侧：指定了 Lambda 表达式需要的参数列表

- 箭头操作符右侧： Lambda 表达式要执行的功能



### 函数式接口

如果一个接口中只实现了一个抽象方法，则此接口就是函数式接口。

#### Java内置四大函数式接口

![image-20210910102454419](/Users/apple/Library/Application Support/typora-user-images/image-20210910102454419.png)

### 方法引用与构造器引用

```java
//方法引用
Comparator<Integer> com3 = Integer :: compare;

int compare3 = com3.compare(32,21);
System.out.println(compare3);
```

## Stream

### 创建流

```java
// 获取员工姓名长度大于3的员工的姓名。
List<Employee> employees = EmployeeData.getEmployees();
Stream<String> namesStream = employees.stream().map(Employee::getName);
namesStream.filter(name -> name.length() > 3).forEach(System.out::println);
```

### Stream的中间操作

#### 映射

- map

```java
// map(Function f)——接收一个函数作为参数，将元素转换成其他形式或提取信息，该函数会被应用到每个元素上，并将其映射成一个新的元素。
List<String> list = Arrays.asList("aa", "bb", "cc", "dd");
list.stream().map(str -> str.toUpperCase()).forEach(System.out::println);
```

- flatmap

相当于把[1,2,3,[4,5,6]]一步拆成[1,2,3,4,5,6]，两层for循环

```java
//flatMap(Function f)——接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。
Stream<Character> characterStream = list.stream().flatMap(StreamAPITest1::fromStringToStream);
characterStream.forEach(System.out::println);
```

### Stream的终止操作

#### 收集

```java
List<Employee> employees = EmployeeData.getEmployees();
List<Employee> employeeList = employees.stream().filter(e -> e.getSalary() > 6000).collect(Collectors.toList());
```



