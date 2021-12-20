# java.lang -> Boolean

## 是什么

Boolean 类是将 boolean 基本类型进行包装。类型为 Boolean 的对象包含一个单一属性 value，其类型为 boolean。
此外还提供了许多将 boolean 转换为 String、String 转换为 boolean，以及其他一些方法。

## 构造器

有两个构造器

```java
public Class Boolean {
    public static final Boolean TRUE = new Boolean(true);
    
    public static final Boolean FALSE = new Boolean(false);
    
    public static final Class<Boolean> TYPE = (Class<Boolean>) Class.getPrimitiveClass("boolean");
    
    private final boolean value;

    public Boolean(boolean value) {
        this.value = value;
    }

    public Boolean(String s) {
        this(parseBoolean(s));
    }
}
```

我们可以看到在内部维护了一个 boolean 类型的 value，在我们调用构造器的时候会将 value 给赋值。

还可以看到内部还有三个常量其中两个是 TRUE、FALSE 分别代表布尔值的两个状态，这样我们在使用的时候就可以直接用而不需要去构建一个。

还有一个 TYPE 是接收了 Class.getPrimitiveClass("boolean"); 的返回值，我们跟进去可以看到 getPrimitiveClass 是一个 native 方法。

```java
static native Class<?> getPrimitiveClass(String name);
```

我们直接查看 openJDK 中对应的 Class.c 文件可以发现对应的方法 Java_java_lang_Class_getPrimitiveClass 

```c++
JNIEXPORT jclass JNICALL
Java_java_lang_Class_getPrimitiveClass(JNIEnv *env,
                                       jclass cls,
                                       jstring name)
{
    const char *utfName;
    jclass result;

    if (name == NULL) {
        JNU_ThrowNullPointerException(env, 0);
        return NULL;
    }

    utfName = (*env)->GetStringUTFChars(env, name, 0);
    if (utfName == 0)
        return NULL;

    result = JVM_FindPrimitiveClass(env, utfName);

    (*env)->ReleaseStringUTFChars(env, name, utfName);

    return result;
}
```

我们可以看到 JVM 会根据我们传递的 "boolean" name 字符串来调用 JVM_FindPrimitiveClass 来获取到 jclass，返回到 java 层则为 Class<Boolean>.

并且如果常量 TYPE 在执行到 toString() 时还是会调用到 native 方法。如下

```java
public String toString() {
    return (isInterface() ? "interface " : (isPrimitive() ? "" : "class "))
        + getName();
}

public String getName() {
    String name = this.name;
    if (name == null)
        // 会调用到 getName0();
        this.name = name = getName0();
    return name;
}

private native String getName0();
```

我们会发现最终会调用到 getName0() 这个 native 方法。所以我们继续去看 C 文件

```c++

static JNINativeMethod methods[] = {
    {"getName0",         "()" STR,          (void *)&JVM_GetClassName},
    {"getSuperclass",    "()" CLS,          NULL},
    {"getInterfaces0",   "()[" CLS,         (void *)&JVM_GetClassInterfaces},
    {"getClassLoader0",  "()" JCL,          (void *)&JVM_GetClassLoader},
    {"isInterface",      "()Z",             (void *)&JVM_IsInterface},
    {"getSigners",       "()[" OBJ,         (void *)&JVM_GetClassSigners},
    {"setSigners",       "([" OBJ ")V",     (void *)&JVM_SetClassSigners},
    {"isArray",          "()Z",             (void *)&JVM_IsArrayClass},
    {"isPrimitive",      "()Z",             (void *)&JVM_IsPrimitiveClass},
    {"getComponentType", "()" CLS,          (void *)&JVM_GetComponentType},
    {"getModifiers",     "()I",             (void *)&JVM_GetClassModifiers},
    {"getDeclaredFields0","(Z)[" FLD,       (void *)&JVM_GetClassDeclaredFields},
    {"getDeclaredMethods0","(Z)[" MHD,      (void *)&JVM_GetClassDeclaredMethods},
    {"getDeclaredConstructors0","(Z)[" CTR, (void *)&JVM_GetClassDeclaredConstructors},
    {"getProtectionDomain0", "()" PD,       (void *)&JVM_GetProtectionDomain},
    {"getDeclaredClasses0",  "()[" CLS,      (void *)&JVM_GetDeclaredClasses},
    {"getDeclaringClass0",   "()" CLS,      (void *)&JVM_GetDeclaringClass},
    {"getGenericSignature0", "()" STR,      (void *)&JVM_GetClassSignature},
    {"getRawAnnotations",      "()" BA,        (void *)&JVM_GetClassAnnotations},
    {"getConstantPool",     "()" CPL,       (void *)&JVM_GetClassConstantPool},
    {"desiredAssertionStatus0","("CLS")Z",(void *)&JVM_DesiredAssertionStatus},
    {"getEnclosingMethod0", "()[" OBJ,      (void *)&JVM_GetEnclosingMethodInfo},
    {"getRawTypeAnnotations", "()" BA,      (void *)&JVM_GetClassTypeAnnotations},
};
```

我们可以看到这个数组的第一行 getName0 就被定义为了 JVM_GetClassName 的函数。大家还记得当时 Object 文章里类似的地方吗，是的没错 Class 类中也有 registerNatives 的本地方法来注册对应的方法。

```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type,
    AnnotatedElement {

    private static native void registerNatives();
    static {
        registerNatives();
    }
}
```

好了我们继续来看 getName0 被定义为对应的 JVM_GetClassName 函数后。最终的 JVM_GetClassName 实现函数是下面展示的.

```c++
JVM_ENTRY(jstring, JVM_GetClassName(JNIEnv *env, jclass cls))
  assert (cls != NULL, "illegal class");
  JVMWrapper("JVM_GetClassName");
  JvmtiVMObjectAllocEventCollector oam;
  ResourceMark rm(THREAD);
  const char* name;
  if (java_lang_Class::is_primitive(JNIHandles::resolve(cls))) {
    // 注意这一行函数里会去 type2name_tab 数组里去寻找
    name = type2name(java_lang_Class::primitive_type(JNIHandles::resolve(cls)));
  } else {
    // Consider caching interned string in Klass
    Klass* k = java_lang_Class::as_Klass(JNIHandles::resolve(cls));
    assert(k->is_klass(), "just checking");
    name = k->external_name();
  }
  oop result = StringTable::intern((char*) name, CHECK_NULL);
  return (jstring) JNIHandles::make_local(env, result);
JVM_END
```

我们可以看到第八行会去调用 type2name 函数，最终根据一个数组获得对应的名称，比如这里下标为4，则名称为”boolean”。

```c++
const char* type2name_tab[T_CONFLICT+1] = {
    NULL, NULL, NULL, NULL,
    "boolean",
    "char",
    "float",
    "double",
    "byte",
    "short",
    "int",
    "long",
    "object",
    "array",
    "void",
    "*address*",
    "*narrowoop*",
    "*conflict*"
};
```

## 重要方法

### parseBoolean

```java
public static boolean parseBoolean(String s) {
    // 判断参数 s 是否不为空并且是 "true" 字符串
    return ((s != null) && s.equalsIgnoreCase("true"));
}
```

### valueOf

```java
public static Boolean valueOf(boolean b) {
    // 三目表达式没什么好说的
    return (b ? TRUE : FALSE);
}
```

### toString

```java
public static String toString(boolean b) {
    // 三目表达式没什么好说的
    return b ? "true" : "false";
}

public String toString() {
    return value ? "true" : "false";
}
```

### hashCode

```java
public static int hashCode(boolean value) {
    // 即 true 返回 1231 而 false 返回 1237
    return value ? 1231 : 1237;
}
```

### equals

```java
public boolean equals(Object obj) {
    // 方法就是先判断是不是从 Boolean 实例化出来的，然后再继续比较是不是相等。
    if (obj instanceof Boolean) {
        return value == ((Boolean)obj).booleanValue();
    }
    return false;
}
```

### getBoolean

```java
public static boolean getBoolean(String name) {
    boolean result = false;
    try {
        // 从系统环境中获取指定 name 如果失败则返回 false
        result = parseBoolean(System.getProperty(name));
    } catch (IllegalArgumentException | NullPointerException e) {
    }
    return result;
}
```

### compareTo

```java
public int compareTo(Boolean b) {
    return compare(this.value, b.value);
}

public static int compare(boolean x, boolean y) {
    // 如果 boolean 相等则返回 0 否则 x 为真返回 1 为假返回 -1
    return (x == y) ? 0 : (x ? 1 : -1);
}
```

### 逻辑运算

```java
public static boolean logicalAnd(boolean a, boolean b) {
    return a && b;
}

public static boolean logicalOr(boolean a, boolean b) {
    return a || b;
}

public static boolean logicalXor(boolean a, boolean b) {
    return a ^ b;
}
```

上面三个方法也没啥好说的大家自己看。