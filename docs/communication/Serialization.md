# 序列化

## Serializable

### 实现

* 继承Serializable接口
* 指定serialVersionUID（非必须），反序列化会检测该字段一致性，不一致抛异常（Java.io.InvalidClassExcrption）。

数据类结构兼容升级避免修改 serialVersionUID ，反序列化会失败。

完全不兼容升级（改变包名、修改成员变量类型等），需要修改 serialVersionUID，避免序列化混乱。

### 特点

* 使用较简单
* 大量IO操作，开销大

## Parcelable

### 实现

核心类 Parcel 内部封装了可序列化的数据，可以在Binder中自由传输。

需要实现的方法：

* describeContents() —— 内容描述
* writeToParcel() —— 序列化
* Parcel.Creator(createFromParcel) —— 反序列化

### 特点

* 使用较麻烦
* 效率高

## 不参与序列化

* 静态成员变量
* transient修饰的成员变量
