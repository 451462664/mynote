# Spring 源码学习——Aop

---

## 什么是 AOP

以下是百度百科的解释：AOP 为 Aspect Oriented Programming 的缩写，意为：面向切面编程通过预编译的方式和运行期动态代理实现程序功能的统一维护的一种技术。

## AOP 的原理

AOP 的原理是什么？估计大家都知道了无非就是通过代理模式来为目标对象生产出一个代理对象来，并将相应的逻辑插入到目标方法的前后等等.如果别人这么问，这么回答没毛病，但是这么回答完发现 AOP 也没什么可说的了不是吗.毕竟确实原理就是动态代理.但是 Spring 在具体的细节实现上可没有那么简单，还是很值的我们去一探究竟的.

## AOP 术语

在学习之前我们需要了解一下基本知识有助于我们去学习.

### 连接点(join point)

> a point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.

引用一下官方的说明，连接点就是程序运行时的一些点，比如我们一个类中的方法调用、异常处理.你可以认为所有的方法均是连接点.

### 切点(point cut)

切点主要的作用就是用来匹配到对应的连接点，对应语法 execution(* *.find*(..)).

### 通知(advice)

为对应切点(point cut)匹配到的连接点做的一些程序代码逻辑.
Spring 定义了如下几个通知类型：

1. 前置通知(Before advice)在目标连接点执行前执行通知代码.
2. 后置通知(After advice)在目标连接点执行后执行通知代码.
3. 返回通知(After returning advice)在目标连接点执行成功后执行通知代码.
4. 异常通知(After throwing advice)在目标连接点异常后执行通知代码.
5. 环绕通知(Around advice)在目标连接点调用前后执行通知代码.

### 切面(Aspect)

切面是整合切点(point cut)、通知(advice)的一个类，也是由这两个组成的.既包含了横切逻辑的定义, 也包括了连接点的定义.

### 织入(Weaving)

现在什么都齐全了，还差一个将他们运行起来的东西，这个东西就是织入.织入就是在切点的匹配下将创建号的通知的逻辑整合到目标类的连接点上.

### 准备就绪

基础知识已经说完了，下面我们将进入到源码环节！

## AOP 的入口在哪里？

AOP 既可以通过 xml 配置来声明也可以通过注解的形式来声明.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <aop:config>
        <aop:aspect ref="aopAspect">
            <aop:pointcut id="helloPointcut" expression="execution(* com.x.spring.aop.x.*.*(..))"/>
            <aop:before method="methodBefore" pointcut-ref="helloPointcut"/>
            <aop:after method="methodAfter" pointcut-ref="helloPointcut"/>
        </aop:aspect>
    </aop:config>
</beans>
```

我们可以看到如果需要配置 aop 的标签我们需要引入 aop schema 的声明 xmlns:aop="http://www.springframework.org/schema/aop".我们直接点进去会发现有这么一行代码.

```xml
<xsd:documentation source="java:org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"><![CDATA[
 Enables the use of the @AspectJ style of Spring AOP.
   ]]></xsd:documentation>
```

虽然它注释了说开启允许使用 Spring AOP 的 @AspectJ 样式.但是所有逻辑都是相同的我们进去就能找到我们需要的了.

**AnnotationAwareAspectJAutoProxyCreator 继承结构**
AbstractAutoProxyCreator
	|--- AbstractAdvisorAutoProxyCreator
		|--- AspectJAwareAdvisorAutoProxyCreator
			|--- AnnotationAwareAspectJAutoProxyCreator

```java
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
		implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {
}

public interface SmartInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessor {
}
```

我们会发现 AbstractAutoProxyCreator 父类实现了 SmartInstantiationAwareBeanPostProcessor 后置处理器接口.并且 SmartInstantiationAwareBeanPostProcessor 还继承了 InstantiationAwareBeanPostProcessor.我们来看看 InstantiationAwareBeanPostProcessor 的说明.

> BeanPostProcessor 的子接口，用于执行 bean 实例化前回调，通常用于处理特定目标 bean 的默认实例化，例如创建具有特殊 TargetSource 的代理(池化目标，延迟初始化目标等)，或实现其他注入策略(如字段注入).
>

## AOP 源码分析

原来 aop 代理对象的生成都是通过 BeanPostProcessor 这样的后置处理器来实现的.Spring AOP 抽象代理创建器实现了 BeanPostProcessor 接口，并在 bean 初始化后置处理过程中向 bean 中织入通知。下面我们就来看看相关源码.我们通过寻找发现 BeanPostProcessor 中的方法 default Object postProcessAfterInitialization(Object bean, String beanName); 在 AbstractAutoProxyCreator 类里被重写的.

```java
/**
 * Create a proxy with the configured interceptors if the bean is
 * identified as one to proxy by the subclass.
 * @see #getAdvicesAndAdvisorsForBean
 */
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (!this.earlyProxyReferences.contains(cacheKey)) {
            // 生成代理对象，如果需要的话
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}

protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }
    // 如果是基础设施类或者需要跳过的类则直接返回
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }

    // 找出对应代理类的所有通知类
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    // 如果找到的话
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        // 创建代理
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }

    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```

**上面的代码主要做了**：

1. 校验.
2. 如果传入的 bean 是基础设施的类或者是需要跳过的类，直接返回该 bean.
3. 找出对应代理类的所有通知类.
4. 如果找到的话就直接创建代理返回.
5. 如果找不到直接返回传入的 bean.

###  找出对应代理类的所有通知类

我们直接看 getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);这个方法中做了些什么.

```java
@Override
@Nullable
protected Object[] getAdvicesAndAdvisorsForBean(
    Class<?> beanClass, String beanName, @Nullable TargetSource targetSource) {
    // 空壳方法在其内部调用了
    List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
    if (advisors.isEmpty()) {
        return DO_NOT_PROXY;
    }
    return advisors.toArray();
}

protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    // 查询出所有的通知器
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    // 过滤出对应 beanClass 上的通知器
    List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
    // 进行一些扩展操作
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}

@Override
protected List<Advisor> findCandidateAdvisors() {
    // 查询出所有的通知器
    List<Advisor> advisors = super.findCandidateAdvisors();
    // 解析 @Aspect 注解，并构建通知器
    if (this.aspectJAdvisorsBuilder != null) {
        advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    }
    return advisors;
}
```

**上述代码主要做了**：

1. 查询出所有的通知器.
2. 在查出的通知器中过滤出可以在目标 beanClass 使用的通知器.
3. 对通知器做一些扩展操作.

这样我们看到他调用了父类的 super.findCandidateAdvisors(); 方法.

```java
protected List<Advisor> findCandidateAdvisors() {
    Assert.state(this.advisorRetrievalHelper != null, "No BeanFactoryAdvisorRetrievalHelper available");
    return this.advisorRetrievalHelper.findAdvisorBeans();
}
```

可以发现原来父类也是只一个简单的方法，其中最主要的逻辑在 BeanFactoryAdvisorRetrievalHelper 的 findAdvisorBeans 方法中，即使不看里面的代码我们也可以猜到干了些什么真的是见名知意.

```java
public List<Advisor> findAdvisorBeans() {
    // 通知器都是缓存在 this.cachedAdvisorBeanNames 中的
    String[] advisorNames = this.cachedAdvisorBeanNames;
    if (advisorNames == null) {
		// 如果发现没有缓存，则去 BeanFactory IoC 容器中去查询出来.
        advisorNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
            this.beanFactory, Advisor.class, true, false);
        // 将查询出来的值赋值给 this.cachedAdvisorBeanNames
        this.cachedAdvisorBeanNames = advisorNames;
    }
    if (advisorNames.length == 0) {
        return new ArrayList<>();
    }
    List<Advisor> advisors = new ArrayList<>();
    for (String name : advisorNames) {
        if (isEligibleBean(name)) {
            // 如果当前的通知器正在创建中则打印日志忽略
            if (this.beanFactory.isCurrentlyInCreation(name)) {
                if (logger.isTraceEnabled()) {
                    logger.trace("Skipping currently created advisor '" + name + "'");
                }
            }
            else {
                try {
                    // 将通知器从容器中获取出来加入到 list 中
                    advisors.add(this.beanFactory.getBean(name, Advisor.class));
                }
                catch (BeanCreationException ex) {
                    throw ex;
                }
            }
        }
    }
    return advisors;
}
```

上面的逻辑就是从 IoC 容器中查找所有 Advisor 类型的 bean，在加这些 bean 缓存起来并返回.既然所有的通知器我们都取到了，那么我们继续来看 buildAspectJAdvisors 的方法.

```java
public List<Advisor> buildAspectJAdvisors() {
    List<String> aspectNames = this.aspectBeanNames;
    if (aspectNames == null) {
        synchronized (this) {
            aspectNames = this.aspectBeanNames;
            if (aspectNames == null) {
                List<Advisor> advisors = new ArrayList<>();
                aspectNames = new ArrayList<>();
                // 获取所有 bean 的 name
                String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
                    this.beanFactory, Object.class, true, false);
                // 遍历所有 beanName
                for (String beanName : beanNames) {
                    if (!isEligibleBean(beanName)) {
                        continue;
                    }
					// 获取到当前 bean 的类型
                    Class<?> beanType = this.beanFactory.getType(beanName);
                    if (beanType == null) {
                        continue;
                    }
                    // 判断当前 bean 是否有 Aspect 注解
                    if (this.advisorFactory.isAspect(beanType)) {
                        aspectNames.add(beanName);
                        AspectMetadata amd = new AspectMetadata(beanType, beanName);
                        if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
                            MetadataAwareAspectInstanceFactory factory =
                                new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
                            // 获取该 bean 上的所有通知器
                            List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
                            if (this.beanFactory.isSingleton(beanName)) {
                                this.advisorsCache.put(beanName, classAdvisors);
                            }
                            else {
                                this.aspectFactoryCache.put(beanName, factory);
                            }
                            // 加入获取到的通知器
                            advisors.addAll(classAdvisors);
                        }
                        else {
                            // Per target or per this.
                            if (this.beanFactory.isSingleton(beanName)) {
                                throw new IllegalArgumentException("Bean with name '" + beanName +
                                                                   "' is a singleton, but aspect instantiation model is not singleton");
                            }
                            MetadataAwareAspectInstanceFactory factory =
                                new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
                            this.aspectFactoryCache.put(beanName, factory);
                            advisors.addAll(this.advisorFactory.getAdvisors(factory));
                        }
                    }
                }
                this.aspectBeanNames = aspectNames;
                return advisors;
            }
        }
    }
    // 如果没有获取就返回一个空集合
    if (aspectNames.isEmpty()) {
        return Collections.emptyList();
    }
    List<Advisor> advisors = new ArrayList<>();
    for (String aspectName : aspectNames) {
        List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
        if (cachedAdvisors != null) {
            advisors.addAll(cachedAdvisors);
        }
        else {
            MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
            advisors.addAll(this.advisorFactory.getAdvisors(factory));
        }
    }
    return advisors;
}
```

通过上面的代码我们发现重要的点在这一行 this.advisorFactory.getAdvisors(factory);那么我们就来看看是怎么获取的.

```java
@Override
public List<Advisor> getAdvisors(MetadataAwareAspectInstanceFactory aspectInstanceFactory) {
    // 获取到 aspectClass 和 aspectName
    Class<?> aspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    String aspectName = aspectInstanceFactory.getAspectMetadata().getAspectName();
    validate(aspectClass);

    MetadataAwareAspectInstanceFactory lazySingletonAspectInstanceFactory =
        new LazySingletonAspectInstanceFactoryDecorator(aspectInstanceFactory);

    List<Advisor> advisors = new ArrayList<>();
    // getAdvisorMethods 或获取除了 @Pointcut 的方法
    for (Method method : getAdvisorMethods(aspectClass)) {
        // 构建出 Advisor
        Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, advisors.size(), aspectName);
        if (advisor != null) {
            advisors.add(advisor);
        }
    }
	// ... 以下代码省略
    return advisors;
}

/**
 * 获取除 @Pointcut 的方法
 */
private List<Method> getAdvisorMethods(Class<?> aspectClass) {
    final List<Method> methods = new ArrayList<>();
    ReflectionUtils.doWithMethods(aspectClass, method -> {
        if (AnnotationUtils.getAnnotation(method, Pointcut.class) == null) {
            methods.add(method);
        }
    });
    methods.sort(METHOD_COMPARATOR);
    return methods;
}
```

上面 this.advisorFactory.getAdvisors(factory); 方法主要做了：

1. 获取到不包含 @Pointcut 修饰的方法.
2. 调用 getAdvisor(); 构建出 Advisor.
3. 将构建好的 Advisor 放入 List<Advisor> advisors = new ArrayList<>();中.

```java
@Override
@Nullable
public Advisor getAdvisor(Method candidateAdviceMethod, MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrderInAspect, String aspectName) {
	// 进行校验
    validate(aspectInstanceFactory.getAspectMetadata().getAspectClass());
	// 获取切点 Pointcut 实例 AspectJExpressionPointcut
    AspectJExpressionPointcut expressionPointcut = getPointcut(
        candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
    if (expressionPointcut == null) {
        return null;
    }
	// 创建一个 Advisor 实例 InstantiationModelAwarePointcutAdvisorImpl
    return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod, this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
}
```

获取切点 Pointcut

```java
@Nullable
private AspectJExpressionPointcut getPointcut(Method candidateAdviceMethod, Class<?> candidateAspectClass) {
    // 在给定方法上查找并返回第一个 AspectJ 注解
    AspectJAnnotation<?> aspectJAnnotation =
        AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
    if (aspectJAnnotation == null) {
        return null;
    }
	// 创建一个 AspectJExpressionPointcut 对象
    AspectJExpressionPointcut ajexp =
        new AspectJExpressionPointcut(candidateAspectClass, new String[0], new Class<?>[0]);
    // 设置切点表达式
    ajexp.setExpression(aspectJAnnotation.getPointcutExpression());
    if (this.beanFactory != null) {
        ajexp.setBeanFactory(this.beanFactory);
    }
    return ajexp;
}

/*
 * AspectJ 相关注释 Pointcut.class 在第一个
 */
private static final Class<?>[] ASPECTJ_ANNOTATION_CLASSES = new Class<?>[] {
			Pointcut.class, Around.class, Before.class, After.class, AfterReturning.class, AfterThrowing.class};

/*
 * 在给定方法上查找并返回第一个 AspectJ 注解
 */
@Nullable
protected static AspectJAnnotation<?> findAspectJAnnotationOnMethod(Method method) {
    for (Class<?> clazz : ASPECTJ_ANNOTATION_CLASSES) {
        AspectJAnnotation<?> foundAnnotation = findAnnotation(method, (Class<Annotation>) clazz);
        if (foundAnnotation != null) {
            return foundAnnotation;
        }
    }
    return null;
}
```

接下来让我们回到 return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod, this, aspectInstanceFactory, declarationOrderInAspect, aspectName);这个代码前查看是怎么创建的.

```java
public InstantiationModelAwarePointcutAdvisorImpl(AspectJExpressionPointcut declaredPointcut,
			Method aspectJAdviceMethod, AspectJAdvisorFactory aspectJAdvisorFactory,
			MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {

    this.declaredPointcut = declaredPointcut;
    this.declaringClass = aspectJAdviceMethod.getDeclaringClass();
    this.methodName = aspectJAdviceMethod.getName();
    this.parameterTypes = aspectJAdviceMethod.getParameterTypes();
    this.aspectJAdviceMethod = aspectJAdviceMethod;
    this.aspectJAdvisorFactory = aspectJAdvisorFactory;
    this.aspectInstanceFactory = aspectInstanceFactory;
    this.declarationOrder = declarationOrder;
    this.aspectName = aspectName;
	// Aspect 是否需要延迟初始化
    if (aspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
        Pointcut preInstantiationPointcut = Pointcuts.union(
            aspectInstanceFactory.getAspectMetadata().getPerClausePointcut(), this.declaredPointcut);
        this.pointcut = new PerTargetInstantiationModelPointcut(
            this.declaredPointcut, preInstantiationPointcut, aspectInstanceFactory);
        this.lazy = true;
    }
    else {
        this.pointcut = this.declaredPointcut;
        this.lazy = false;
        // 解析 Advice
        this.instantiatedAdvice = instantiateAdvice(this.declaredPointcut);
    }
}
```

我们看到比较重点的方法就是在最后一行，其他的我们不需要取关心直接看最后一行的方法.

```java
private Advice instantiateAdvice(AspectJExpressionPointcut pointcut) {
    // 调用 this.aspectJAdvisorFactory.getAdvice
    Advice advice = this.aspectJAdvisorFactory.getAdvice(this.aspectJAdviceMethod, pointcut, this.aspectInstanceFactory, this.declarationOrder, this.aspectName);
    return (advice != null ? advice : EMPTY_ADVICE);
}

@Override
@Nullable
public Advice getAdvice(Method candidateAdviceMethod, AspectJExpressionPointcut expressionPointcut, MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {
	// 获取到 AspectClass
    Class<?> candidateAspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    // 校验
    validate(candidateAspectClass);
    // 找到第一个 AspectJ 相关注解
    AspectJAnnotation<?> aspectJAnnotation =
        AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
    if (aspectJAnnotation == null) {
        return null;
    }
	// ...省略判断和日志打印
    
    // 获取方法上的注解类型并实例化对应的实例
    AbstractAspectJAdvice springAdvice;
    switch (aspectJAnnotation.getAnnotationType()) {
        case AtPointcut:
            return null;
        case AtAround:
            springAdvice = new AspectJAroundAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtBefore:
            springAdvice = new AspectJMethodBeforeAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfter:
            springAdvice = new AspectJAfterAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfterReturning:
            springAdvice = new AspectJAfterReturningAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterReturning afterReturningAnnotation = (AfterReturning) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterReturningAnnotation.returning())) {
                springAdvice.setReturningName(afterReturningAnnotation.returning());
            }
            break;
        case AtAfterThrowing:
            springAdvice = new AspectJAfterThrowingAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterThrowing afterThrowingAnnotation = (AfterThrowing) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterThrowingAnnotation.throwing())) {
                springAdvice.setThrowingName(afterThrowingAnnotation.throwing());
            }
            break;
        default:
            throw new UnsupportedOperationException(
                "Unsupported advice type on method: " + candidateAdviceMethod);
    }
    springAdvice.setAspectName(aspectName);
    springAdvice.setDeclarationOrder(declarationOrder);
    // 获取方法上的参数名
    String[] argNames = this.parameterNameDiscoverer.getParameterNames(candidateAdviceMethod);
    if (argNames != null) {
        // 设置参数名
        springAdvice.setArgumentNamesFromStringArray(argNames);
    }
    springAdvice.calculateArgumentBindings();
    return springAdvice;
}
```

上面的代码比较清晰就是拿之前取到的方法创建出对应的通知对象，我们就拿第一个 AspectJAroundAdvice 通知对象来看看.

```java
public class AspectJAroundAdvice extends AbstractAspectJAdvice implements MethodInterceptor, Serializable {
	
    // 构造器
	public AspectJAroundAdvice(
			Method aspectJAroundAdviceMethod, AspectJExpressionPointcut pointcut, AspectInstanceFactory aif) {

		super(aspectJAroundAdviceMethod, pointcut, aif);
	}

	@Override
	public boolean isBeforeAdvice() {
		return false;
	}

	@Override
	public boolean isAfterAdvice() {
		return false;
	}

	@Override
	protected boolean supportsProceedingJoinPoint() {
		return true;
	}

	@Override
	public Object invoke(MethodInvocation mi) throws Throwable {
        // 判断是否是 ProxyMethodInvocation 类型
		if (!(mi instanceof ProxyMethodInvocation)) {
			throw new IllegalStateException("MethodInvocation is not a Spring ProxyMethodInvocation: " + mi);
		}
        // 强转 ProxyMethodInvocation
		ProxyMethodInvocation pmi = (ProxyMethodInvocation) mi;
        // 返回 MethodInvocationProceedingJoinPoint 实例
		ProceedingJoinPoint pjp = lazyGetProceedingJoinPoint(pmi);
		JoinPointMatch jpm = getJoinPointMatch(pmi);
        // 进行反射调用对应的 Method
		return invokeAdviceMethod(pjp, jpm, null, null);
	}

    /*
     * 返回当前调用的 ProceedingJoinPoint
     */
	protected ProceedingJoinPoint lazyGetProceedingJoinPoint(ProxyMethodInvocation rmi) {
		return new MethodInvocationProceedingJoinPoint(rmi);
	}
}
```

至此所有的通知器我们就已经获取到了，让我们在回到开始获取的地方，我们已经陷入到很深层次的调用现在返回.

```java
protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    // 匹配对应的通知器
    List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}
```

我们现在可以看 findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);这个方法是怎么做的.

```java
protected List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> beanClass, String beanName) {
    // 将 beanName 放入 ProxyCreationContext 的 ThreadLocal 中
    ProxyCreationContext.setCurrentProxiedBeanName(beanName);
    try {
        return AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass);
    }
    finally {
        ProxyCreationContext.setCurrentProxiedBeanName(null);
    }
}

public static List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> clazz) {
    if (candidateAdvisors.isEmpty()) {
        return candidateAdvisors;
    }
    List<Advisor> eligibleAdvisors = new ArrayList<>();
    for (Advisor candidate : candidateAdvisors) {
        // 首先将 IntroductionAdvisor 类型的通知获取到
        if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
            eligibleAdvisors.add(candidate);
        }
    }
    boolean hasIntroductions = !eligibleAdvisors.isEmpty();
    for (Advisor candidate : candidateAdvisors) {
        if (candidate instanceof IntroductionAdvisor) {
            continue;
        }
        // 再将普通的通知器获取到
        if (canApply(candidate, clazz, hasIntroductions)) {
            eligibleAdvisors.add(candidate);
        }
    }
    return eligibleAdvisors;
}

public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
    Assert.notNull(pc, "Pointcut must not be null");
    if (!pc.getClassFilter().matches(targetClass)) {
        return false;
    }
	// 获取匹配器 MethodMatcher
    MethodMatcher methodMatcher = pc.getMethodMatcher();
    if (methodMatcher == MethodMatcher.TRUE) {
        return true;
    }

    IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
    if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
        introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
    }

    Set<Class<?>> classes = new LinkedHashSet<>();
    if (!Proxy.isProxyClass(targetClass)) {
        // 将目标类的 Class 对象放入进去
        classes.add(ClassUtils.getUserClass(targetClass));
    }
    // 获取到目标类的所有接口信息
    classes.addAll(ClassUtils.getAllInterfacesForClassAsSet(targetClass));

    for (Class<?> clazz : classes) {
        // 获取目标类及父类所有方法
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
        for (Method method : methods) {
            // 对目标类的方法进行通知器匹配
            if (introductionAwareMethodMatcher != null ?
                introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions) :
                methodMatcher.matches(method, targetClass)) {
                return true;
            }
        }
    }
    return false;
}
```

这样目标类对应的所有通知就已经从所有的通知器中被筛选了出来.接下来我们返回继续往下看.

```java
protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    // 获取所有通知器——我们已经看完
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
     // 匹配对应的通知器——我们已经看完
    List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
    // 对通知器进行扩展
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}
```

我们继续来看 extendAdvisors(eligibleAdvisors);这个方法.

```java
@Override
protected void extendAdvisors(List<Advisor> candidateAdvisors) {
    // 调用 makeAdvisorChainAspectJCapableIfNecessary
    AspectJProxyUtils.makeAdvisorChainAspectJCapableIfNecessary(candidateAdvisors);
}

public static boolean makeAdvisorChainAspectJCapableIfNecessary(List<Advisor> advisors) {
    // 如果通知器为空就忽略
    if (!advisors.isEmpty()) {
        boolean foundAspectJAdvice = false;
        for (Advisor advisor : advisors) {
            // 判断每个通知器是否包含 AspectJ advice
            if (isAspectJAdvice(advisor)) {
                foundAspectJAdvice = true;
            }
        }
        // 将 ExposeInvocationInterceptor.ADVISOR 放在第一位返回
        if (foundAspectJAdvice && !advisors.contains(ExposeInvocationInterceptor.ADVISOR)) {
            advisors.add(0, ExposeInvocationInterceptor.ADVISOR);
            return true;
        }
    }
    return false;
}

private static boolean isAspectJAdvice(Advisor advisor) {
    return (advisor instanceof InstantiationModelAwarePointcutAdvisor ||
            advisor.getAdvice() instanceof AbstractAspectJAdvice ||
            (advisor instanceof PointcutAdvisor &&
             ((PointcutAdvisor) advisor).getPointcut() instanceof AspectJExpressionPointcut));
}
```

### 创建代理对象

通知器也都获取到了，那么该创建对应的代理对象了，我们回到 org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#wrapIfNecessary 中.

```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }

    // 获取通知器
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        // 创建代理对象
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }

    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```

接下来我们可以继续往下跟踪来看看 createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));

```java
protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource) {

    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        // 在对应的 BeanDefinition 添加属性
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }
	// 创建一个 ProxyFactory
    ProxyFactory proxyFactory = new ProxyFactory();
    // 将当前类对象放入进去当前是 AbstractAutoProxyCreator
    proxyFactory.copyFrom(this);
	// 因为刚 new 出来 ProxyFactory proxyTargetClass 为 false
    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }
        else {
            // 获取出目标类的接口信息放入 proxyFactory 中
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }
	// 将 Object[] 类型的 specificInterceptors 转换成 Advisor 类型的数组
    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    proxyFactory.addAdvisors(advisors);
    proxyFactory.setTargetSource(targetSource);
    customizeProxyFactory(proxyFactory);
	// 是否冻结
    proxyFactory.setFrozen(this.freezeProxy);
    // advisorsPreFiltered() 默认返回 true
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }
	// 创建动态代理
    return proxyFactory.getProxy(getProxyClassLoader());
}

/*
 * 获取出目标类的接口信息
 */
protected void evaluateProxyInterfaces(Class<?> beanClass, ProxyFactory proxyFactory) {
    // 获取所有接口 Class
    Class<?>[] targetInterfaces = ClassUtils.getAllInterfacesForClass(beanClass, getProxyClassLoader());
    boolean hasReasonableProxyInterface = false;
    for (Class<?> ifc : targetInterfaces) {
        // 判断接口是不是指定的几种类型
        if (!isConfigurationCallbackInterface(ifc) && !isInternalLanguageInterface(ifc) &&
            ifc.getMethods().length > 0) {
            hasReasonableProxyInterface = true;
            break;
        }
    }
    if (hasReasonableProxyInterface) {
		// 遍历获取的接口信息放入 proxyFactory 中
        for (Class<?> ifc : targetInterfaces) {
            proxyFactory.addInterface(ifc);
        }
    }
    else {
        proxyFactory.setProxyTargetClass(true);
    }
}

protected Advisor[] buildAdvisors(@Nullable String beanName, @Nullable Object[] specificInterceptors) {
    // 将拦截器转成通知器对象，我本地模拟的是没有拦截器的情况
    Advisor[] commonInterceptors = resolveInterceptorNames();
	// 新建一个 List
    List<Object> allInterceptors = new ArrayList<>();
    if (specificInterceptors != null) {
        // 将参数转成 List
        allInterceptors.addAll(Arrays.asList(specificInterceptors));
        // commonInterceptors 为空
        if (commonInterceptors.length > 0) {
            if (this.applyCommonInterceptorsFirst) {
                allInterceptors.addAll(0, Arrays.asList(commonInterceptors));
            }
            else {
                allInterceptors.addAll(Arrays.asList(commonInterceptors));
            }
        }
    }
    // 转成 Advisor 类型的数组并返回
    Advisor[] advisors = new Advisor[allInterceptors.size()];
    for (int i = 0; i < allInterceptors.size(); i++) {
        advisors[i] = this.advisorAdapterRegistry.wrap(allInterceptors.get(i));
    }
    return advisors;
}
```

上面代码获取到了目标类的接口信息与通知器信息都放入了 proxyFactory 对象中，下面就要进入 proxyFactory.getProxy(getProxyClassLoader()); 方法了.

```java
public Object getProxy(@Nullable ClassLoader classLoader) {
    // 首先创建一个代理再执行 getProxy()
    return createAopProxy().getProxy(classLoader);
}

protected final synchronized AopProxy createAopProxy() {
    if (!this.active) {
        activate();
    }
    // 获取到 AopProxyFactory 再调用 createAopProxy();
    return getAopProxyFactory().createAopProxy(this);
}

@Override
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("...");
        }
        // 如果有接口的话也优先 JDK 代理
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        // 否则使用 cglib 代理
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        // 默认返回 JDK 代理
        return new JdkDynamicAopProxy(config);
    }
}
```

看到了上面的代码发现离创建代理对象越来越近了.我们继续跟踪 JdkDynamicAopProxy.getProxy(@Nullable ClassLoader classLoader);

```java
@Override
public Object getProxy(@Nullable ClassLoader classLoader) {
    if (logger.isTraceEnabled()) {
        logger.trace("Creating JDK dynamic proxy: " + this.advised.getTargetSource());
    }
    // 返回接口 Class 数组
    Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
    // 查找 Equals 和 HashCode 方法存在则设置对应属性为 true
    findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
    // 通过 Proxy.newProxyInstance 动态编译生成代理对象返回
    return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```

上面代码，请把目光移至最后一行有效代码上，会发现 JdkDynamicAopProxy 最终调用 Proxy.newProxyInstance 方法创建代理对象，这下是终于创建完了代理对象返回.

## 结束

由于时间精力有限 aop 就分析到这，如有不对的地方，请大家指出共同进步，感谢观看。