# WebDataBinder应用和原理
web环境统一对数据绑定`DataBinder`进行了增强

它的作用就是从web request里（**注意：这里指的web请求，并不一定就是ServletRequest请求哟**）把web请求的 `parameters` 绑定到 `JavaBean` 上 Controller 方法的参数类型可以是基本类型，也可以是封装后的普通Java类型。若这个普通Java类型没有声明任何注解，则意味着它的每一个属性都需要到 Request 中去查找对应的请求参数。

单从WebDataBinder来说，它对父类进行了增强，提供的增强能力如下：

* 支持对属性名以\_打头的默认值处理（自动挡，能够自动处理所有的Bool、Collection、Map等）
* 支持对属性名以!打头的默认值处理（手动档，需要手动给某个属性赋默认值，自己控制的灵活性很高）
* 提供方法，支持把 MultipartFile 绑定到 JavaBean 的属性上\~

Demo示例
下面以一个示例来演示使用它增强的这些功能：

```java
@Getter
@Setter
@ToString
public class Person {
	public String name;
	public Integer age;

	// 基本数据类型
	public Boolean flag;
	public int index;
	public List<String> list;
	public Map<String, String> map;	
}
```

演示使用!手动精确控制字段的默认值：

```java
public static void main(String[] args) {
    Person person = new Person();
    WebDataBinder binder = new WebDataBinder(person, "person");

    // 设置属性（此处演示一下默认值）
    MutablePropertyValues pvs = new MutablePropertyValues();

    // 使用!来模拟各个字段手动指定默认值
    //pvs.add("name", "fsx");
    pvs.add("!name", "不知火舞");
    pvs.add("age", 18);
    pvs.add("!age", 10); // 上面有确切的值了，默认值不会再生效

    binder.bind(pvs);
    System.out.println(person);
}
```

打印输出（符合预期）：

```
`Person(name=null, age=null, flag=false, index=0, list=[], map={}) `&#x20;
```
请用此打印结果对比一下上面的结果，你是会有很多发现，比如能够发现基本类型的默认值就是它自己。 另一个很显然的道理：若你啥都不做特殊处理，包装类型默认值那铁定都是null了\~

了解了 `WebDataBinder` 后，继续看看它的一个重要子类 `ServletRequestDataBinder`

### ServletRequestDataBinder&#x20;

前面说了这么多，亲有没有发现还木有聊到过我们最为常见的 Web 场景API：javax.servlet.ServletRequest。本类从命名上就知道，它就是为此而生。

它的目标就是：data binding from servlet request parameters to JavaBeans, including support for multipart files.从 `Servlet Request` 里把参数绑定到 `JavaBean` 里，支持multipart。

> 备注：到此类为止就已经把web请求限定为了Servlet Request，和Servlet规范强绑定了。

```java
public class ServletRequestDataBinder extends WebDataBinder {
	... // 沿用父类构造 
	// 注意这个可不是父类的方法，是本类增强的~~~~意思就是kv都从request里来~~当然内部还是适配成了一个MutablePropertyValues
	public void bind(ServletRequest request) { 
		// 内部最核心方法是它：WebUtils.getParametersStartingWith()  把request参数转换成一个Map
		// request.getParameterNames()
		MutablePropertyValues mpvs = new ServletRequestParameterPropertyValues(request);
		MultipartRequest multipartRequest = WebUtils.getNativeRequest(request, MultipartRequest.class);
		// 调用父类的bindMultipart方法，把MultipartFile都放进MutablePropertyValues里去~~~
		if (multipartRequest != null) {
			bindMultipart(multipartRequest.getMultiFileMap(), mpvs);
		}
		// 这个方法是本类流出来的一个扩展点~~~子类可以复写此方法自己往里继续添加
		// 比如ExtendedServletRequestDataBinder它就复写了这个方法，进行了增强（下面会说）  支持到了uriTemplateVariables的绑定
		addBindValues(mpvs, request);
		doBind(mpvs);
	}

	// 这个方法和父类的close方法类似，很少直接调用
	public void closeNoCatch() throws ServletRequestBindingException {
		if (getBindingResult().hasErrors()) {
			throw new ServletRequestBindingException("Errors binding onto object '" + getBindingResult().getObjectName() + "'", new BindException(getBindingResult()));
		}
	}
}
```

下面就以 `MockHttpServletRequest` 为例作为Web 请求实体，演示一个使用的小Demo。说明：`MockHttpServletRequest` 它是 `HttpServletRequest` 的实现类\~

Demo示例

```java
public static void main(String[] args) {
	Person person = new Person();
	ServletRequestDataBinder binder = new ServletRequestDataBinder(person, "person");  
	// 构造参数，此处就不用MutablePropertyValues，以HttpServletRequest的实现类MockHttpServletRequest为例吧
    MockHttpServletRequest request = new MockHttpServletRequest();
    // 模拟请求参数
    request.addParameter("name", "fsx");
    request.addParameter("age", "18");

    // flag不仅仅可以用true/false  用0和1也是可以的？
    request.addParameter("flag", "1");

    // 设置多值的
    request.addParameter("list", "4", "2", "3", "1");
    // 给map赋值(Json串)
    // request.addParameter("map", "{'key1':'value1','key2':'value2'}"); // 这样可不行
    request.addParameter("map['key1']", "value1");
    request.addParameter("map['key2']", "value2");

     一次性设置多个值（传入Map）
    //request.setParameters(new HashMap<String, Object>() {{
    //    put("name", "fsx");
    //    put("age", "18");
    //}});

    binder.bind(request);
    System.out.println(person);
}
```

打印输出：

```
`Person(name=fsx, age=18, flag=true, index=0, list=[4, 2, 3, 1], map={key1=value1, key2=value2}) ` 完美。
```

思考题：小伙伴可以思考为何给Map属性传值是如上，而不是value写个json就行呢？

ExtendedServletRequestDataBinder
此类代码不多但也不容小觑，它是对ServletRequestDataBinder的一个增强，它用于把URI template variables参数添加进来用于绑定。它会去从request的HandlerMapping.class.getName() + ".uriTemplateVariables";这个属性里查找到值出来用于绑定\~\~\~

比如我们熟悉的@PathVariable它就和这相关：它负责把参数从url模版中解析出来，然后放在attr上，最后交给ExtendedServletRequestDataBinder进行绑定\~\~\~

介于此：我觉得它还有一个作用，就是定制我们全局属性变量用于绑定\~

向此属性放置值的地方是：AbstractUrlHandlerMapping.lookupHandler() --> chain.addInterceptor(new UriTemplateVariablesHandlerInterceptor(uriTemplateVariables)); --> preHandle()方法 -> exposeUriTemplateVariables(this.uriTemplateVariables, request); -> request.setAttribute(URI\_TEMPLATE\_VARIABLES\_ATTRIBUTE, uriTemplateVariables);

```java
// @since 3.1
public class ExtendedServletRequestDataBinder extends ServletRequestDataBinder {
... // 沿用父类构造
//本类的唯一方法
@Override
@SuppressWarnings("unchecked")
protected void addBindValues(MutablePropertyValues mpvs, ServletRequest request) {
	// 它的值是：HandlerMapping.class.getName() + ".uriTemplateVariables";
	String attr = HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE;

	// 注意：此处是attr，而不是parameter
	Map<String, String> uriVars = (Map<String, String>) request.getAttribute(attr);
	if (uriVars != null) {
		uriVars.forEach((name, value) -> {
			
			// 若已经存在确切的key了，不会覆盖~~~~
			if (mpvs.contains(name)) {
				if (logger.isWarnEnabled()) {
					logger.warn("Skipping URI variable '" + name + "' because request contains bind value with same name.");
				}
			} else {
				mpvs.addPropertyValue(name, value);
			}
		});
	}
}
```

可见，通过它我们亦可以很方便的做到在每个ServletRequest提供一份共用的模版属性们，供以绑定\~

此类基本都沿用父类的功能，比较简单，此处就不写Demo了（Demo请参照父类）\~

说明：ServletRequestDataBinder一般不会直接使用，而是使用更强的子类ExtendedServletRequestDataBinder

WebExchangeDataBinder
它是Spring5.0后提供的，对Reactive编程的Mono数据绑定提供支持，因此暂略\~

data binding from URL query params or form data in the request data to Java objects
1
MapDataBinder
它位于org.springframework.data.web是和Spring-Data相关，专门用于处理target是Map\<String, Object>类型的目标对象的绑定，它并非一个public类\~

它用的属性访问器是MapPropertyAccessor：一个继承自AbstractPropertyAccessor的私有静态内部类\~（也支持到了SpEL哦）

WebRequestDataBinder
它是用于处理Spring自己定义的org.springframework.web.context.request.WebRequest的，旨在处理和容器无关的web请求数据绑定，有机会详述到这块的时候，再详细说\~

如何注册自己的PropertyEditor来实现自定义类型数据绑定？
通过前面的分析我们知道了，数据绑定这一块最终会依托于PropertyEditor来实现具体属性值的转换（毕竟request传进来的都是字符串嘛\~）

一般来说，像String, int, long会自动绑定到参数都是能够自动完成绑定的，因为前面有说，默认情况下Spring是给我们注册了N多个解析器的：

```java
public class PropertyEditorRegistrySupport implements PropertyEditorRegistry {

    @Nullable
    private Map<Class<?>, PropertyEditor> defaultEditors;

    private void createDefaultEditors() {
    	this.defaultEditors = new HashMap<>(64);

    	// Simple editors, without parameterization capabilities.
    	// The JDK does not contain a default editor for any of these target types.
    	this.defaultEditors.put(Charset.class, new CharsetEditor());
    	this.defaultEditors.put(Class.class, new ClassEditor());
    	...
    	// Default instances of collection editors.
    	// Can be overridden by registering custom instances of those as custom editors.
    	this.defaultEditors.put(Collection.class, new CustomCollectionEditor(Collection.class));
    	this.defaultEditors.put(Set.class, new CustomCollectionEditor(Set.class));
    	this.defaultEditors.put(SortedSet.class, new CustomCollectionEditor(SortedSet.class));
    	this.defaultEditors.put(List.class, new CustomCollectionEditor(List.class));
    	this.defaultEditors.put(SortedMap.class, new CustomMapEditor(SortedMap.class));
    	...
    	// 这里就部全部枚举出来了
    }
}
```

虽然默认注册支持的Editor众多，但是依旧发现它并没有对Date类型、以及Jsr310提供的各种事件、日期类型的转换（当然也包括我们的自定义类型）。
因此我相信小伙伴都遇到过这样的痛点：Date、LocalDate等类型使用自动绑定老不方便了，并且还经常傻傻搞不清楚。所以最终很多都无奈选择了语义不是非常清晰的时间戳来传递

演示Date类型的数据绑定Demo：
```java
@Getter
@Setter
@ToString
public class Person {

    public String name;
    public Integer age;

    // 以Date类型为示例
    private Date start;
    private Date end;
    private Date endTest;

}

public static void main(String[] args) {
    Person person = new Person();
    DataBinder binder = new DataBinder(person, "person");

    // 设置属性
    MutablePropertyValues pvs = new MutablePropertyValues();
    pvs.add("name", "fsx");

    // 事件类型绑定
    pvs.add("start", new Date());
    pvs.add("end", "2019-07-20");
    // 试用试用标准的事件日期字符串形式~
    pvs.add("endTest", "Sat Jul 20 11:00:22 CST 2019");


    binder.bind(pvs);
    System.out.println(person);
}
```

打印输出：

```
Person(name=fsx, age=null, start=Sat Jul 20 11:05:29 CST 2019, end=null, endTest=Sun Jul 21 01:00:22 CST 2019)
```

结果是符合我预期的：start有值，end没有，endTest却有值。
可能小伙伴对start、end都可以理解，最诧异的是endTest为何会有值呢？？？
此处我简单解释一下处理步骤：

BeanWrapper调用setPropertyValue()给属性赋值，传入的value值都会交给convertForProperty()方法根据get方法的返回值类型进行转换\~（比如此处为Date类型）
委托给this.typeConverterDelegate.convertIfNecessary进行类型转换（比如此处为string->Date类型）
先this.propertyEditorRegistry.findCustomEditor(requiredType, propertyName);找到一个合适的PropertyEditor（显然此处我们没有自定义Custom处理Date的PropertyEditor，返回null）
回退到使用ConversionService，显然此处我们也没有设置，返回null
回退到使用默认的editor = findDefaultEditor(requiredType);（注意：此处只根据类型去找了，因为上面说了默认不处理了Date，所以也是返回null）
最终的最终，回退到Spring对Array、Collection、Map的默认值处理问题，最终若是String类型，都会调用BeanUtils.instantiateClass(strCtor, convertedValue)也就是有参构造进行初始化\~\~\~(请注意这必须是String类型才有的权利)

1.  所以本例中，到最后一步就相当于new Date("Sat Jul 20 11:00:22 CST 2019")，因为该字符串是标准的时间日期串，所以是阔仪的，也就是endTest是能被正常赋值的\~
    通过这个简单的步骤分析，解释了为何end没值，endTest有值了。
    其实通过回退到的最后一步处理，我们还可以对此做巧妙的应用。比如我给出如下的一个巧用例子：

```java
@Getter
@Setter
@ToString
public class Person {
private String name;
// 备注：child是有有一个入参的构造器的
private Child child;
}

@Getter
@Setter
@ToString
public class Child {
    private String name;
    private Integer age;
    public Child() {
    }
    public Child(String name) {
    this.name = name;
    }
}

public static void main(String[] args) {
    Person person = new Person();
    DataBinder binder = new DataBinder(person, "person");

    // 设置属性
    MutablePropertyValues pvs = new MutablePropertyValues();
    pvs.add("name", "fsx");

    // 给child赋值，其实也可以传一个字符串就行了 非常的方便   Spring会自动给我们new对象
    pvs.add("child", "fsx-son");
    
    binder.bind(pvs);
    System.out.println(person);
}
```

打印输出：

```
Person(name=fsx, child=Child(name=fsx-son, age=null))
```

完美。

废话不多说，下面我通过自定义属性编辑器的手段，来让能够支持处理上面我们传入2019-07-20这种非标准的时间字符串。

我们知道DataBinder本身就是个PropertyEditorRegistry，因此我只需要自己注册一个自定义的PropertyEditor即可：

1、通过继承PropertyEditorSupport实现一个自己的处理Date的编辑器：

```java
public class MyDatePropertyEditor extends PropertyEditorSupport {

    private static final String PATTERN = "yyyy-MM-dd";

    @Override
    public String getAsText() {
        Date date = (Date) super.getValue();
        return new SimpleDateFormat(PATTERN).format(date);
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        try {
            super.setValue(new SimpleDateFormat(PATTERN).parse(text));
        } catch (ParseException e) {
            System.out.println("ParseException....................");
        }
    }
}
```

2、注册进DataBinder并运行

```java
public static void main(String[] args) {
    Person person = new Person();
    DataBinder binder = new DataBinder(person, "person");
    binder.registerCustomEditor(Date.class, new MyDatePropertyEditor());
    //binder.registerCustomEditor(Date.class, "end", new MyDatePropertyEditor());

    // 设置属性
    MutablePropertyValues pvs = new MutablePropertyValues();
    pvs.add("name", "fsx");

    // 事件类型绑定
    pvs.add("start", new Date());
    pvs.add("end", "2019-07-20");
    // 试用试用标准的事件日期字符串形式~
    pvs.add("endTest", "Sat Jul 20 11:00:22 CST 2019");


    binder.bind(pvs);
    System.out.println(person);
}
```
运行打印如下：

```
ParseException....................
Person(name=fsx, age=null, start=Sat Jul 20 11:41:49 CST 2019, end=Sat Jul 20 00:00:00 CST 2019, endTest=null)
```

结果符合预期。不过对此结果我仍旧抛出如下两个问题供小伙伴自行思考：
1、输出了ParseException…
2、start有值，endTest值却为null了

理解这块最后我想说：通过自定义编辑器，我们可以非常自由、高度定制化的完成自定义类型的封装，可以使得我们的Controller更加容错、更加智能、更加简洁。有兴趣的可以运用此块知识，自行实践\~

WebBindingInitializer和WebDataBinderFactory
WebBindingInitializer
WebBindingInitializer：实现此接口重写initBinder方法注册的属性编辑器是全局的属性编辑器，对所有的Controller都有效。

可以简单粗暴的理解为：WebBindingInitializer为编码方式，@InitBinder为注解方式（当然注解方式还能控制到只对当前Controller有效，实现更细粒度的控制）

观察发现，Spring对这个接口的命名很有意思：它用的Binding正在进行时态\~

```java
// @since 2.5   Spring在初始化WebDataBinder时候的回调接口，给调用者自定义\~
public interface WebBindingInitializer {
    // @since 5.0
    void initBinder(WebDataBinder binder);

    // @deprecated as of 5.0 in favor of {@link #initBinder(WebDataBinder)}
    @Deprecated
    default void initBinder(WebDataBinder binder, WebRequest request) {
    	initBinder(binder);
    }
}
```

此接口它的内建唯一实现类为：ConfigurableWebBindingInitializer，若你自己想要扩展，建议继承它\~

```java
public class ConfigurableWebBindingInitializer implements WebBindingInitializer {
private boolean autoGrowNestedPaths = true;
private boolean directFieldAccess = false; // 显然这里是false

    // 下面这些参数，不就是WebDataBinder那些可以配置的属性们吗？
    @Nullable
    private MessageCodesResolver messageCodesResolver;
    @Nullable
    private BindingErrorProcessor bindingErrorProcessor;
    @Nullable
    private Validator validator;
    @Nullable
    private ConversionService conversionService;
    // 此处使用的PropertyEditorRegistrar来管理的，最终都会被注册进PropertyEditorRegistry嘛
    @Nullable
    private PropertyEditorRegistrar[] propertyEditorRegistrars;

    ... //  省略所有get/set

    // 它做的事无非就是把配置的值都放进去而已~~
    @Override
    public void initBinder(WebDataBinder binder) {
    	binder.setAutoGrowNestedPaths(this.autoGrowNestedPaths);
    	if (this.directFieldAccess) {
    		binder.initDirectFieldAccess();
    	}
    	if (this.messageCodesResolver != null) {
    		binder.setMessageCodesResolver(this.messageCodesResolver);
    	}
    	if (this.bindingErrorProcessor != null) {
    		binder.setBindingErrorProcessor(this.bindingErrorProcessor);
    	}
    	// 可以看到对校验器这块  内部还是做了容错的
    	if (this.validator != null && binder.getTarget() != null && this.validator.supports(binder.getTarget().getClass())) {
    		binder.setValidator(this.validator);
    	}
    	if (this.conversionService != null) {
    		binder.setConversionService(this.conversionService);
    	}
    	if (this.propertyEditorRegistrars != null) {
    		for (PropertyEditorRegistrar propertyEditorRegistrar : this.propertyEditorRegistrars) {
    			propertyEditorRegistrar.registerCustomEditors(binder);
    		}
    	}
    }

}
```

此实现类主要是提供了一些可配置项，方便使用。注意：此接口一般不直接使用，而是结合InitBinderDataBinderFactory、WebDataBinderFactory等一起使用\~

WebDataBinderFactory
顾名思义它就是来创造一个WebDataBinder的工厂。

```java
// @since 3.1   注意：WebDataBinder 可是1.2就有了\~
public interface WebDataBinderFactory {
// 此处使用的是Spring自己的NativeWebRequest   后面两个参数就不解释了
WebDataBinder createBinder(NativeWebRequest webRequest, @Nullable Object target, String objectName) throws Exception;
}
```

它的继承树如下：

```java
DefaultDataBinderFactory
public class DefaultDataBinderFactory implements WebDataBinderFactory {
@Nullable
private final WebBindingInitializer initializer;
// 注意：这是唯一构造函数
public DefaultDataBinderFactory(@Nullable WebBindingInitializer initializer) {
this.initializer = initializer;
}

    // 实现接口的方法
    @Override
    @SuppressWarnings("deprecation")
    public final WebDataBinder createBinder(NativeWebRequest webRequest, @Nullable Object target, String objectName) throws Exception {

    	WebDataBinder dataBinder = createBinderInstance(target, objectName, webRequest);
    	
    	// 可见WebDataBinder 创建好后，此处就会回调（只有一个）
    	if (this.initializer != null) {
    		this.initializer.initBinder(dataBinder, webRequest);
    	}
    	// 空方法 子类去实现，比如InitBinderDataBinderFactory实现了词方法
    	initBinder(dataBinder, webRequest);
    	return dataBinder;
    }

    //  子类可以复写，默认实现是WebRequestDataBinder
    // 比如子类ServletRequestDataBinderFactory就复写了，使用的new ExtendedServletRequestDataBinder(target, objectName)
    protected WebDataBinder createBinderInstance(@Nullable Object target, String objectName, NativeWebRequest webRequest) throws Exception 
    	return new WebRequestDataBinder(target, objectName);
    }

}
```

按照Spring一贯的设计，本方法实现了模板动作，子类只需要复写对应的动作即可达到效果。

InitBinderDataBinderFactory
它继承自DefaultDataBinderFactory，主要用于处理标注有@InitBinder的方法做初始绑定\~

```java
// @since 3.1
public class InitBinderDataBinderFactory extends DefaultDataBinderFactory {

    // 需要注意的是：`@InitBinder`可以标注N多个方法~  所以此处是List
    private final List<InvocableHandlerMethod> binderMethods;

    // 此子类的唯一构造函数
    public InitBinderDataBinderFactory(@Nullable List<InvocableHandlerMethod> binderMethods, @Nullable WebBindingInitializer initializer) {
    	super(initializer);
    	this.binderMethods = (binderMethods != null ? binderMethods : Collections.emptyList());
    }

    // 上面知道此方法的调用方法生initializer.initBinder之后
    // 所以使用注解它生效的时机是在直接实现接口的后面的~
    @Override
    public void initBinder(WebDataBinder dataBinder, NativeWebRequest request) throws Exception {
    	for (InvocableHandlerMethod binderMethod : this.binderMethods) {
    		// 判断@InitBinder是否对dataBinder持有的target对象生效~~~（根据name来匹配的）
    		if (isBinderMethodApplicable(binderMethod, dataBinder)) {
    			// 关于目标方法执行这块，可以参考另外一篇@InitBinder的原理说明~
    			Object returnValue = binderMethod.invokeForRequest(request, null, dataBinder);

    			// 标注@InitBinder的方法不能有返回值
    			if (returnValue != null) {
    				throw new IllegalStateException("@InitBinder methods must not return a value (should be void): " + binderMethod);
    			}
    		}
    	}
    }

    //@InitBinder有个Value值，它是个数组。它是用来匹配dataBinder.getObjectName()是否匹配的   若匹配上了，现在此注解方法就会生效
    // 若value为空，那就对所有生效~~~
    protected boolean isBinderMethodApplicable(HandlerMethod initBinderMethod, WebDataBinder dataBinder) {
    	InitBinder ann = initBinderMethod.getMethodAnnotation(InitBinder.class);
    	Assert.state(ann != null, "No InitBinder annotation");
    	String[] names = ann.value();
    	return (ObjectUtils.isEmpty(names) || ObjectUtils.containsElement(names, dataBinder.getObjectName()));
    }

// @since 3.1 public class ServletRequestDataBinderFactory extends InitBinderDataBinderFactory { public ServletRequestDataBinderFactory(@Nullable List\<InvocableHandlerMethod> binderMethods, @Nullable WebBindingInitializer initializer) {
    super(binderMethods, initializer);
}
@Override
protected ServletRequestDataBinder createBinderInstance(
    @Nullable Object target, String objectName, NativeWebRequest request) throws Exception  {
        return new ExtendedServletRequestDataBinder(target, objectName);
    }
}
```

此工厂是RequestMappingHandlerAdapter这个适配器默认使用的一个数据绑定器工厂，而RequestMappingHandlerAdapter却又是当下使用得最频繁、功能最强大的一个适配器

总结
WebDataBinder在SpringMVC中使用，它不需要我们自己去创建，我们只需要向它注册参数类型对应的属性编辑器PropertyEditor。PropertyEditor可以将字符串转换成其真正的数据类型，它的void setAsText(String text)方法实现数据转换的过程。

好好掌握这部分内容，这在Spring MVC中结合@InitBinder注解一起使用将有非常大的威力，能一定程度上简化你的开发，提高效率
