# Android Lint

Android Lint是 Google 提供的 Android 静态代码检查工具。

## 基础概念

1. Issue：对应一个 Lint 规则
2. Detector：Issue 对应的检测器，每个Issue都要指定Detector
3. Scope：扫描的范围
4. Scanner：扫描器，一个 Issue 可以对应多个
5. IssueRegistry：Lint 规则加载入口

Lint 开发过程中最主要的工作就是实现 Scanner ：

Scanner 解析源码为 AST 的发展过程经历了三个大版本：

1. lombok-ast
2. IntelliJ 的 PSI （Program Structure Interface）
3. IntelliJ 的 UAST（Unified AST）

后两者都是IDEA中用于解析代码的一套API，可将文件的内容表示为特定编程语言中的元素的层级结构。
推荐使用最新的 UAST 作为扫描器的实现方式。

Android Lint 还在不稳定地迭代中，可以发现不同版本，API 可能存在不兼容性。官方许多 API 注释中也备注者不稳定的 API，不保证未来不会修改。

后文基于 **Lint 26.3.2 版本/ Gradle Plugin 3.3.2 版本**进行说明。

## Lint自定义规则

关注官方原生 Lint 实现库：

1. lint-api(Lint 工具集的一个封装，实现了一组API接口，用于启动 Lint)
2. lint-checks(实现内置的规则检测器)

实现自定义规则步骤：

### 1. 新建 自定义规则库

新建一个 Java Library 作为自定义规则库，对标原生 lint-checks 库。（所以具体规则的编写可参照 lint-checks 库）

### 2. gradle 引入 Lint 依赖

```gradle
compileOnly 'com.android.tools.lint:lint-api:26.3.2'
compileOnly 'com.android.tools.lint:lint-checks:26.3.2'
```

### 3. 新建类继承 IssuesRegister

新建一个XXXIssuesRegister类继承 IssuesRegister，作为 Lint 检测器入口。

同时在 build.gradle 配置指定该新建的类为 'Lint-Registry'。（指定类全限定名）

```gradle
jar {
    manifest {
        attributes 'Lint-Registry': 'com.xxx.xxx.XXXIssuesRegister'
    }
}
```

### 4. 编写单个自定义 Lint 规则

可参照 lint-checks 库里 XXXDetector 的实现。

新建一个 XXX Detector 继承自 Detector。可以实现一个或多个Scanner接口,选择实现哪种接口取决于你想要的扫描范围。

主要实现：

1. issue（规则） 定义
2. 检测实现

#### Issue 定义

- id : 唯一值，应当能简短描述当前问题。利用 Java 注解或者 XML 属性进行屏蔽时，使用的就是这个id。
- summary : 简短的总结，通常5-6个字符，描述问题而不是修复措施。
- explanation : 完整的问题解释和修复建议。
- category : 问题类别。详见下文详述部分。已有类别 Lint、Correctness (包含 Messages)、Security、Performance、Usability (包含 Icons, Typography)、Accessibility、Internationalization、Icons、Typography、Messages
- priority : 优先级。1-10的数字，10为最重要/最严重。
- severity : 严重级别：Fatal, Error, Warning, Informational, Ignore。
- Implementation : 为 Issue 和 Detector 提供映射关系，Detector 就是当前 Detector。声明扫描检测的范围 Scope，Scope用来描述  Detector需要分析时需要考虑的文件集，包括：Resource文件或目录、Java文件、Class文件。

#### 检测实现

1. 通过选择性实现 getApplicableXXX 系列方法，来决定规则的适用范围
2. 经过第一步缩小的适用范围进行具体规则的判断是否违反，判断是否进行 report，report 同时可以传入 LintFix 提供 quickFix。

注：在编写具体规则时会用到上文提到的 IntelliJ 的 UAST。

### 5. 测试自定义 Lint 规则使用

1. Demo 模块 引入 自定义规则库。

```gradle
lintChecks project(':XXX')
```

注：较新版本可能需要使用 lintPublish。这里阐述的版本还没有 lintPublish api。
2. 故意编写一个违反规则的样例。
3. 查看 Android Studio -> Preferences -> Editor -> Inspections -> Android -> Lint 下是否有自定义 Issue 规则。
4. 运行 gradle lint 任务，查看生成的 Lint 报告是否含有自定义的 Lint Issue。

### 6. Debug Lint

1. 先提前在要 Debug 的自定义 Lint 规则上打好断点
2. 执行 debug 命令

```bash
./gradlew --no-daemon -Dorg.gradle.debug=true lintDebug
```

注：Terminal 中会停留显示 > Starting Daemon，说明正在等在 attach debugger
3. 新建配置，Edit Configurations -> Add New Configurations -> Remote 使用默认参数确认，然后运行新建的Remote配置 Debug

此时切换到 Terminal ，你会发现正在跑 lint 任务，过一会断点就会命中。

## Git hook 实现提交到远程仓库前进行 Lint 检测

Lint 可以在提交前手动执行命令。为了避免遗忘或者强制执行，可以利用 Git Hook来实现。

.git/hooks文件夹下自带一些sample.

### 团队共享hooks

项目新建 hooks 文件夹，文件夹下编写 hooks 脚本(pre-commit)，并加入 git 追踪管理。

编写 gradle 任务，将项目根目录下的hooks脚本拷贝到 .git/hooks 目录下。拷贝时机为 preBuild 之后。

```gradle
task installGitHook(type: Copy) {
    from new File(rootProject.rootDir, 'hooks/pre-commit')
    into { new File(rootProject.rootDir, '.git/hooks') }
    fileMode 0777
}
subprojects {
    afterEvaluate {
        def preBuild = project.tasks.findByName('preBuild')
        if (preBuild != null) {
            println('instGitHook' + preBuild)
            preBuild.dependsOn installGitHook
        }
    }
}
```

注：hook文件返回 0 表示通过, 会进行下一步操作. 否则后面的操作会被放弃。如果不想跑hook可以加上 --no-verify。

### 踩坑

#### Lint Version 与 Gradle Version 有关

Gradle Plugin x.y.z
Lint 应该为 23+x.y.z

如果版本不对应，可能导致执行 gradle lint 任务扫描不出自定义的 Lint 规则。

---
参考文章：

官方：

- [通过 lint 检查改进代码](https://developer.android.com/studio/write/lint)
- [Android Studio Project Site Android Lint](http://tools.android.com/tips/lint)

美团系列：

- [Android自定义Lint实践](https://tech.meituan.com/2016/03/21/android-custom-lint.html)
- [Android自定义Lint实践2——改进原生Detector](https://tech.meituan.com/2017/03/09/android-custom-lint2.html)
- [美团外卖Android Lint代码检查实践](https://tech.meituan.com/2018/04/13/waimai-android-lint.html)
- [Kotlin代码检查在美团的探索与实践](https://tech.meituan.com/2018/07/05/kotlin-code-inspect.html)
- [Android静态代码扫描效率优化与实践](https://tech.meituan.com/2019/11/07/android-static-code-canning.html)

其他：

- [Android Lint代码检查实践](https://juejin.im/post/6861562664582119432#heading-22)
- [Lint增量扫描实践](https://juejin.im/post/6871128918611460110)
- [Android自定义Lint实践 （Custom Lint Rules & Lint Plugin](https://www.wanandroid.com/blog/show/2665)
- [Android Lint增量扫描实战纪要](https://juejin.im/entry/6844903517325361159)
- [okcheck](https://github.com/lingochamp/okcheck)