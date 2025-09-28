# 设计模式深度解析：行为型模式的哲学

## 引言

行为型设计模式关注对象之间的通信和职责分配，它们定义了对象之间如何进行交互以及如何分配职责。这些模式描述了算法和对象间职责的分配，让对象之间的交互更加灵活和可维护。在这篇文章中，我将深入探讨11种核心的行为型模式，揭示它们的设计哲学、实现技巧和实际应用。

## 1. 责任链模式：传递的艺术

### 1.1 模式本质与设计哲学

责任链模式让多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止：

```java
// 抽象处理器
public abstract class Handler {
    protected Handler nextHandler;

    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }

    public abstract void handleRequest(Request request);
}

// 具体处理器A
public class ConcreteHandlerA extends Handler {
    @Override
    public void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE_A) {
            System.out.println("处理器A处理请求: " + request.getName());
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }
}

// 具体处理器B
public class ConcreteHandlerB extends Handler {
    @Override
    public void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE_B) {
            System.out.println("处理器B处理请求: " + request.getName());
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }
}

// 请求类
public class Request {
    private String name;
    private RequestType type;

    public Request(String name, RequestType type) {
        this.name = name;
        this.type = type;
    }

    public String getName() { return name; }
    public RequestType getType() { return type; }
}

public enum RequestType {
    TYPE_A, TYPE_B, TYPE_C
}
```

**使用示例**：
```java
Handler handlerA = new ConcreteHandlerA();
Handler handlerB = new ConcreteHandlerB();
Handler handlerC = new ConcreteHandlerC();

handlerA.setNextHandler(handlerB);
handlerB.setNextHandler(handlerC);

handlerA.handleRequest(new Request("请求1", RequestType.TYPE_A));
handlerA.handleRequest(new Request("请求2", RequestType.TYPE_B));
handlerA.handleRequest(new Request("请求3", RequestType.TYPE_C));
```

**设计思考**：责任链模式体现了"职责分离"的思想，每个处理器只关心自己能处理的请求，将无法处理的请求传递给下一个处理器。

### 1.2 实际应用场景

#### 日志级别处理

```java
public abstract class Logger {
    protected Logger nextLogger;
    protected LogLevel level;

    public Logger(LogLevel level) {
        this.level = level;
    }

    public void setNextLogger(Logger nextLogger) {
        this.nextLogger = nextLogger;
    }

    public void logMessage(LogLevel level, String message) {
        if (this.level.ordinal() <= level.ordinal()) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.logMessage(level, message);
        }
    }

    protected abstract void write(String message);
}

public enum LogLevel {
    DEBUG, INFO, WARNING, ERROR
}

public class DebugLogger extends Logger {
    public DebugLogger() {
        super(LogLevel.DEBUG);
    }

    @Override
    protected void write(String message) {
        System.out.println("DEBUG: " + message);
    }
}

public class ErrorLogger extends Logger {
    public ErrorLogger() {
        super(LogLevel.ERROR);
    }

    @Override
    protected void write(String message) {
        System.out.println("ERROR: " + message);
    }
}
```

#### Web框架的过滤器链

```java
public interface Filter {
    void doFilter(Request request, Response response, FilterChain chain);
}

public class AuthenticationFilter implements Filter {
    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        if (isAuthenticated(request)) {
            chain.doFilter(request, response);
        } else {
            response.sendError(401, "Unauthorized");
        }
    }

    private boolean isAuthenticated(Request request) {
        // 认证逻辑
        return true;
    }
}

public class AuthorizationFilter implements Filter {
    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        if (isAuthorized(request)) {
            chain.doFilter(request, response);
        } else {
            response.sendError(403, "Forbidden");
        }
    }

    private boolean isAuthorized(Request request) {
        // 授权逻辑
        return true;
    }
}

public class FilterChain implements Filter {
    private List<Filter> filters = new ArrayList<>();
    private int currentFilter = 0;

    public void addFilter(Filter filter) {
        filters.add(filter);
    }

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        if (currentFilter < filters.size()) {
            Filter filter = filters.get(currentFilter++);
            filter.doFilter(request, response, this);
        }
    }
}
```

**哲学思考**：责任链模式体现了"链条式处理"的思想，通过将处理器组织成链状结构，实现了请求的灵活传递和处理。

## 2. 命令模式：请求的封装

### 2.1 模式核心思想

命令模式将请求封装成对象，从而让你可以用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作：

```java
// 命令接口
public interface Command {
    void execute();
    void undo();
}

// 具体命令A
public class ConcreteCommandA implements Command {
    private Receiver receiver;

    public ConcreteCommandA(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.actionA();
    }

    @Override
    public void undo() {
        receiver.undoActionA();
    }
}

// 具体命令B
public class ConcreteCommandB implements Command {
    private Receiver receiver;

    public ConcreteCommandB(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.actionB();
    }

    @Override
    public void undo() {
        receiver.undoActionB();
    }
}

// 接收者
public class Receiver {
    public void actionA() {
        System.out.println("执行操作A");
    }

    public void undoActionA() {
        System.out.println("撤销操作A");
    }

    public void actionB() {
        System.out.println("执行操作B");
    }

    public void undoActionB() {
        System.out.println("撤销操作B");
    }
}

// 调用者
public class Invoker {
    private Command command;
    private Stack<Command> history = new Stack<>();

    public void setCommand(Command command) {
        this.command = command;
    }

    public void executeCommand() {
        command.execute();
        history.push(command);
    }

    public void undoCommand() {
        if (!history.isEmpty()) {
            Command lastCommand = history.pop();
            lastCommand.undo();
        }
    }
}
```

**使用示例**：
```java
Receiver receiver = new Receiver();
Command commandA = new ConcreteCommandA(receiver);
Command commandB = new ConcreteCommandB(receiver);

Invoker invoker = new Invoker();
invoker.setCommand(commandA);
invoker.executeCommand();

invoker.setCommand(commandB);
invoker.executeCommand();

invoker.undoCommand();
```

**设计智慧**：命令模式体现了"封装"的思想，它将请求封装成对象，使得请求可以被参数化、排队、记录和撤销。

### 2.2 实际应用场景

#### 文本编辑器

```java
public interface TextCommand {
    void execute();
    void undo();
}

public class InsertTextCommand implements TextCommand {
    private TextEditor editor;
    private String text;
    private int position;

    public InsertTextCommand(TextEditor editor, String text, int position) {
        this.editor = editor;
        this.text = text;
        this.position = position;
    }

    @Override
    public void execute() {
        editor.insertText(text, position);
    }

    @Override
    public void undo() {
        editor.deleteText(position, text.length());
    }
}

public class DeleteTextCommand implements TextCommand {
    private TextEditor editor;
    private int position;
    private String deletedText;

    public DeleteTextCommand(TextEditor editor, int position, int length) {
        this.editor = editor;
        this.position = position;
        this.deletedText = editor.getText(position, length);
    }

    @Override
    public void execute() {
        editor.deleteText(position, deletedText.length());
    }

    @Override
    public void undo() {
        editor.insertText(deletedText, position);
    }
}

public class TextEditor {
    private StringBuilder text = new StringBuilder();
    private Stack<TextCommand> history = new Stack<>();

    public void insertText(String text, int position) {
        this.text.insert(position, text);
    }

    public void deleteText(int position, int length) {
        this.text.delete(position, position + length);
    }

    public String getText(int position, int length) {
        return this.text.substring(position, position + length);
    }

    public void executeCommand(TextCommand command) {
        command.execute();
        history.push(command);
    }

    public void undo() {
        if (!history.isEmpty()) {
            TextCommand command = history.pop();
            command.undo();
        }
    }
}
```

#### 事务管理

```java
public interface TransactionCommand {
    void execute();
    void rollback();
}

public class UpdateAccountCommand implements TransactionCommand {
    private Account account;
    private double oldBalance;
    private double newBalance;

    public UpdateAccountCommand(Account account, double newBalance) {
        this.account = account;
        this.newBalance = newBalance;
        this.oldBalance = account.getBalance();
    }

    @Override
    public void execute() {
        account.setBalance(newBalance);
    }

    @Override
    public void rollback() {
        account.setBalance(oldBalance);
    }
}

public class TransactionManager {
    private List<TransactionCommand> commands = new ArrayList<>();

    public void addCommand(TransactionCommand command) {
        commands.add(command);
    }

    public void commit() {
        for (TransactionCommand command : commands) {
            command.execute();
        }
        commands.clear();
    }

    public void rollback() {
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).rollback();
        }
        commands.clear();
    }
}
```

**哲学思考**：命令模式体现了"可撤销操作"的思想，它通过将操作封装成对象，实现了操作的记录和撤销功能。

## 3. 解释器模式：语言的处理

### 3.1 模式概念与实现

解释器模式给定一种语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子：

```java
// 抽象表达式
public interface Expression {
    int interpret();
}

// 终结符表达式
public class NumberExpression implements Expression {
    private int number;

    public NumberExpression(int number) {
        this.number = number;
    }

    @Override
    public int interpret() {
        return number;
    }
}

// 非终结符表达式 - 加法
public class AddExpression implements Expression {
    private Expression left;
    private Expression right;

    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}

// 非终结符表达式 - 减法
public class SubtractExpression implements Expression {
    private Expression left;
    private Expression right;

    public SubtractExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() - right.interpret();
    }
}

// 非终结符表达式 - 乘法
public class MultiplyExpression implements Expression {
    private Expression left;
    private Expression right;

    public MultiplyExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() * right.interpret();
    }
}

// 上下文
public class Context {
    private Map<String, Integer> variables = new HashMap<>();

    public void setVariable(String name, int value) {
        variables.put(name, value);
    }

    public int getVariable(String name) {
        return variables.get(name);
    }
}

// 变量表达式
public class VariableExpression implements Expression {
    private String name;
    private Context context;

    public VariableExpression(String name, Context context) {
        this.name = name;
        this.context = context;
    }

    @Override
    public int interpret() {
        return context.getVariable(name);
    }
}
```

**使用示例**：
```java
// 表达式: (5 + 3) * 2
Expression five = new NumberExpression(5);
Expression three = new NumberExpression(3);
Expression add = new AddExpression(five, three);
Expression two = new NumberExpression(2);
Expression multiply = new MultiplyExpression(add, two);

int result = multiply.interpret();
System.out.println("结果: " + result); // 输出: 16
```

**设计智慧**：解释器模式体现了"语言处理"的思想，它通过定义语言的文法表示，实现了语言的解释和执行。

### 3.2 实际应用场景

#### 正则表达式解析

```java
public interface RegexExpression {
    boolean match(String text);
}

public class LiteralExpression implements RegexExpression {
    private String literal;

    public LiteralExpression(String literal) {
        this.literal = literal;
    }

    @Override
    public boolean match(String text) {
        return text.equals(literal);
    }
}

public class OrExpression implements RegexExpression {
    private RegexExpression left;
    private RegexExpression right;

    public OrExpression(RegexExpression left, RegexExpression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public boolean match(String text) {
        return left.match(text) || right.match(text);
    }
}

public class ConcatExpression implements RegexExpression {
    private RegexExpression left;
    private RegexExpression right;

    public ConcatExpression(RegexExpression left, RegexExpression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public boolean match(String text) {
        // 简化的连接匹配
        return text.contains(left.toString()) && text.contains(right.toString());
    }
}
```

#### SQL解析器

```java
public interface SqlExpression {
    String evaluate(Map<String, Object> data);
}

public class ColumnExpression implements SqlExpression {
    private String columnName;

    public ColumnExpression(String columnName) {
        this.columnName = columnName;
    }

    @Override
    public String evaluate(Map<String, Object> data) {
        return String.valueOf(data.get(columnName));
    }
}

public class ComparisonExpression implements SqlExpression {
    private SqlExpression left;
    private SqlExpression right;
    private String operator;

    public ComparisonExpression(SqlExpression left, String operator, SqlExpression right) {
        this.left = left;
        this.operator = operator;
        this.right = right;
    }

    @Override
    public String evaluate(Map<String, Object> data) {
        String leftValue = left.evaluate(data);
        String rightValue = right.evaluate(data);

        switch (operator) {
            case "=":
                return leftValue.equals(rightValue) ? "true" : "false";
            case ">":
                return Double.parseDouble(leftValue) > Double.parseDouble(rightValue) ? "true" : "false";
            case "<":
                return Double.parseDouble(leftValue) < Double.parseDouble(rightValue) ? "true" : "false";
            default:
                return "false";
        }
    }
}
```

**哲学思考**：解释器模式体现了"领域特定语言(DSL)"的思想，它让开发者能够定义和处理特定领域的语言。

## 4. 迭代器模式：遍历的艺术

### 4.1 模式核心概念

迭代器模式提供一种方法顺序访问一个聚合对象中各个元素，而又不暴露该对象的内部表示：

```java
// 迭代器接口
public interface Iterator<T> {
    boolean hasNext();
    T next();
}

// 聚合接口
public interface Aggregate<T> {
    Iterator<T> createIterator();
}

// 具体聚合
public class ConcreteAggregate<T> implements Aggregate<T> {
    private List<T> items = new ArrayList<>();

    public void addItem(T item) {
        items.add(item);
    }

    @Override
    public Iterator<T> createIterator() {
        return new ConcreteIterator(items);
    }
}

// 具体迭代器
public class ConcreteIterator<T> implements Iterator<T> {
    private List<T> items;
    private int position = 0;

    public ConcreteIterator(List<T> items) {
        this.items = items;
    }

    @Override
    public boolean hasNext() {
        return position < items.size();
    }

    @Override
    public T next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        return items.get(position++);
    }
}
```

**使用示例**：
```java
ConcreteAggregate<String> aggregate = new ConcreteAggregate<>();
aggregate.addItem("元素1");
aggregate.addItem("元素2");
aggregate.addItem("元素3");

Iterator<String> iterator = aggregate.createIterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    System.out.println(item);
}
```

**设计智慧**：迭代器模式体现了"访问分离"的思想，它将数据的访问方式从数据结构中分离出来，使得访问方式可以独立变化。

### 4.2 现代Java应用

#### Java集合框架

```java
// Java集合框架广泛使用了迭代器模式
List<String> list = Arrays.asList("A", "B", "C");
Iterator<String> iterator = list.iterator();

// 使用增强for循环（内部使用迭代器）
for (String item : list) {
    System.out.println(item);
}

// 使用Java 8的forEach
list.forEach(System.out::println);

// 自定义迭代器
public class TreeNodeIterator implements Iterator<TreeNode> {
    private Queue<TreeNode> queue = new LinkedList<>();

    public TreeNodeIterator(TreeNode root) {
        if (root != null) {
            queue.add(root);
        }
    }

    @Override
    public boolean hasNext() {
        return !queue.isEmpty();
    }

    @Override
    public TreeNode next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        TreeNode node = queue.poll();
        if (node.getLeft() != null) {
            queue.add(node.getLeft());
        }
        if (node.getRight() != null) {
            queue.add(node.getRight());
        }
        return node;
    }
}
```

#### 数据库结果集

```java
public class ResultSetIterator<T> implements Iterator<T> {
    private ResultSet resultSet;
    private RowMapper<T> rowMapper;
    private boolean hasNext;

    public ResultSetIterator(ResultSet resultSet, RowMapper<T> rowMapper) {
        this.resultSet = resultSet;
        this.rowMapper = rowMapper;
        try {
            this.hasNext = resultSet.next();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public boolean hasNext() {
        return hasNext;
    }

    @Override
    public T next() {
        if (!hasNext) {
            throw new NoSuchElementException();
        }
        try {
            T item = rowMapper.mapRow(resultSet);
            hasNext = resultSet.next();
            return item;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**哲学思考**：迭代器模式体现了"统一访问"的思想，它为不同的数据结构提供了统一的访问方式。

## 5. 中介者模式：协调的中心

### 5.1 模式概念与设计

中介者模式用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互：

```java
// 中介者接口
public interface Mediator {
    void sendMessage(String message, Colleague colleague);
    void registerColleague(Colleague colleague);
}

// 具体中介者
public class ConcreteMediator implements Mediator {
    private List<Colleague> colleagues = new ArrayList<>();

    @Override
    public void registerColleague(Colleague colleague) {
        colleagues.add(colleague);
    }

    @Override
    public void sendMessage(String message, Colleague sender) {
        for (Colleague colleague : colleagues) {
            if (colleague != sender) {
                colleague.receiveMessage(message);
            }
        }
    }
}

// 同事类
public abstract class Colleague {
    protected Mediator mediator;
    protected String name;

    public Colleague(Mediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }

    public abstract void sendMessage(String message);
    public abstract void receiveMessage(String message);
}

// 具体同事A
public class ConcreteColleagueA extends Colleague {
    public ConcreteColleagueA(Mediator mediator, String name) {
        super(mediator, name);
    }

    @Override
    public void sendMessage(String message) {
        System.out.println(name + " 发送消息: " + message);
        mediator.sendMessage(message, this);
    }

    @Override
    public void receiveMessage(String message) {
        System.out.println(name + " 收到消息: " + message);
    }
}

// 具体同事B
public class ConcreteColleagueB extends Colleague {
    public ConcreteColleagueB(Mediator mediator, String name) {
        super(mediator, name);
    }

    @Override
    public void sendMessage(String message) {
        System.out.println(name + " 发送消息: " + message);
        mediator.sendMessage(message, this);
    }

    @Override
    public void receiveMessage(String message) {
        System.out.println(name + " 收到消息: " + message);
    }
}
```

**使用示例**：
```java
Mediator mediator = new ConcreteMediator();
Colleague colleagueA = new ConcreteColleagueA(mediator, "同事A");
Colleague colleagueB = new ConcreteColleagueB(mediator, "同事B");
Colleague colleagueC = new ConcreteColleagueC(mediator, "同事C");

mediator.registerColleague(colleagueA);
mediator.registerColleague(colleagueB);
mediator.registerColleague(colleagueC);

colleagueA.sendMessage("大家好！");
colleagueB.sendMessage("收到消息！");
```

**设计智慧**：中介者模式体现了"集中控制"的思想，它通过中介者对象来协调多个对象的交互，减少了对象之间的直接耦合。

### 5.2 实际应用场景

#### 聊天室系统

```java
public interface ChatMediator {
    void sendMessage(String message, User user);
    void addUser(User user);
}

public class ChatRoom implements ChatMediator {
    private List<User> users = new ArrayList<>();

    @Override
    public void addUser(User user) {
        users.add(user);
    }

    @Override
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receive(message);
            }
        }
    }
}

public abstract class User {
    protected ChatMediator mediator;
    protected String name;

    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }

    public abstract void send(String message);
    public abstract void receive(String message);
}

public class ChatUser extends User {
    public ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
    }

    @Override
    public void send(String message) {
        System.out.println(name + " 发送: " + message);
        mediator.sendMessage(message, this);
    }

    @Override
    public void receive(String message) {
        System.out.println(name + " 收到: " + message);
    }
}
```

#### 航空管制系统

```java
public interface AirTrafficControlMediator {
    void requestLanding(Aircraft aircraft);
    void requestTakeoff(Aircraft aircraft);
    void acknowledgeLanding(Aircraft aircraft);
    void acknowledgeTakeoff(Aircraft aircraft);
}

public class Tower implements AirTrafficControlMediator {
    private List<Aircraft> aircrafts = new ArrayList<>();
    private boolean runwayAvailable = true;

    @Override
    public void requestLanding(Aircraft aircraft) {
        if (runwayAvailable) {
            runwayAvailable = false;
            System.out.println("塔台允许 " + aircraft.getName() + " 降落");
            aircraft.receiveClearance("允许降落");
        } else {
            System.out.println("塔台指示 " + aircraft.getName() + " 等待");
            aircraft.receiveClearance("等待降落");
        }
    }

    @Override
    public void requestTakeoff(Aircraft aircraft) {
        if (runwayAvailable) {
            runwayAvailable = false;
            System.out.println("塔台允许 " + aircraft.getName() + " 起飞");
            aircraft.receiveClearance("允许起飞");
        } else {
            System.out.println("塔台指示 " + aircraft.getName() + " 等待");
            aircraft.receiveClearance("等待起飞");
        }
    }

    @Override
    public void acknowledgeLanding(Aircraft aircraft) {
        runwayAvailable = true;
        System.out.println(aircraft.getName() + " 已降落，跑道可用");
    }

    @Override
    public void acknowledgeTakeoff(Aircraft aircraft) {
        runwayAvailable = true;
        System.out.println(aircraft.getName() + " 已起飞，跑道可用");
    }
}
```

**哲学思考**：中介者模式体现了"解耦"的思想，它通过中介者对象来管理对象之间的交互，避免了对象之间的直接依赖。

## 6. 备忘录模式：状态的保存

### 6.1 模式核心思想

备忘录模式在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态：

```java
// 备忘录
public class Memento {
    private String state;

    public Memento(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }
}

// 发起人
public class Originator {
    private String state;

    public void setState(String state) {
        this.state = state;
        System.out.println("状态设置为: " + state);
    }

    public String getState() {
        return state;
    }

    public Memento saveStateToMemento() {
        return new Memento(state);
    }

    public void getStateFromMemento(Memento memento) {
        state = memento.getState();
        System.out.println("状态恢复为: " + state);
    }
}

// 负责人
public class Caretaker {
    private List<Memento> mementoList = new ArrayList<>();

    public void addMemento(Memento memento) {
        mementoList.add(memento);
    }

    public Memento getMemento(int index) {
        return mementoList.get(index);
    }
}
```

**使用示例**：
```java
Originator originator = new Originator();
Caretaker caretaker = new Caretaker();

originator.setState("状态1");
originator.setState("状态2");
caretaker.addMemento(originator.saveStateToMemento());

originator.setState("状态3");
caretaker.addMemento(originator.saveStateToMemento());

originator.setState("状态4");

System.out.println("当前状态: " + originator.getState());
originator.getStateFromMemento(caretaker.getMemento(1));
System.out.println("恢复后状态: " + originator.getState());
```

**设计智慧**：备忘录模式体现了"状态管理"的思想，它通过备忘录对象来保存和恢复对象的状态，实现了撤销和恢复功能。

### 6.2 实际应用场景

#### 文本编辑器撤销

```java
public class TextDocument {
    private StringBuilder content = new StringBuilder();
    private Stack<TextMemento> history = new Stack<>();

    public void insertText(String text) {
        content.append(text);
    }

    public void deleteText(int start, int length) {
        content.delete(start, start + length);
    }

    public void save() {
        history.push(new TextMemento(content.toString()));
    }

    public void undo() {
        if (!history.isEmpty()) {
            TextMemento memento = history.pop();
            content = new StringBuilder(memento.getContent());
        }
    }

    public String getContent() {
        return content.toString();
    }

    private static class TextMemento {
        private final String content;

        public TextMemento(String content) {
            this.content = content;
        }

        public String getContent() {
            return content;
        }
    }
}
```

#### 游戏状态保存

```java
public class GameState {
    private int level;
    private int score;
    private int health;
    private List<String> inventory;

    public GameState(int level, int score, int health, List<String> inventory) {
        this.level = level;
        this.score = score;
        this.health = health;
        this.inventory = new ArrayList<>(inventory);
    }

    public GameStateMemento createMemento() {
        return new GameStateMemento(level, score, health, new ArrayList<>(inventory));
    }

    public void restoreFromMemento(GameStateMemento memento) {
        this.level = memento.getLevel();
        this.score = memento.getScore();
        this.health = memento.getHealth();
        this.inventory = new ArrayList<>(memento.getInventory());
    }

    public static class GameStateMemento {
        private final int level;
        private final int score;
        private final int health;
        private final List<String> inventory;

        public GameStateMemento(int level, int score, int health, List<String> inventory) {
            this.level = level;
            this.score = score;
            this.health = health;
            this.inventory = inventory;
        }

        public int getLevel() { return level; }
        public int getScore() { return score; }
        public int getHealth() { return health; }
        public List<String> getInventory() { return inventory; }
    }
}
```

**哲学思考**：备忘录模式体现了"状态快照"的思想，它通过保存对象的状态快照，实现了状态的保存和恢复功能。

## 7. 观察者模式：通知的机制

### 7.1 模式概念与实现

观察者模式定义了对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新：

```java
// 主题接口
public interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

// 观察者接口
public interface Observer {
    void update(String message);
}

// 具体主题
public class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String state;

    public void setState(String state) {
        this.state = state;
        notifyObservers();
    }

    public String getState() {
        return state;
    }

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(state);
        }
    }
}

// 具体观察者A
public class ConcreteObserverA implements Observer {
    @Override
    public void update(String message) {
        System.out.println("观察者A收到通知: " + message);
    }
}

// 具体观察者B
public class ConcreteObserverB implements Observer {
    @Override
    public void update(String message) {
        System.out.println("观察者B收到通知: " + message);
    }
}
```

**使用示例**：
```java
Subject subject = new ConcreteSubject();
Observer observerA = new ConcreteObserverA();
Observer observerB = new ConcreteObserverB();

subject.registerObserver(observerA);
subject.registerObserver(observerB);

subject.setState("新状态1");
subject.setState("新状态2");
```

**设计智慧**：观察者模式体现了"发布-订阅"的思想，它通过定义观察者和主题之间的依赖关系，实现了松耦合的通知机制。

### 7.2 实际应用场景

#### 事件处理系统

```java
public interface EventListener {
    void onEvent(Event event);
}

public class EventManager {
    private Map<String, List<EventListener>> listeners = new HashMap<>();

    public void addListener(String eventType, EventListener listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }

    public void removeListener(String eventType, EventListener listener) {
        listeners.getOrDefault(eventType, new ArrayList<>()).remove(listener);
    }

    public void fireEvent(String eventType, Object data) {
        Event event = new Event(eventType, data);
        List<EventListener> eventListeners = listeners.get(eventType);
        if (eventListeners != null) {
            for (EventListener listener : eventListeners) {
                listener.onEvent(event);
            }
        }
    }
}

public class Event {
    private String type;
    private Object data;

    public Event(String type, Object data) {
        this.type = type;
        this.data = data;
    }

    public String getType() { return type; }
    public Object getData() { return data; }
}
```

#### 股票价格通知

```java
public class Stock {
    private String symbol;
    private double price;
    private List<StockObserver> observers = new ArrayList<>();

    public Stock(String symbol, double price) {
        this.symbol = symbol;
        this.price = price;
    }

    public void setPrice(double price) {
        this.price = price;
        notifyObservers();
    }

    public void addObserver(StockObserver observer) {
        observers.add(observer);
    }

    public void removeObserver(StockObserver observer) {
        observers.remove(observer);
    }

    private void notifyObservers() {
        for (StockObserver observer : observers) {
            observer.onPriceChange(symbol, price);
        }
    }
}

public interface StockObserver {
    void onPriceChange(String symbol, double price);
}

public class StockHolder implements StockObserver {
    private String name;

    public StockHolder(String name) {
        this.name = name;
    }

    @Override
    public void onPriceChange(String symbol, double price) {
        System.out.println(name + ": " + symbol + " 价格变更为 " + price);
    }
}
```

**哲学思考**：观察者模式体现了"事件驱动"的思想，它是事件驱动编程的基础，广泛应用于GUI框架、消息系统等场景。

## 8. 状态模式：行为的切换

### 8.1 模式核心概念

状态模式允许一个对象在其内部状态改变时改变它的行为。对象看起来似乎修改了它的类：

```java
// 状态接口
public interface State {
    void handle(Context context);
}

// 具体状态A
public class ConcreteStateA implements State {
    @Override
    public void handle(Context context) {
        System.out.println("当前状态A");
        context.setState(new ConcreteStateB());
    }
}

// 具体状态B
public class ConcreteStateB implements State {
    @Override
    public void handle(Context context) {
        System.out.println("当前状态B");
        context.setState(new ConcreteStateA());
    }
}

// 上下文
public class Context {
    private State state;

    public Context(State state) {
        this.state = state;
    }

    public void setState(State state) {
        this.state = state;
    }

    public void request() {
        state.handle(this);
    }
}
```

**使用示例**：
```java
Context context = new Context(new ConcreteStateA());
context.request(); // 输出: 当前状态A
context.request(); // 输出: 当前状态B
context.request(); // 输出: 当前状态A
```

**设计智慧**：状态模式体现了"状态分离"的思想，它将对象的状态和行为分离，使得状态的变化能够自然地驱动行为的变化。

### 8.2 实际应用场景

#### 订单状态管理

```java
public interface OrderState {
    void confirmOrder(Order order);
    void shipOrder(Order order);
    void deliverOrder(Order order);
    void cancelOrder(Order order);
}

public class NewOrderState implements OrderState {
    @Override
    public void confirmOrder(Order order) {
        System.out.println("订单已确认");
        order.setState(new ConfirmedOrderState());
    }

    @Override
    public void shipOrder(Order order) {
        System.out.println("错误：订单未确认，无法发货");
    }

    @Override
    public void deliverOrder(Order order) {
        System.out.println("错误：订单未发货，无法送达");
    }

    @Override
    public void cancelOrder(Order order) {
        System.out.println("订单已取消");
        order.setState(new CancelledOrderState());
    }
}

public class ConfirmedOrderState implements OrderState {
    @Override
    public void confirmOrder(Order order) {
        System.out.println("错误：订单已确认");
    }

    @Override
    public void shipOrder(Order order) {
        System.out.println("订单已发货");
        order.setState(new ShippedOrderState());
    }

    @Override
    public void deliverOrder(Order order) {
        System.out.println("错误：订单未发货，无法送达");
    }

    @Override
    public void cancelOrder(Order order) {
        System.out.println("订单已取消");
        order.setState(new CancelledOrderState());
    }
}

public class Order {
    private OrderState state;
    private String orderId;

    public Order(String orderId) {
        this.orderId = orderId;
        this.state = new NewOrderState();
    }

    public void setState(OrderState state) {
        this.state = state;
    }

    public void confirmOrder() {
        state.confirmOrder(this);
    }

    public void shipOrder() {
        state.shipOrder(this);
    }

    public void deliverOrder() {
        state.deliverOrder(this);
    }

    public void cancelOrder() {
        state.cancelOrder(this);
    }
}
```

#### 文档编辑器

```java
public interface DocumentState {
    void open(Document document);
    void edit(Document document);
    void save(Document document);
    void close(Document document);
}

public class OpenState implements DocumentState {
    @Override
    public void open(Document document) {
        System.out.println("文档已打开");
    }

    @Override
    public void edit(Document document) {
        System.out.println("编辑文档");
        document.setState(new EditState());
    }

    @Override
    public void save(Document document) {
        System.out.println("保存文档");
    }

    @Override
    public void close(Document document) {
        System.out.println("关闭文档");
        document.setState(new ClosedState());
    }
}

public class EditState implements DocumentState {
    @Override
    public void open(Document document) {
        System.out.println("文档已打开");
    }

    @Override
    public void edit(Document document) {
        System.out.println("继续编辑文档");
    }

    @Override
    public void save(Document document) {
        System.out.println("保存文档");
        document.setState(new OpenState());
    }

    @Override
    public void close(Document document) {
        System.out.println("警告：文档未保存，确定关闭吗？");
    }
}
```

**哲学思考**：状态模式体现了"状态驱动"的思想，它将复杂的状态逻辑分散到不同的状态类中，使得代码更加清晰和可维护。

## 9. 策略模式：算法的封装

### 9.1 模式概念与设计

策略模式定义了算法家族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化不会影响到使用算法的客户：

```java
// 策略接口
public interface Strategy {
    int execute(int a, int b);
}

// 具体策略A：加法
public class AddStrategy implements Strategy {
    @Override
    public int execute(int a, int b) {
        return a + b;
    }
}

// 具体策略B：减法
public class SubtractStrategy implements Strategy {
    @Override
    public int execute(int a, int b) {
        return a - b;
    }
}

// 具体策略C：乘法
public class MultiplyStrategy implements Strategy {
    @Override
    public int execute(int a, int b) {
        return a * b;
    }
}

// 上下文
public class Context {
    private Strategy strategy;

    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public int executeStrategy(int a, int b) {
        return strategy.execute(a, b);
    }
}
```

**使用示例**：
```java
Context context = new Context(new AddStrategy());
System.out.println(context.executeStrategy(5, 3)); // 输出: 8

context.setStrategy(new MultiplyStrategy());
System.out.println(context.executeStrategy(5, 3)); // 输出: 15
```

**设计智慧**：策略模式体现了"算法分离"的思想，它将算法的定义和使用分离，使得算法可以独立变化。

### 9.2 实际应用场景

#### 支付策略

```java
public interface PaymentStrategy {
    boolean pay(double amount);
}

public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String name;
    private String cvv;

    public CreditCardPayment(String cardNumber, String name, String cvv) {
        this.cardNumber = cardNumber;
        this.name = name;
        this.cvv = cvv;
    }

    @Override
    public boolean pay(double amount) {
        System.out.println("使用信用卡支付: " + amount);
        return true;
    }
}

public class AlipayPayment implements PaymentStrategy {
    private String accountId;

    public AlipayPayment(String accountId) {
        this.accountId = accountId;
    }

    @Override
    public boolean pay(double amount) {
        System.out.println("使用支付宝支付: " + amount);
        return true;
    }
}

public class WechatPayment implements PaymentStrategy {
    private String openId;

    public WechatPayment(String openId) {
        this.openId = openId;
    }

    @Override
    public boolean pay(double amount) {
        System.out.println("使用微信支付: " + amount);
        return true;
    }
}

public class PaymentContext {
    private PaymentStrategy paymentStrategy;

    public PaymentContext(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public boolean processPayment(double amount) {
        return paymentStrategy.pay(amount);
    }
}
```

#### 排序策略

```java
public interface SortStrategy {
    void sort(int[] array);
}

public class BubbleSortStrategy implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("使用冒泡排序");
        // 冒泡排序实现
    }
}

public class QuickSortStrategy implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("使用快速排序");
        // 快速排序实现
    }
}

public class MergeSortStrategy implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("使用归并排序");
        // 归并排序实现
    }
}

public class SortContext {
    private SortStrategy sortStrategy;

    public SortContext(SortStrategy sortStrategy) {
        this.sortStrategy = sortStrategy;
    }

    public void setSortStrategy(SortStrategy sortStrategy) {
        this.sortStrategy = sortStrategy;
    }

    public void performSort(int[] array) {
        sortStrategy.sort(array);
    }
}
```

**哲学思考**：策略模式体现了"开闭原则"，它让算法可以独立变化，而不需要修改使用算法的客户端代码。

## 10. 模板方法模式：流程的骨架

### 10.1 模式核心思想

模板方法模式定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤：

```java
// 抽象类
public abstract class AbstractTemplate {
    // 模板方法
    public final void templateMethod() {
        step1();
        step2();
        step3();
        if (hook()) {
            step4();
        }
        step5();
    }

    // 基本方法
    private void step1() {
        System.out.println("步骤1：固定的操作");
    }

    private void step5() {
        System.out.println("步骤5：固定的操作");
    }

    // 抽象方法
    protected abstract void step2();
    protected abstract void step3();
    protected abstract void step4();

    // 钩子方法
    protected boolean hook() {
        return true;
    }
}

// 具体类A
public class ConcreteTemplateA extends AbstractTemplate {
    @Override
    protected void step2() {
        System.out.println("具体类A的步骤2");
    }

    @Override
    protected void step3() {
        System.out.println("具体类A的步骤3");
    }

    @Override
    protected void step4() {
        System.out.println("具体类A的步骤4");
    }

    @Override
    protected boolean hook() {
        return false;
    }
}

// 具体类B
public class ConcreteTemplateB extends AbstractTemplate {
    @Override
    protected void step2() {
        System.out.println("具体类B的步骤2");
    }

    @Override
    protected void step3() {
        System.out.println("具体类B的步骤3");
    }

    @Override
    protected void step4() {
        System.out.println("具体类B的步骤4");
    }
}
```

**使用示例**：
```java
AbstractTemplate templateA = new ConcreteTemplateA();
templateA.templateMethod();

AbstractTemplate templateB = new ConcreteTemplateB();
templateB.templateMethod();
```

**设计智慧**：模板方法模式体现了"流程控制"的思想，它通过定义算法的骨架，让子类可以实现具体的步骤。

### 10.2 实际应用场景

#### 数据处理流程

```java
public abstract class DataProcessor {
    // 模板方法
    public final void processData() {
        validateInput();
        loadData();
        transformData();
        saveData();
        cleanup();
    }

    // 基本方法
    private void validateInput() {
        System.out.println("验证输入数据");
    }

    private void cleanup() {
        System.out.println("清理资源");
    }

    // 抽象方法
    protected abstract void loadData();
    protected abstract void transformData();
    protected abstract void saveData();
}

public class CsvDataProcessor extends DataProcessor {
    @Override
    protected void loadData() {
        System.out.println("加载CSV数据");
    }

    @Override
    protected void transformData() {
        System.out.println("转换CSV数据");
    }

    @Override
    protected void saveData() {
        System.out.println("保存CSV数据");
    }
}

public class JsonDataProcessor extends DataProcessor {
    @Override
    protected void loadData() {
        System.out.println("加载JSON数据");
    }

    @Override
    protected void transformData() {
        System.out.println("转换JSON数据");
    }

    @Override
    protected void saveData() {
        System.out.println("保存JSON数据");
    }
}
```

#### HTTP请求处理

```java
public abstract class HttpRequestHandler {
    // 模板方法
    public final void handleRequest(HttpRequest request, HttpResponse response) {
        validateRequest(request);
        authenticate(request);
        authorize(request);
        processRequest(request, response);
        logRequest(request);
    }

    // 基本方法
    private void validateRequest(HttpRequest request) {
        System.out.println("验证HTTP请求");
    }

    private void authenticate(HttpRequest request) {
        System.out.println("认证用户");
    }

    private void authorize(HttpRequest request) {
        System.out.println("授权用户");
    }

    private void logRequest(HttpRequest request) {
        System.out.println("记录请求日志");
    }

    // 抽象方法
    protected abstract void processRequest(HttpRequest request, HttpResponse response);
}

public class GetRequestHandler extends HttpRequestHandler {
    @Override
    protected void processRequest(HttpRequest request, HttpResponse response) {
        System.out.println("处理GET请求");
    }
}

public class PostRequestHandler extends HttpRequestHandler {
    @Override
    protected void processRequest(HttpRequest request, HttpResponse response) {
        System.out.println("处理POST请求");
    }
}
```

**哲学思考**：模板方法模式体现了"代码复用"的思想，它通过定义算法的骨架，实现了代码的复用和扩展。

## 11. 访问者模式：操作的扩展

### 11.1 模式概念与实现

访问者模式表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作：

```java
// 访问者接口
public interface Visitor {
    void visit(ConcreteElementA element);
    void visit(ConcreteElementB element);
}

// 具体访问者A
public class ConcreteVisitorA implements Visitor {
    @Override
    public void visit(ConcreteElementA element) {
        System.out.println("访问者A访问元素A: " + element.operationA());
    }

    @Override
    public void visit(ConcreteElementB element) {
        System.out.println("访问者A访问元素B: " + element.operationB());
    }
}

// 具体访问者B
public class ConcreteVisitorB implements Visitor {
    @Override
    public void visit(ConcreteElementA element) {
        System.out.println("访问者B访问元素A: " + element.operationA());
    }

    @Override
    public void visit(ConcreteElementB element) {
        System.out.println("访问者B访问元素B: " + element.operationB());
    }
}

// 元素接口
public interface Element {
    void accept(Visitor visitor);
}

// 具体元素A
public class ConcreteElementA implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

    public String operationA() {
        return "元素A的操作";
    }
}

// 具体元素B
public class ConcreteElementB implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

    public String operationB() {
        return "元素B的操作";
    }
}

// 对象结构
public class ObjectStructure {
    private List<Element> elements = new ArrayList<>();

    public void addElement(Element element) {
        elements.add(element);
    }

    public void removeElement(Element element) {
        elements.remove(element);
    }

    public void accept(Visitor visitor) {
        for (Element element : elements) {
            element.accept(visitor);
        }
    }
}
```

**使用示例**：
```java
ObjectStructure structure = new ObjectStructure();
structure.addElement(new ConcreteElementA());
structure.addElement(new ConcreteElementB());

Visitor visitorA = new ConcreteVisitorA();
structure.accept(visitorA);

Visitor visitorB = new ConcreteVisitorB();
structure.accept(visitorB);
```

**设计智慧**：访问者模式体现了"操作分离"的思想，它将数据结构和操作分离，使得可以方便地添加新的操作。

### 11.2 实际应用场景

#### 文档格式转换

```java
public interface DocumentVisitor {
    void visit(TextElement element);
    void visit(ImageElement element);
    void visit(TableElement element);
}

public class HtmlExportVisitor implements DocumentVisitor {
    @Override
    public void visit(TextElement element) {
        System.out.println("<p>" + element.getText() + "</p>");
    }

    @Override
    public void visit(ImageElement element) {
        System.out.println("<img src='" + element.getSrc() + "' alt='" + element.getAlt() + "' />");
    }

    @Override
    public void visit(TableElement element) {
        System.out.println("<table>");
        for (String[] row : element.getRows()) {
            System.out.println("<tr>");
            for (String cell : row) {
                System.out.println("<td>" + cell + "</td>");
            }
            System.out.println("</tr>");
        }
        System.out.println("</table>");
    }
}

public class PdfExportVisitor implements DocumentVisitor {
    @Override
    public void visit(TextElement element) {
        System.out.println("PDF文本: " + element.getText());
    }

    @Override
    public void visit(ImageElement element) {
        System.out.println("PDF图片: " + element.getSrc());
    }

    @Override
    public void visit(TableElement element) {
        System.out.println("PDF表格: " + element.getRows().size() + "行");
    }
}
```

#### 代码分析工具

```java
public interface CodeElement {
    void accept(CodeVisitor visitor);
}

public class ClassElement implements CodeElement {
    private String name;
    private List<MethodElement> methods = new ArrayList<>();

    @Override
    public void accept(CodeVisitor visitor) {
        visitor.visit(this);
        for (MethodElement method : methods) {
            method.accept(visitor);
        }
    }

    // getters and setters
}

public class MethodElement implements CodeElement {
    private String name;
    private String returnType;
    private List<String> parameters;

    @Override
    public void accept(CodeVisitor visitor) {
        visitor.visit(this);
    }

    // getters and setters
}

public interface CodeVisitor {
    void visit(ClassElement element);
    void visit(MethodElement element);
}

public class ComplexityAnalyzer implements CodeVisitor {
    @Override
    public void visit(ClassElement element) {
        System.out.println("分析类复杂度: " + element.getName());
    }

    @Override
    public void visit(MethodElement element) {
        System.out.println("分析方法复杂度: " + element.getName());
    }
}
```

**哲学思考**：访问者模式体现了"双重分派"的思想，它通过将操作逻辑集中到访问者中，实现了对数据结构操作的灵活扩展。

## 12. 行为型模式的比较与选择

### 12.1 模式对比

| 模式 | 主要目的 | 优点 | 缺点 |
|------|----------|------|------|
| 责任链 | 请求传递 | 灵活处理请求 | 请求可能不被处理 |
| 命令 | 封装请求 | 支持撤销和重做 | 产生大量命令类 |
| 解释器 | 语言处理 | 易于扩展语法 | 性能开销大 |
| 迭代器 | 遍历集合 | 统一访问方式 | 增加系统复杂度 |
| 中介者 | 对象协调 | 减少对象耦合 | 中介者可能复杂 |
| 备忘录 | 状态保存 | 简化状态管理 | 可能破坏封装 |
| 观察者 | 通知机制 | 松耦合设计 | 可能导致内存泄漏 |
| 状态 | 行为切换 | 清晰的状态逻辑 | 增加类数量 |
| 策略 | 算法封装 | 算法可切换 | 客户端了解策略 |
| 模板方法 | 流程控制 | 代码复用 | 继承限制 |
| 访问者 | 操作扩展 | 易于添加操作 | 增加元素困难 |

### 12.2 选择原则

1. **责任链模式**：当需要多个对象处理请求时
2. **命令模式**：当需要支持撤销和重做时
3. **观察者模式**：当需要一对多通知时
4. **状态模式**：当对象行为随状态变化时
5. **策略模式**：当需要算法可切换时
6. **模板方法模式**：当有固定流程但部分步骤可变时
7. **访问者模式**：当需要为对象结构添加新操作时

### 12.3 现代应用趋势

在现代Java开发中，行为型模式的应用方式正在演进：

```java
// 使用函数式接口实现策略模式
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.stream().filter(n -> n > 3).collect(Collectors.toList());

// 使用观察者模式的响应式编程
Flux.just("item1", "item2", "item3")
    .subscribe(item -> System.out.println("收到: " + item));
```

## 13. 最佳实践与反模式

### 13.1 最佳实践

1. **单一职责**：每个类应该只有一个改变的理由
2. **开闭原则**：对扩展开放，对修改封闭
3. **依赖倒置**：依赖抽象而不是具体实现
4. **组合优于继承**：优先使用组合而不是继承

### 13.2 常见反模式

```java
// 错误：过度使用状态模式
public class OvercomplicatedState {
    private State state;
    // 状态过多，难以维护
}

// 错误：观察者模式中的内存泄漏
public class LeakyObserver {
    private static List<Observer> observers = new ArrayList<>();
    // 静态集合可能导致内存泄漏
}
```

## 14. 未来发展趋势

行为型模式在未来软件开发中依然具有重要价值：

1. **响应式编程**：观察者模式的演进
2. **函数式编程**：策略模式和命令模式的函数式实现
3. **事件驱动架构**：责任链和观察者模式的应用
4. **AI和机器学习**：策略模式在算法选择中的应用

## 结语

行为型设计模式是构建复杂交互系统的重要工具。它们提供了优雅的方式来管理对象之间的交互和职责分配，使得系统更加灵活、可维护和可扩展。

通过深入理解这些模式的设计哲学，我们能够更好地应对复杂的软件设计挑战。记住，最好的设计不是使用最多的模式，而是在合适的场景选择合适的模式。

**优秀的行为设计应该让对象之间的交互既清晰又灵活，既简单又强大。行为型模式正是帮助我们实现这一目标的强大工具。**

---

*这篇文章深入探讨了行为型设计模式的核心概念、实现技巧和实际应用。通过具体的代码示例和实际应用场景，展示了这些模式如何帮助我们管理对象之间的交互和职责分配。浅者可以学会基本的使用方法，深者能够理解其设计哲学和应用技巧。*