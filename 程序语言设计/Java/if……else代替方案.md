# if...esle代替方案


## 错误示范
一般来说我们正常的后台管理系统都有所谓的角色的概念，不同管理员权限不一样，能够行使的操作也不一样，比如：

* 系统管理员（ ROLE_ROOT_ADMIN）：有 A操作权限
* 订单管理员（ ROLE_ORDER_ADMIN）：有 B操作权限
* 普通用户（ ROLE_NORMAL）：有 C操作权限

比如一个用户进来，我们需要根据不同用户的角色来判断其有哪些行为，这时候if...else代码出现了：

```java
public class JudgeRole {
    public String judge(String roleName){
        String result = "";
        if(roleName.equals("ROLE_ROOT_ADMIN")){
            // 系统管理员有A权限
            result = "ROLE_ROOT_ADMIN: " + "has AAA permission";
        } else if(roleName.equals("ROLE_ORDER_ADMIN")){
            // 订单管理员有B权限
            result = "ROLE_ORDER_ADMIN: " + "has BBB permission";
        } else if(roleName.equals("ROLE_NORMAL")){
            // 普通用户有C权限
            result = "ROLE_NORMAL: " + "has CCC permission";
        } else{
            result = "XXX";
        }
        return result;
    }
}
```

## 枚举和接口
首先定义一个公用接口 _RoleOperation_，表示不同角色所能做的操作：

```java
public interface RoleOperation {
    String op (); // 表示某个角色可以做哪些op操作
}
```

接下来将不同角色的情况全部交由枚举类来做，定义一个不同角色有不同权限的枚举类 _RoleEnum_：

```java
public enum RoleEnum implements RoleOperation{
    // 系统管理员(有A操作权限)
    ROLE_ROOT_ADMIN {
        @Override
        public String op(){
            return "ROLE_ROOT_ADMIN:" + " has AAA permission";
        }
    },
    // 订单管理员(有B操作权限)
    ROLE_ORDER_ADMIN {
        @Override
        public String op(){
            return "ROLE_ORDER_ADMIN:" + " has BBB permission";
        }
    },
    // 普通用户(有C操作权限)
    ROLE_NORMAL {
        @Override
        public String op() {
            return "ROLE_NORMAL:" + " has CCC permission";
        }
    };
}
```

接下来调用就变得异常简单了，一行代码就行了， if/else也灰飞烟灭了：

```java
public class JudgeRole {
    public String judge(String roleName) {
        // 一行代码搞定！之前的if/else没了！
        return RoleEnum.valueOf(roleName).op();
    }
}
```

## 工厂模式

不同分支做不同的事情，很明显就提供了使用工厂模式的契机，我们只需要将不同情况单独定义好，然后去工厂类里面聚合即可。
首先，针对不同的角色，单独定义其业务类：

```java
// 系统管理员(有A操作权限)
public class RootAdminRole implements RoleOperation {
    private String roleName;
    public RootAdminRole(String roleName) {
        this.roleName = roleName;
    }
    @Override
    public String op() {
        return roleName + " has AAA permission";
    }
}
// 订单管理员(有B操作权限)
public class OrderAdminRole implements RoleOperation {
    private String roleName;
    public OrderAdminRole(String roleName) {
        this.roleName = roleName;
    }
    @Override
    public String op() {
        return roleName + " has BBB permission";
    }
}
// 普通用户(有C操作权限)
public class NormalRole implements RoleOperation {
    private String roleName;
    public NormalRole(String roleName) {
        this.roleName = roleName;
    }
    @Override
    public String op() {
        return roleName + " has CCC permission";
    }
}
```

接下来再写一个工厂类 RoleFactory对上面不同角色进行聚合：

```java
public class RoleFactory {
    static Map<String, RoleOperation> roleOperationMap = new HashMap<>();
    // 在静态块中先把初始化工作全部做完
    static {
        roleOperationMap.put("ROLE_ROOT_ADMIN", new RootAdminRole("ROLE_ROOT_ADMIN"));
        roleOperationMap.put("ROLE_ORDER_ADMIN", new OrderAdminRole("ROLE_ORDER_ADMIN"));
        roleOperationMap.put("ROLE_NORMAL", new NormalRole("ROLE_NORMAL"));
    }
    public static RoleOperation getOp(String roleName) {
        return roleOperationMap.get(roleName);
    }
}
```

接下来借助上面这个工厂，业务代码调用也只需一行代码， if/else同样被消除了：

```java
public class JudgeRole {
    public String judge(String roleName) {
        // 一行代码搞定！之前的 if/else也没了！
        return RoleFactory.getOp(roleName).op();
    }
}
```

## 策略模式

策略模式和工厂模式写起来其实区别也不大！
在上面工厂模式代码的基础上，按照策略模式的指导思想，我们也来创建一个所谓的**策略上下文类**，这里命名为 RoleContext：

```java
public class RoleContext {
    // 可更换的策略，传入不同的策略对象，业务即相应变化
    private RoleOperation operation;
    public RoleContext(RoleOperation operation) {
        this.operation = operation;
    }
    public String execute() {
        return operation.op();
    }
}
```

很明显上面传入的参数 operation就是表示不同的“策略”。我们在业务代码里传入不同的角色，即可得到不同的操作结果：

```java
public class JudgeRole {
    public String judge(RoleOperation roleOperation) {
        RoleContext roleContext = new RoleContext(roleOperation);
        return roleContext.execute();
    }
}
public static void main(String[] args) {
    JudgeRole judgeRole = new JudgeRole();
    String result1 = judgeRole.judge(new RootAdminRole("ROLE_ROOT_ADMIN"));
    System.out.println(result1);
    String result2 = judgeRole.judge(new OrderAdminRole("ROLE_ORDER_ADMIN"));
    System.out.println(result2);
    String result3 = judgeRole.judge(new NormalRole("ROLE_NORMAL"));
    System.out.println(result3);
}
```

## WEB应用策略模式+工厂模式
方便spring获取RoleService，创建一个工厂类：

```java
public class RoleServiceStrategyFactory {
    private static Map<String,RoleService> services = new ConcurrentHashMap<String,RoleService>();
    public static RoleService getByRole(String role){
        return services.get(role);
    }
    public static void register(String role, RoleService roleService){
        Assert.notNull(role, "role can't be null");
        services.put(role, roleService);
    }
}
```

有了这个工厂类之后，角色权限的代码得以优化：

```java
public static void main(String[] args){
    String role = "ROLE_ROOT_ADMIN";
    RoleService strategy = RoleServiceStrategyFactory.getByRole(role);
    System.out.pringln(strategy.op());
}
```

工厂模式中保存的所有策略需要在初始化，借用Spring提供的InitializingBean接口：

```java
@Service
public class RootAdminRoleService implements RoleService,InitializingBean {
    @Override
    public String op (){
        return "ROLE_ROOT_ADMIN:" + " has AAA permission";
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        RoleServiceStrategyFactory.register("ROLE_ROOT_ADMIN",this);
    }
}

@Service
public class OrderAdminRoleService implements RoleService,InitializingBean {
    @Override
    public String op (){
        return "ROLE_ORDER_ADMIN:" + " has BBB permission";
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        RoleServiceStrategyFactory.register("ROLE_ORDER_ADMIN",this);
    }
}

@Service
public class NormalService implements RoleService,InitializingBean {
    @Override
    public String op (){
        return "ROLE_NORMAL:" + " has CCC permission";
    }
    @Override
    public void afterPropertiesSet() throws Exception{
        RoleServiceStrategyFactory.register("ROLE_NORMAL",this);
    }
}
```