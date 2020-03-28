# Entrust

双亲委托模式：

某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子 ClassLoader 再加载一次。如果不使用这种委托模式，那我们就可以随时使用自定义的类来动态替代一些核心的类，存在非常大的安全隐患。

DexPathList：

DexClassLoader 重载了 findClass 方法，在加载类时会调用其内部的 DexPathList 去加载。DexPathList 是在构造 DexClassLoader 时生成的，其内部包含了 DexFile。

DexPathList.java：

```java
public Class findClass(String name) {
    for (Element element : dexElements) {
        DexFile dex = element.dexFile;
        if (dex != null) {
            Class clazz = dex.loadClassBinaryName(name, definingContext);
            if (clazz != null) {
                return clazz;
            }
        }
    }
    return null;
}
```
