# Object

## registerNatives() - Native

static 块内调用，在类加载的时候将 Object中 中的 Native 方法绑定。

## getClass() - Native

返回运行时的类型。

## hashCode - Native

## equals

默认 return (this == obj)

## clone - Native

创建并克隆对象。

子类未继承 Cloneable，抛出 CloneNotSupportedException 。

## toString

返回对象的代表字符串。

## notify

唤醒 wait 的线程。

## notifyAll

## wait

释放对象锁。

## finalize

析构函数。

当垃圾回收器要宣告一个对象死亡时，至少要经过两次标记过程：如果对象在进行可达性分析后发现没有和GC Roots相连接的引用链，就会被第一次标记，并且判断是否执行finalizer( )方法，如果对象覆盖finalizer( )方法且未被虚拟机调用过，那么这个对象会被放置在F-Queue队列中，并在稍后由一个虚拟机自动建立的低优先级的Finalizer线程区执行触发finalizer( )方法，但不承诺等待其运行结束。

finalization的目的：对象逃脱死亡的最后一次机会。（只要重新与引用链上的任何一个对象建立关联即可。）但是不建议使用，运行代价高昂，不确定性大，且无法保证各个对象的调用顺序。可用try-finally或其他替代。

尽量避免使用finalize():

finalize()不一定会被调用, 因为java的垃圾回收器的特性就决定了它不一定会被调用
就算finalize()函数被调用, 它被调用的时间充满了不确定性, 因为程序中其他线程的优先级远远高于执行finalize（）函数线程的优先级。也许等到finalize()被调用, 数据库的连接池或者文件句柄早就耗尽了.
如果一种未被捕获的异常在使用finalize方法时被抛出，这个异常不会被捕获，finalize方法的终结过程也会终止，造成对象出于破坏的状态。被破坏的对象又很可能导致部分资源无法被回收, 造成浪费.
finalize()和垃圾回收器的运行本身就要耗费资源, 也许会导致程序的暂时停止.

## 其他事项

### equals 和 == 操作符

默认情况下也就是从超类 Object 继承而来的 equals 方法与‘==’是完全等价的，比较的都是对象的内存地址，但我们可以重写 equals 方法，使其按照我们的需求的方式进行比较，如 String 类重写了 equals 方法，使其比较的是字符的序列，而不再是内存地址。

### hasCode and equals

1. HashCode 的存在主要是用于查找的快捷性，如 Hashtable，HashMap 等，HashCode 经常用于确定对象的存储地址；

2. 如果两个对象相同， equals方法一定返回 true，并且这两个对象的HashCode一定相同；

3. 两个对象的 HashCode 相同，并不一定表示两个对象就相同，即 equals()不一定为 true，只能说明这两个对象在一个散列存储结构中。

4. 如果对象的 equals 方法被重写，那么对象的 HashCode 也尽量重写。

### 推荐写法

#### equals推荐写法

* 自反性。对于任何非 null 的引用值x，x.equals(x)应返回true。

* 对称性。对于任何非 null 的引用值x与y，当且仅当：y.equals(x)返回true时，x.equals(y)才返回true。

* 传递性。对于任何非 null 的引用值x、y与z，如果y.equals(x)返回true，y.equals(z)返回true，那么x.equals(z)也应返回true。

* 一致性。对于任何非 null 的引用值x与y，假设对象上equals比较中的信息没有被修改，则多次调用x.equals(y)始终返回true或者始终返回false。

* 对于任何非空引用值x，x.equal(null)应返回false。

```equals推荐写法
@Override
public boolean equals(Object otherObject) {
    // 1 相当于hashcode相等，具有相同的散列数
    if (this == otherObject) {
        return true;
    }
    // 2 如果另一个对象不是当前的 class 类型，那必然是不相等的
    if (!(o instanceof MyType)) {
        return false;
    }

    // 3 判断对象的私有成员是否相等。
    MyType other = (MyType) otherObject;
    return primitiveField == other.primitiveField &&
            referenceField.equals(other.referenceField) &&
            (nullableField == null ? other.nullableField == null : nullableField.equals(other.nullableField));
}
```

#### hashCode推荐写法

```hashCode推荐写法
Override
public int hashCode() {
     // 初始值为一个非零数
    int result = 17;

    // 为每个成员变量计算值
    result = 31 * result + (booleanField ? 1 : 0);
    result = 31 * result + byteField;
    result = 31 * result + charField;
    result = 31 * result + shortField;
    result = 31 * result + intField;
    result = 31 * result + (int) (longField ^ (longField >>> 32));
    result = 31 * result + Float.floatToIntBits(floatField);
    long doubleFieldBits = Double.doubleToLongBits(doubleField);
    result = 31 * result + (int) (doubleFieldBits ^ (doubleFieldBits >>> 32));
    result = 31 * result + Arrays.hashCode(arrayField);
    result = 31 * result + referenceField.hashCode();
    result = 31 * result + (nullableReferenceField == null ? 0 : nullableReferenceField.hashCode());
    return result;
}
```

为什么hashCode的计算中要采用31这个数字做乘积？

1. 因为31为素数（只能被1和本身整除），在存储数据计算hash地址时，要尽量减少重复的hash值。所以在选择系数的时候要选择尽量长的系数并且让乘法尽量不要溢出的系数，因为如果计算出来的hash地址越大，所谓的“冲突”就越少，查找起来效率也会提高。
2. 31可以 由i*31== (i<<5)-1来表示，现在很多虚拟机里面都有做相关优化，使用31的原因可能是为了更好的分配hash地址，并且31只占用5bits！
3. 在Java乘法中如果数字相乘过大会导致溢出的问题，从而导致数据的丢失.而31则是素数（质数）而且不是很长的数字

#### toString推荐写法

```toString推荐写法
Override
public String toString() {
    return getClass().getName() + "[" +
        "primitiveField=" + primitiveField + ", " +
        "referenceField=" + referenceField + ", " +
        "arrayField=" + Arrays.toString(arrayField) + "]";
}
```
