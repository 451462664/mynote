# java.lang -> Object

## 是什么

Object 类是类层次结构的根，是 Java 中唯一一个没有父类的类，Java 中所有对象包括数组都继承了 Object 类中的方法。

## 重要方法

```java
public class Object {
    private static native void registerNatives();
    static {
        registerNatives();
    }
}
```

首先引入眼帘的就是一个静态的 native 方法 registerNatives() 通过名字就能大概判断出来时注册本地方法的意思.我们可以通过 OpenJDK 中找到对应的文件来查看。

路径是：/jdk8/jdk/src/share/native/java/lang/Object.c

![1575440823927](D:\Typora\image\1575440823927.png)

之后就能看到 Object.c 的文件了我们点击打开后会发现 Java_java_lang_Object_registerNatives 这样的一个函数名称，这是为了让 JVM 找到本地函数，它们会被一定方式命名而我们的 private static native void registerNatives(); 对应的函数名就是 Java_java_lang_Object_registerNatives。

```c++

#include <stdio.h>
#include <signal.h>
#include <limits.h>

#include "jni.h"
#include "jni_util.h"
#include "jvm.h"

#include "java_lang_Object.h"

static JNINativeMethod methods[] = {
    {"hashCode",    "()I",                    (void *)&JVM_IHashCode},
    {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
    {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
    {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
    {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
};

JNIEXPORT void JNICALL
Java_java_lang_Object_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls,
                            methods, sizeof(methods)/sizeof(methods[0]));
}

JNIEXPORT jclass JNICALL
Java_java_lang_Object_getClass(JNIEnv *env, jobject this)
{
    if (this == NULL) {
        JNU_ThrowNullPointerException(env, NULL);
        return 0;
    } else {
        return (*env)->GetObjectClass(env, this);
    }
}
```

看了上面我们发现了有一个名字叫 methods 的数组里面不是定义了 Object 中的函数么。我们可以发现 registerNatives 函数的作用原来是将指定的本地方法绑定到指定的函数。将 hashCode 绑定到 JVM_IHashCode 等等。

而然我们发现了 Object 类中的所有方法都是 native 的并且通过上面的都已经注册了对应的 C++ 函数。

### public final native Class<?> getClass();

通过上图我们发现 getClass() 函数对应了 Java_java_lang_Object_getClass 方法在里面调用了 GetObjectClass。

### 一些非本地方法

#### equals

```java
public boolean equals(Object obj) {
    // 直接对比对象在堆内存中的物理地址
    return (this == obj);
}
```

#### toString

```java
public String toString() {
    // 调用本地方法 getClass() 后再获取 name + "@" + 根据 hashCode 的 16 机制无符号整数
    // Integer 会在包装类中讲
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

### 还有一个比较有意思的函数

```java
/**
 * timeout 毫秒
 * nanos 纳秒
 */
public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
	// 如果纳秒数小于 0 或者 > 999 999 则抛出异常
    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }
	// 如果那描述在 0 ~ 999 999 范围内
    if (nanos > 0) {
        // 重点来了考试要考,居然让毫秒数 + 1 这个感觉有点萌 很皮 皮一把很开心
        timeout++;
    }

    wait(timeout);
}
```