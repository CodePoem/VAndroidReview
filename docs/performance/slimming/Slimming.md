# Apk体积优化

[ReDex](https://github.com/facebook/redex)是Facebook 的一个开源编译工具值得深入研究。

## 为什么要优化

- 下载转化率、升级转化率

- 推广成本

- 应用市场要求

## 优化手段

### 资源优化

[正常思路资源优化](/performance/slimming/ResourceSlimming.md)

[AndResGuard](/performance/slimming/AndResGuard.md)

#### 无用资源

##### 第一阶段：Lint

静态资源监测，开发阶段删除无用资源。

##### 第二阶段：shrinkResources

编译期“假删除”无用资源。

需要配合 ProGurad 的“minifyEnabled”功能同时使用。

缺陷：

- 没有处理 resources.arsc 文件。
- 没有真正删除资源文件。

由于 Java 代码需要用到资源的 R.java 文件，所以我们就需要把 R.java 提前准备好。在编译 Java 代码过程，已经根据 R.java 文件，直接将代码中资源的引用替换为常量。如果我们在这个过程强行把无用资源文件删除，resources.arsc 和 R.java 文件的资源 ID 都会改变（因为默认都是连续的），这个时候代码中已经替换过的引用就会出现资源错乱或者找不到的情况。因此系统为了避免发生这种情况，采用了折中的方法，并没有二次处理 resources.arsc 文件，只是仅仅把无用的 Drawable 和 Layout 文件替换为空文件。

##### 第三阶段（尚未实现）：realShrinkResources

真正实现无用资源的删除功能。

利用 resources.arsc 中 Public ID 的机制，实现非连续的资源 ID。keep 住保留资源的 ID，保证已经编译完的代码可以正常找到对应的资源。重写 resources.arsc 的方法会比资源混淆更加复杂，我们既要从这个文件中抹去所有的无用资源相关信息，还要 keep 住所有保留资源的 ID，相当于把整个文件都重写了。正因为异常复杂，所以目前 Android 还没有提供这套方案的完整实现。

Matrix 已实现。（待研究）

##### 第四阶段（待定）：Assets 无用资源

### 代码优化

#### ProGuard 优化

- 可以把非 exported 的四大组件以及 View 混淆。

- 去掉 Debug 信息或者去掉行号。

#### Dex 分包优化

- 将有调用关系的类和方法分配到同一个 Dex 中，即减少跨 Dex 的调用的情况，从而减少跨 Dex 调用造成的这些冗余信息。

#### Dex 压缩优化

### Native Library 优化

#### Library 压缩

#### Library 合并与裁剪

## 包体积监控

版本迭代过程，体积又会增大，因此需要进行监控。

### 大小监控

每个版本跟上一个版本包体积的对比情况。

### 依赖监控

开源库引入管控。

- 规则监控

规则监控也就是将包体积的监控抽象为规则，例如无用资源、大文件、重复文件、R 文件等。

[Matrix-ApkChecker](https://mp.weixin.qq.com/s/tP3dtK330oHW8QBUwGUDtA)
