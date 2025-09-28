# Java 8+新特性的深度思考 - 函数式编程的革命

## 引言

Java 8的发布是Java语言发展史上最重要的里程碑之一。它不仅仅是一次版本更新，而是一场深刻的编程范式革命。Lambda表达式、Stream API、Optional类等新特性的引入，让Java这个传统的面向对象语言拥抱了函数式编程的思想。

这场革命并非偶然，而是软件工程发展的必然趋势。在多核处理器普及、大数据处理需求增长的背景下，传统的命令式编程面临着前所未有的挑战。Java 8+的新特性为我们提供了更优雅、更高效的解决方案。

本文将深入探讨Java 8+新特性背后的设计哲学，分析函数式编程如何改变我们的编程思维，以及这些新特性在实际开发中的最佳实践。

## 1. Lambda表达式 - 函数式编程的基石

### 1.1 Lambda表达式的本质

Lambda表达式不仅仅是一个语法糖，它代表了Java编程思维的根本转变：

```java
// 传统匿名内部类方式
Runnable runnable1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};

// Lambda表达式方式
Runnable runnable2 = () -> System.out.println("Hello World");

// 更复杂的Lambda表达式
Comparator<String> comparator = (String s1, String s2) -> {
    if (s1.length() != s2.length()) {
        return s1.length() - s2.length();
    }
    return s1.compareTo(s2);
};
```

**设计哲学**：Lambda表达式体现了"行为作为参数"的思想，将代码块作为一等公民来处理，这是函数式编程的核心特征。

### 1.2 Lambda表达式的实现原理

```java
@FunctionalInterface
interface MyFunctionalInterface {
    int calculate(int a, int b);
}

public class LambdaImplementation {
    public static void main(String[] args) {
        // Lambda表达式会被编译器转换为invokedynamic指令
        MyFunctionalInterface adder = (a, b) -> a + b;

        // 这实际上在运行时会创建一个实现了MyFunctionalInterface的实例
        int result = adder.calculate(5, 3); // 输出8

        // 传统方式 vs Lambda方式的字节码对比
        // 传统方式：编译器生成匿名内部类
        // Lambda方式：使用invokedynamic动态生成实现类
    }
}

// Lambda表达式的底层实现机制
class LambdaMechanism {
    // Lambda表达式在运行时的实际处理过程
    // 1. 编译器生成invokedynamic指令
    // 2. 运行时通过LambdaMetafactory生成实现类
    // 3. 实现类缓存以提升性能

    // 性能优势：
    // - 避免了匿名内部类的类加载开销
    // - 减少了内存占用
    // - 支持延迟绑定
}
```

### 1.3 Lambda表达式的最佳实践

```java
public class LambdaBestPractices {

    // 1. 保持Lambda表达式简洁
    public void goodLambdaExample() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // 好的实践：简洁明了
        names.forEach(name -> System.out.println(name));

        // 避免过于复杂的Lambda表达式
        names.forEach(name -> {
            // 如果逻辑复杂，提取为方法
            String processedName = name.toUpperCase();
            System.out.println(processedName);
        });
    }

    // 2. 使用方法引用提高可读性
    public void methodReferences() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // Lambda表达式
        names.forEach(name -> System.out.println(name));

        // 方法引用 - 更简洁
        names.forEach(System.out::println);

        // 构造器引用
        Supplier<List<String>> listSupplier = ArrayList::new;

        // 静态方法引用
        Function<String, Integer> stringToInt = Integer::parseInt;

        // 实例方法引用
        BiPredicate<String, String> equals = String::equals;
    }

    // 3. 避免Lambda表达式中的副作用
    public void avoidSideEffects() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        List<Integer> result = new ArrayList<>();

        // 坏的做法：Lambda表达式中有副作用
        numbers.forEach(n -> {
            if (n % 2 == 0) {
                result.add(n); // 修改外部状态
            }
        });

        // 好的做法：使用Stream API
        result = numbers.stream()
                       .filter(n -> n % 2 == 0)
                       .collect(Collectors.toList());
    }
}
```

## 2. Stream API - 数据处理的革命

### 2.1 Stream API的设计哲学

Stream API引入了"函数式数据处理"的范式，彻底改变了Java中集合处理的模式：

```java
public class StreamAPIDesign {
    public void streamPhilosophy() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

        // 传统命令式编程
        List<String> filteredNames = new ArrayList<>();
        for (String name : names) {
            if (name.length() > 4) {
                filteredNames.add(name.toUpperCase());
            }
        }
        Collections.sort(filteredNames);

        // Stream API函数式编程
        List<String> result = names.stream()
            .filter(name -> name.length() > 4)
            .map(String::toUpperCase)
            .sorted()
            .collect(Collectors.toList());

        // 设计思想对比：
        // 命令式：描述"如何做"
        // 函数式：描述"做什么"
    }
}
```

**核心特性**：
- **声明式编程**：关注"做什么"而非"如何做"
- **链式操作**：流畅的API设计
- **惰性求值**：中间操作不会立即执行
- **内部迭代**：由Stream API控制迭代过程

### 2.2 Stream操作深度解析

```java
public class StreamOperationsDeep {

    // 1. 中间操作（Intermediate Operations）
    public void intermediateOperations() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

        // filter - 过滤元素
        Stream<String> filtered = names.stream()
            .filter(name -> name.startsWith("A"));

        // map - 转换元素
        Stream<Integer> lengths = names.stream()
            .map(String::length);

        // flatMap - 扁平化处理
        List<List<String>> nestedLists = Arrays.asList(
            Arrays.asList("A", "B"),
            Arrays.asList("C", "D")
        );

        Stream<String> flattened = nestedLists.stream()
            .flatMap(List::stream);

        // sorted - 排序
        Stream<String> sorted = names.stream()
            .sorted(Comparator.reverseOrder());

        // distinct - 去重
        Stream<String> distinct = names.stream()
            .distinct();

        // peek - 调试操作
        Stream<String> peeked = names.stream()
            .peek(System.out::println);
    }

    // 2. 终端操作（Terminal Operations）
    public void terminalOperations() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

        // collect - 收集结果
        List<String> collected = names.stream()
            .filter(name -> name.length() > 4)
            .collect(Collectors.toList());

        // forEach - 遍历元素
        names.stream()
            .forEach(System.out::println);

        // reduce - 归约操作
        Optional<String> concatenated = names.stream()
            .reduce((s1, s2) -> s1 + s2);

        // 查找操作
        Optional<String> first = names.stream()
            .findFirst();

        Optional<String> any = names.stream()
            .findAny();

        // 匹配操作
        boolean allMatch = names.stream()
            .allMatch(name -> name.length() > 3);

        boolean anyMatch = names.stream()
            .anyMatch(name -> name.startsWith("A"));

        boolean noneMatch = names.stream()
            .noneMatch(name -> name.isEmpty());

        // 统计操作
        long count = names.stream()
            .count();

        Optional<String> max = names.stream()
            .max(Comparator.naturalOrder());

        Optional<String> min = names.stream()
            .min(Comparator.naturalOrder());
    }

    // 3. 并行流（Parallel Stream）
    public void parallelStream() {
        List<Integer> numbers = IntStream.range(1, 1000000)
            .boxed()
            .collect(Collectors.toList());

        // 顺序流
        long sequentialTime = System.currentTimeMillis();
        long sequentialSum = numbers.stream()
            .mapToLong(Integer::longValue)
            .sum();
        long sequentialDuration = System.currentTimeMillis() - sequentialTime;

        // 并行流
        long parallelTime = System.currentTimeMillis();
        long parallelSum = numbers.parallelStream()
            .mapToLong(Integer::longValue)
            .sum();
        long parallelDuration = System.currentTimeMillis() - parallelTime;

        // 并行流的注意事项：
        // 1. 数据量大时才有性能优势
        // 2. 操作复杂时才能体现并行优势
        // 3. 避免共享可变状态
        // 4. 考虑线程安全问题
    }
}
```

### 2.3 高级Stream操作

```java
public class AdvancedStreamOperations {

    // 1. 自定义收集器
    public void customCollector() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

        // 自定义收集器：将字符串连接为逗号分隔的形式
        String joined = names.stream()
            .collect(Collector.of(
                () -> new StringBuilder(),
                (sb, name) -> {
                    if (sb.length() > 0) {
                        sb.append(", ");
                    }
                    sb.append(name);
                },
                (sb1, sb2) -> sb1.append(", ").append(sb2),
                StringBuilder::toString
            ));

        System.out.println("Joined names: " + joined);
    }

    // 2. 分组与分区
    public void groupingAndPartitioning() {
        List<Person> people = Arrays.asList(
            new Person("Alice", 25, "F"),
            new Person("Bob", 30, "M"),
            new Person("Charlie", 25, "M"),
            new Person("David", 35, "M")
        );

        // 按年龄分组
        Map<Integer, List<Person>> byAge = people.stream()
            .collect(Collectors.groupingBy(Person::getAge));

        // 按性别分组，并统计数量
        Map<String, Long> countByGender = people.stream()
            .collect(Collectors.groupingBy(
                Person::getGender,
                Collectors.counting()
            ));

        // 多级分组
        Map<String, Map<Integer, List<Person>>> multiLevel = people.stream()
            .collect(Collectors.groupingBy(
                Person::getGender,
                Collectors.groupingBy(Person::getAge)
            ));

        // 分区（分为两组）
        Map<Boolean, List<Person>> byAdult = people.stream()
            .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));
    }

    // 3. Stream的数学操作
    public void mathematicalOperations() {
        IntStream numbers = IntStream.range(1, 10);

        // 统计信息
        IntSummaryStatistics stats = numbers.summaryStatistics();
        System.out.println("Count: " + stats.getCount());
        System.out.println("Sum: " + stats.getSum());
        System.out.println("Average: " + stats.getAverage());
        System.out.println("Min: " + stats.getMin());
        System.out.println("Max: " + stats.getMax());

        // 数值流的特殊操作
        DoubleStream doubles = DoubleStream.generate(Math::random)
            .limit(100);

        double average = doubles.average()
            .orElse(Double.NaN);

        // 数值范围
        IntStream.range(1, 100)
            .forEach(System.out::println);

        IntStream.rangeClosed(1, 100)
            .forEach(System.out::println);
    }

    // 4. 无限流
    public void infiniteStreams() {
        // 生成无限流
        Stream<Integer> naturalNumbers = Stream.iterate(0, n -> n + 1);

        // 生成斐波那契数列
        Stream<Integer> fibonacci = Stream.iterate(
            new int[]{0, 1},
            fib -> new int[]{fib[1], fib[0] + fib[1]}
        ).map(fib -> fib[0]);

        // 限制无限流
        fibonacci.limit(20)
            .forEach(System.out::println);

        // 生成随机数流
        Random random = new Random();
        IntStream randomNumbers = random.ints(10, 1, 100);

        randomNumbers.forEach(System.out::println);
    }
}

class Person {
    private String name;
    private int age;
    private String gender;

    public Person(String name, int age, String gender) {
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getGender() { return gender; }
}
```

## 3. Optional - 空指针的艺术

### 3.1 Optional的设计理念

Optional类的引入彻底改变了Java中处理null值的方式：

```java
public class OptionalDesign {

    // 传统null检查方式
    public String getUserNameTraditional(User user) {
        if (user != null) {
            Address address = user.getAddress();
            if (address != null) {
                return address.getStreet();
            }
        }
        return "Unknown";
    }

    // Optional方式
    public String getUserNameOptional(User user) {
        return Optional.ofNullable(user)
            .map(User::getAddress)
            .map(Address::getStreet)
            .orElse("Unknown");
    }

    // Optional的设计哲学：
    // 1. 明确表达"可能为空"的意图
    // 2. 强制调用者处理空值情况
    // 3. 提供流畅的API来处理空值
    // 4. 避免空指针异常
    }
}

class User {
    private Address address;
    public Address getAddress() { return address; }
}

class Address {
    private String street;
    public String getStreet() { return street; }
}
```

### 3.2 Optional的深入应用

```java
public class OptionalDeepDive {

    // 1. 创建Optional
    public void creatingOptional() {
        // 创建空的Optional
        Optional<String> empty = Optional.empty();

        // 创建非空的Optional
        Optional<String> nonEmpty = Optional.of("Hello");

        // 创建可能为空的Optional
        Optional<String> nullable = Optional.ofNullable(getNullableString());
    }

    // 2. Optional的转换操作
    public void optionalTransformations() {
        Optional<String> optional = Optional.of("Hello");

        // map - 转换值
        Optional<Integer> length = optional.map(String::length);

        // flatMap - 扁平化转换
        Optional<String> result = optional.flatMap(value ->
            Optional.of(value.toUpperCase()));

        // filter - 过滤值
        Optional<String> filtered = optional.filter(s -> s.length() > 3);
    }

    // 3. Optional的获取操作
    public void optionalRetrieval() {
        Optional<String> optional = Optional.of("Hello");

        // orElse - 提供默认值
        String value1 = optional.orElse("Default");

        // orElseGet - 延迟提供默认值
        String value2 = optional.orElseGet(() -> {
            // 复杂的计算逻辑
            return "Default";
        });

        // orElseThrow - 抛出异常
        String value3 = optional.orElseThrow(() ->
            new IllegalArgumentException("Value is empty"));

        // or - Java 9+ 提供新的Optional
        Optional<String> value4 = optional.or(() -> Optional.of("Default"));

        // ifPresent - 如果存在则执行操作
        optional.ifPresent(System.out::println);

        // ifPresentOrElse - Java 9+ 如果存在或不存在时执行操作
        optional.ifPresentOrElse(
            System.out::println,
            () -> System.out.println("Value is empty")
        );
    }

    // 4. Optional的流式操作
    public void optionalStreamOperations() {
        List<Optional<String>> optionals = Arrays.asList(
            Optional.of("Alice"),
            Optional.empty(),
            Optional.of("Bob"),
            Optional.empty()
        );

        // 过滤掉空的Optional
        List<String> nonEmptyValues = optionals.stream()
            .flatMap(Optional::stream)
            .collect(Collectors.toList());

        // Java 8的方式
        List<String> java8Way = optionals.stream()
            .filter(Optional::isPresent)
            .map(Optional::get)
            .collect(Collectors.toList());

        System.out.println("Non-empty values: " + nonEmptyValues);
    }

    private String getNullableString() {
        return Math.random() > 0.5 ? "Hello" : null;
    }
}
```

### 3.3 Optional的最佳实践

```java
public class OptionalBestPractices {

    // 1. 不要用Optional作为字段类型
    public class BadExample {
        // 不好的做法：不要用Optional作为字段
        private Optional<String> name; // 避免这样做

        // 好的做法：使用普通的类型
        private String name2;
    }

    // 2. 不要用Optional作为方法参数
    public void badMethod(Optional<String> param) {
        // 不好的做法：不要用Optional作为参数
        // 应该使用方法重载
    }

    // 好的做法：使用方法重载
    public void goodMethod(String param) {
        // 处理非空参数
    }

    public void goodMethod() {
        // 处理空参数情况
    }

    // 3. 在返回类型中使用Optional
    public Optional<String> findUserById(String id) {
        // 好的做法：明确表达可能返回空
        return Optional.ofNullable(findUserInDatabase(id));
    }

    // 4. 避免嵌套的Optional
    public Optional<String> avoidNestedOptional() {
        Optional<Optional<String>> nested = Optional.of(Optional.of("Hello"));

        // 不好的做法：嵌套Optional
        // Optional<Optional<String>> nested

        // 好的做法：使用flatMap
        Optional<String> flattened = nested.flatMap(opt -> opt);

        return flattened;
    }

    // 5. Optional与集合的结合使用
    public void optionalWithCollections() {
        List<User> users = Arrays.asList(
            new User("Alice"),
            new User("Bob"),
            null
        );

        // 好的做法：使用Optional处理集合中的空值
        List<String> names = users.stream()
            .map(Optional::ofNullable)
            .map(opt -> opt.map(User::getName).orElse("Unknown"))
            .collect(Collectors.toList());

        System.out.println("Names: " + names);
    }
}
```

## 4. CompletableFuture - 异步编程的新时代

### 4.1 CompletableFuture的设计理念

CompletableFuture代表了Java异步编程的重大进步：

```java
public class CompletableFutureDesign {

    // 传统异步编程方式
    public void traditionalAsync() {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        Future<String> future = executor.submit(() -> {
            Thread.sleep(1000);
            return "Result";
        });

        try {
            String result = future.get(); // 阻塞等待
            System.out.println(result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        executor.shutdown();
    }

    // CompletableFuture方式
    public void completableFutureAsync() {
        CompletableFuture.supplyAsync(() -> {
            // 异步执行的任务
            try {
                Thread.sleep(1000);
                return "Result";
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).thenAccept(result -> {
            // 非阻塞地处理结果
            System.out.println(result);
        });

        // 立即继续执行，不等待异步任务完成
        System.out.println("Main thread continues...");
    }

    // CompletableFuture的优势：
    // 1. 非阻塞的异步编程
    // 2. 链式操作
    // 3. 组合多个异步操作
    // 4. 异常处理机制
    // 5. 超时控制
}
```

### 4.2 CompletableFuture的深入应用

```java
public class CompletableFutureDeepDive {

    // 1. 基本的异步操作
    public void basicOperations() {
        // 运行异步任务
        CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
            System.out.println("Running async task");
        });

        // 异步返回结果
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
            return "Async result";
        });

        // 指定执行器
        ExecutorService executor = Executors.newFixedThreadPool(4);
        CompletableFuture<String> future3 = CompletableFuture.supplyAsync(() -> {
            return "Result with custom executor";
        }, executor);
    }

    // 2. 链式操作
    public void chainingOperations() {
        CompletableFuture.supplyAsync(() -> {
            return "Hello";
        }).thenApply(result -> {
            return result + " World";
        }).thenAccept(finalResult -> {
            System.out.println(finalResult);
        }).thenRun(() -> {
            System.out.println("Operation completed");
        });

        // thenApply - 转换结果
        // thenAccept - 消费结果
        // thenRun - 不关心结果，执行操作
    }

    // 3. 组合操作
    public void compositionOperations() {
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
                return "Result 1";
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(500);
                return "Result 2";
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        // thenCompose - 顺序组合
        CompletableFuture<String> composed = future1.thenCompose(result1 -> {
            return CompletableFuture.supplyAsync(() -> result1 + " + " + result2.join());
        });

        // thenCombine - 并行组合
        CompletableFuture<String> combined = future1.thenCombine(future2, (r1, r2) -> {
            return r1 + " + " + r2;
        });

        // allOf - 等待所有任务完成
        CompletableFuture<Void> all = CompletableFuture.allOf(future1, future2);

        // anyOf - 等待任一任务完成
        CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2);
    }

    // 4. 异常处理
    public void exceptionHandling() {
        CompletableFuture.supplyAsync(() -> {
            if (Math.random() > 0.5) {
                throw new RuntimeException("Something went wrong");
            }
            return "Success";
        }).exceptionally(ex -> {
            System.out.println("Exception occurred: " + ex.getMessage());
            return "Fallback value";
        }).thenAccept(result -> {
            System.out.println("Final result: " + result);
        });

        // handle - 无论成功或失败都执行
        CompletableFuture.supplyAsync(() -> {
            return "Success";
        }).handle((result, ex) -> {
            if (ex != null) {
                System.out.println("Exception: " + ex.getMessage());
                return "Error recovery";
            }
            return result;
        });

        // whenComplete - 类似于handle，但不转换结果
        CompletableFuture.supplyAsync(() -> {
            return "Success";
        }).whenComplete((result, ex) -> {
            if (ex != null) {
                System.out.println("Exception in whenComplete");
            } else {
                System.out.println("Success in whenComplete");
            }
        });
    }

    // 5. 超时控制
    public void timeoutControl() {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
                return "Result after delay";
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        // 设置超时
        CompletableFuture<String> withTimeout = future.orTimeout(1, TimeUnit.SECONDS);

        // 设置超时后的默认值
        CompletableFuture<String> withTimeoutDefault = future.completeOnTimeout(
            "Timeout default", 1, TimeUnit.SECONDS);

        try {
            String result = withTimeout.get();
            System.out.println("Result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            System.out.println("Timeout occurred");
        }
    }
}
```

## 5. Java 9+ 新特性的演进

### 5.1 Java 9 模块系统

```java
// module-info.java - 模块声明文件
module com.example.myapp {
    requires java.base;
    requires java.sql;
    requires spring.core;

    exports com.example.myapp.service;
    exports com.example.myapp.controller to spring.web;

    opens com.example.myapp.model to hibernate.core;

    uses com.example.myapp.spi.PluginService;
    provides com.example.myapp.spi.PluginService
        with com.example.myapp.impl.DefaultPluginService;
}

public class ModuleSystemDeep {

    // 模块化的优势：
    // 1. 强封装性
    // 2. 清晰的依赖关系
    // 3. 可靠的配置
    // 4. 安全的访问控制

    public void moduleBenefits() {
        // 模块化的设计理念：
        // - 显式依赖声明
        // - 强封装机制
        // - 模块间的访问控制
        // - 服务发现机制

        // 模块与传统的classpath对比：
        // - 模块化：编译时检查依赖，运行时确保一致性
        // - classpath：运行时才检查，容易出现依赖冲突
    }
}
```

### 5.2 Java 10 局部变量类型推断

```java
public class LocalVariableTypeInference {

    public void varExamples() {
        // 基本类型推断
        var number = 42;                  // int
        var decimal = 3.14;               // double
        var text = "Hello";              // String
        var flag = true;                 // boolean

        // 集合类型推断
        var names = List.of("Alice", "Bob", "Charlie");
        var ages = Set.of(25, 30, 35);
        var scores = Map.of("Math", 90, "English", 85);

        // 流操作中的使用
        var filtered = names.stream()
            .filter(name -> name.length() > 4)
            .collect(Collectors.toList());

        // 在try-with-resources中的使用
        try (var reader = new FileReader("test.txt")) {
            // 自动推断为FileReader
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // var的局限性
    public void varLimitations() {
        // 不能用于字段声明
        // private var name = "John"; // 编译错误

        // 不能用于方法参数
        // public void method(var param) {} // 编译错误

        // 不能用于方法返回类型
        // public var getName() { return name; } // 编译错误

        // Lambda表达式参数不能使用var（Java 10）
        // (var x, var y) -> x + y; // 编译错误

        // 需要显式类型的情况
        // var numbers = {1, 2, 3}; // 编译错误，需要显式类型
        var numbers = new int[]{1, 2, 3}; // 正确

        // 不能初始化为null
        // var value = null; // 编译错误，无法推断类型
    }
}
```

### 5.3 Java 11+ 新特性

```java
public class Java11PlusFeatures {

    // 1. Java 11 - HTTP Client
    public void httpClientExample() {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.example.com/data"))
            .GET()
            .build();

        CompletableFuture<String> response = client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
            .thenApply(HttpResponse::body);

        response.thenAccept(System.out::println);
    }

    // 2. Java 11 - String新增方法
    public void stringNewMethods() {
        String text = "  Hello World  ";

        // isBlank() - 检查是否为空或只包含空白字符
        boolean isEmpty = text.isBlank();

        // strip() - 删除首尾空白字符（支持Unicode）
        String stripped = text.strip();

        // stripLeading() - 删除开头空白字符
        String leadingStripped = text.stripLeading();

        // stripTrailing() - 删除结尾空白字符
        String trailingStripped = text.stripTrailing();

        // lines() - 按行分割字符串
        String multiLine = "Line 1\nLine 2\nLine 3";
        Stream<String> lines = multiLine.lines();

        // repeat() - 重复字符串
        String repeated = "Hello ".repeat(3);
    }

    // 3. Java 14 - Record类型
    public record PersonRecord(String name, int age, String email) {}

    public void recordExample() {
        // 自动生成构造器、getter、equals、hashCode、toString
        var person = new PersonRecord("Alice", 25, "alice@example.com");

        System.out.println(person.name());     // 访问器方法
        System.out.println(person.age());
        System.out.println(person.toString());

        // Record的特性：
        // 1. 不可变性
        // 2. 自动生成方法
        // 3. 简洁的数据载体
        // 4. 适用于数据传输对象
    }

    // 4. Java 17 - Sealed Classes
    public sealed class Shape permits Circle, Rectangle, Triangle {
        public abstract double area();
    }

    public final class Circle extends Shape {
        private final double radius;

        public Circle(double radius) {
            this.radius = radius;
        }

        @Override
        public double area() {
            return Math.PI * radius * radius;
        }
    }

    public final class Rectangle extends Shape {
        private final double width;
        private final double height;

        public Rectangle(double width, double height) {
            this.width = width;
            this.height = height;
        }

        @Override
        public double area() {
            return width * height;
        }
    }

    public final class Triangle extends Shape {
        private final double base;
        private final double height;

        public Triangle(double base, double height) {
            this.base = base;
            this.height = height;
        }

        @Override
        public double area() {
            return 0.5 * base * height;
        }
    }

    public void sealedClassExample() {
        // Sealed Classes的优势：
        // 1. 限制继承层次
        // 2. 编译时检查
        // 3. 更好的模式匹配支持
        // 4. 更安全的代码设计

        Shape shape = new Circle(5.0);
        double area = shape.area();
        System.out.println("Area: " + area);
    }
}
```

## 6. 函数式编程的最佳实践

### 6.1 函数式编程的设计模式

```java
public class FunctionalDesignPatterns {

    // 1. 函数式构建器模式
    public class FunctionalBuilder<T> {
        private final Supplier<T> supplier;
        private final List<Consumer<T>> consumers = new ArrayList<>();

        public FunctionalBuilder(Supplier<T> supplier) {
            this.supplier = supplier;
        }

        public FunctionalBuilder<T> with(Consumer<T> consumer) {
            consumers.add(consumer);
            return this;
        }

        public T build() {
            T instance = supplier.get();
            consumers.forEach(consumer -> consumer.accept(instance));
            return instance;
        }
    }

    public void functionalBuilderExample() {
        User user = new FunctionalBuilder<>(User::new)
            .with(u -> u.setName("Alice"))
            .with(u -> u.setAge(25))
            .with(u -> u.setEmail("alice@example.com"))
            .build();
    }

    // 2. 函数式策略模式
    @FunctionalInterface
    public interface Strategy<T, R> {
        R apply(T input);
    }

    public class StrategyContext<T, R> {
        private final Map<String, Strategy<T, R>> strategies = new HashMap<>();

        public void addStrategy(String name, Strategy<T, R> strategy) {
            strategies.put(name, strategy);
        }

        public R executeStrategy(String name, T input) {
            Strategy<T, R> strategy = strategies.get(name);
            if (strategy == null) {
                throw new IllegalArgumentException("Unknown strategy: " + name);
            }
            return strategy.apply(input);
        }
    }

    public void strategyPatternExample() {
        StrategyContext<Integer, String> context = new StrategyContext<>();

        context.addStrategy("upper", n -> n.toString().toUpperCase());
        context.addStrategy("lower", n -> n.toString().toLowerCase());
        context.addStrategy("binary", n -> Integer.toBinaryString(n));

        String result = context.executeStrategy("binary", 42);
        System.out.println("Binary representation: " + result);
    }

    // 3. 函数式观察者模式
    public class FunctionalObservable<T> {
        private final List<Consumer<T>> observers = new ArrayList<>();

        public void addObserver(Consumer<T> observer) {
            observers.add(observer);
        }

        public void removeObserver(Consumer<T> observer) {
            observers.remove(observer);
        }

        public void notifyObservers(T value) {
            observers.forEach(observer -> observer.accept(value));
        }
    }

    public void observerPatternExample() {
        FunctionalObservable<String> observable = new FunctionalObservable<>();

        Consumer<String> logger = message -> System.out.println("Logger: " + message);
        Consumer<String> notifier = message -> System.out.println("Notifier: " + message);

        observable.addObserver(logger);
        observable.addObserver(notifier);

        observable.notifyObservers("Hello Observers!");
    }
}
```

### 6.2 函数式编程的性能考虑

```java
public class FunctionalPerformance {

    // 1. 避免过度装箱/拆箱
    public void boxingPerformance() {
        // 好的做法：使用原始类型流
        IntStream.range(0, 1000000)
            .filter(n -> n % 2 == 0)
            .map(n -> n * n)
            .sum();

        // 避免的做法：使用对象流导致装箱
        Stream<Integer> stream = IntStream.range(0, 1000000)
            .boxed();

        long sum = stream.filter(n -> n % 2 == 0)
            .map(n -> n * n)
            .mapToLong(Long::longValue)
            .sum();
    }

    // 2. 合理使用并行流
    public void parallelStreamPerformance() {
        List<Integer> numbers = IntStream.range(0, 1000000)
            .boxed()
            .collect(Collectors.toList());

        // 简单操作，并行流可能更慢
        long start = System.currentTimeMillis();
        long sum1 = numbers.stream()
            .mapToInt(Integer::intValue)
            .sum();
        long end = System.currentTimeMillis();
        System.out.println("Sequential: " + (end - start) + "ms");

        start = System.currentTimeMillis();
        long sum2 = numbers.parallelStream()
            .mapToInt(Integer::intValue)
            .sum();
        end = System.currentTimeMillis();
        System.out.println("Parallel: " + (end - start) + "ms");
    }

    // 3. 避免Lambda表达式的重复创建
    public void lambdaCreationOptimization() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // 好的做法：重复使用Lambda表达式
        Predicate<String> longName = name -> name.length() > 4;

        List<String> filtered1 = names.stream()
            .filter(longName)
            .collect(Collectors.toList());

        List<String> filtered2 = names.stream()
            .filter(longName)
            .collect(Collectors.toList());

        // 避免的做法：每次创建新的Lambda表达式
        List<String> filtered3 = names.stream()
            .filter(name -> name.length() > 4)
            .collect(Collectors.toList());
    }

    // 4. 使用方法引用提高性能
    public void methodReferencePerformance() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // 方法引用通常比Lambda表达式更高效
        names.forEach(System.out::println);

        // 相当于
        names.forEach(name -> System.out.println(name));
    }
}
```

## 7. 函数式编程的未来发展

### 7.1 Project Valhalla 和 Value Types

```java
// 未来可能出现的Value Types特性
public class ValueTypesFuture {

    // Value Classes - 避免对象头开销
    value class Point {
        final int x;
        final int y;

        Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    // Primitive Types的泛型支持
    public void primitiveGenerics() {
        // 未来可能支持的语法
        List<int> numbers = List.of(1, 2, 3, 4, 5);

        // 避免了装箱开销
        int sum = numbers.stream()
            .reduce(0, (a, b) -> a + b);
    }

    // 模式匹配的进一步增强
    public void patternMatchingEnhancement() {
        Object obj = "Hello";

        // 未来可能支持的模式匹配
        switch (obj) {
            case String s when s.length() > 5:
                System.out.println("Long string: " + s);
                break;
            case Integer i when i > 100:
                System.out.println("Large integer: " + i);
                break;
            case Point(x, y) when x == y:
                System.out.println("Point on diagonal: (" + x + ", " + y + ")");
                break;
            default:
                System.out.println("Unknown object");
        }
    }
}
```

### 7.2 虚拟线程（Project Loom）

```java
public class VirtualThreads {

    // 传统线程模型
    public void traditionalThreads() {
        ExecutorService executor = Executors.newFixedThreadPool(1000);

        for (int i = 0; i < 10000; i++) {
            final int taskId = i;
            executor.submit(() -> {
                try {
                    Thread.sleep(1000);
                    System.out.println("Task " + taskId + " completed");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
    }

    // 虚拟线程模型（Java 19+）
    public void virtualThreads() {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 10000; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    try {
                        Thread.sleep(1000);
                        System.out.println("Virtual Task " + taskId + " completed");
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
        }

        // 虚拟线程的优势：
        // 1. 轻量级，可以创建数百万个
        // 2. 由JVM调度，不依赖操作系统线程
        // 3. 适用于I/O密集型应用
        // 4. 与现有代码兼容
    }
}
```

## 8. 总结与展望

Java 8+的函数式编程革命不仅仅是语言特性的增强，更是编程思维的重大转变。通过Lambda表达式、Stream API、Optional、CompletableFuture等新特性，Java开发者能够以更简洁、更优雅的方式编写代码。

### 8.1 核心收获

1. **函数式思维**：从"如何做"到"做什么"的转变
2. **不可变性**：减少副作用，提高代码可预测性
3. **声明式编程**：更简洁、更易读的代码
4. **并行处理**：充分利用多核处理器的计算能力
5. **异步编程**：更高效地处理并发任务

### 8.2 最佳实践总结

1. **合理使用Lambda表达式**：保持简洁，避免过度复杂
2. **掌握Stream API**：熟练使用各种流操作提高开发效率
3. **善用Optional**：优雅地处理空值，避免NullPointerException
4. **拥抱异步编程**：使用CompletableFuture提高系统吞吐量
5. **关注性能**：避免不必要的装箱开销，合理使用并行流

### 8.3 未来发展

Java的函数式编程之路仍在继续：

1. **更好的类型推断**：更智能的类型推导
2. **模式匹配**：更优雅的数据解构和处理
3. **值类型**：减少对象开销，提高性能
4. **虚拟线程**：彻底改变并发编程模式
5. **更丰富的函数式特性**：持续吸收其他语言的优秀特性

**Java的函数式编程革命告诉我们，优秀的编程语言不是固步自封的，而是不断吸收新的思想和技术，在保持稳定性的同时拥抱变化。作为Java开发者，我们需要持续学习，不断进步，才能在这个快速变化的技术世界中立于不败之地。**

函数式编程不仅仅是一种编程范式，更是一种思维方式。它让我们从不同的角度思考问题，找到更优雅、更高效的解决方案。在这个充满挑战和机遇的时代，掌握函数式编程将成为Java开发者的核心竞争力。

---

*这篇文章深入探讨了Java 8+新特性背后的函数式编程思想，从Lambda表达式到Stream API，从Optional到CompletableFuture，全面分析了函数式编程如何改变Java的开发方式。通过大量的代码示例和最佳实践，帮助读者深入理解函数式编程的精髓，并在实际项目中灵活应用这些新特性。*