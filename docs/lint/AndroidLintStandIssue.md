# Lint 标准 Issue

分类：

1. Correctness
2. Correctness:Message
3. Security
4. Performance
5. Usability:Typography
6. Usability:Icons
7. Usability
8. Accessibility
9. Internationalization

以下整理了部分常见Issue。

## Correctness（正确性）

- OnClick（onClick method does not exist）： Oncick属性没有对应的上下⽂ public ⽅法。

- WrongViewCast（Mismatched view type）：错误的View类型转换 >> 例如把⼀个 findViewId 为 TextView 的强转为 ImageView。

- RecyclerView（RecycleView Problem）：RecyclerView 中的数据有位置改变（⽐如删除）时⼀般不会重新调⽤ onBindViewHolder() ⽅法，除⾮这个元素不可⽤。也就是说 onBindViewHolder() ⽅法中的位置参数 position 不是实时更新的，所以在我们删除元素后，item的position 没有改变。为了实时获取元素的位置，RecyclerView 为我们提供了 ViewHolder.getAdapterPosition() ⽅法。

- ResourceName（Resource with Wrong Prefix）：多模块资源应该加上模块前缀，可以避免资源命名重复。

- DuplicateIds（Duplicate ids within a single layout）：在⼀个布局⽂件中，不应该有重复的ViewId。

- NestedScrolling（Nested scrolling widgets）：不应该使⽤嵌套的滚动控件，例如 ScrollView 嵌套。

- ListView ResourceAsColor（Should pass resolved color instead of resource id）：⽅法带有整型资源参数传递的，颜⾊资源传值应该是解析后的⼀个传递具体的 RGB 值，⽽不应该传递资源 id 。可以调⽤ getResources().getColor(resource) 解析颜⾊资源 id 。

- ScrollViewSize（ScrollView size validation）：ScrollView 的⼦元素必须设置 layout_width 或 layout_height 属性为 wrap_content ⽽不是 fill_parent。

- ApplySharedPref（Use apply() on SharedPreferences）：使⽤ SharedPreferences 存储数据的时候考虑使⽤ apply⽅法 ⽽不是 commit ⽅法， commit 是同步⽅法， apply 是异步⽅法。 Assert（Assertions）：断⾔在运⾏时不检查。正确的做法，可以举个例⼦

```java
assert speed > 0
```

可以⽤以下代码代替

```java
if (BuildConfig.DEBUG &&!(speed > 0)) {
     throw new AssertionError()
}
```

- CutPasteId（Likely cut & paste mistakes）：可能是剪切粘贴错误，⽐如，你原本想初始化两个字段 prev 和 next ，你复制了 findViewById(R.id.prev) 但忘记去更新第⼆个初始化参数为 R.id.next。

- DuplicateDefinition（Duplicate definitions of resources）：再同⼀个资源⽂件中重复定义资源。

- DuplicateIncludedIds（Duplicate ids across layouts combined with include tags）：在同⼀个布局⽂件中重复定义 include 布局的id InvalidPackage（Package not included in Android）：引⽤的 .jar 或者其他项⽬使⽤了不⽀持的 APIs 。

- InvalidR2Usage（Invalid usage of R2）：⾮法 R2 使⽤。

- NewApi（Calling new methods on older versions）：APIs 版本适配问题，调⽤某个⽅法需要考虑API版本兼容。代码中使⽤的某些 API ⾼于 Manifest 中的 MinSDK 。

- OldTargetApi（Target SDK attribute is not targeting latest version）： Target SDK 属性没有指向最新的版本。

- UnusedAttribute（Attribute unused on older versions）：在xml中设置的属性或者标签⽐你设置的 minSdkVersion 中使⽤的要新。

- StateListReachable（Unreachable state in a \<selector\>）：在⼀个选择器中，只有状态列表的最后⼀个⼦项能省略状态限定符，否则在这⼀项之后的所有⼦项都会被忽略，因为这⼀项会匹配所有的状态。

- UnknownIdInLayout（Reference to an id that is not in the current layout）：在布局中引⽤了未知的id。 AppCompatCustomView（Appcompat Custom Widgets）：⾃定义组件 应该继承⾃AppcompatXXX,以获得更好的兼容性。

- GradleDependency（Obsolete Gradle Dependency）：使⽤的库的版本不是最近的稳定版。

- ExtraText（Extraneous text in resource files）：在资源⽂件中应该只包含元素和属性，不包含⽂本内容。 PrivateResource（Using private resources）：私有的资源不应该被引⽤。

- SpUsage（Using dp instead of sp for text sizes）：使⽤ sp ⽽不是 dp 来描述⽂字⼤⼩。

- SwitchIntDef（Missing @IntDef in Switch）：switch 语句没有显式包含 @IntDef 定义的所有值。

- Orientation（Missing explicit orientation）：线性布局 LinearLayout 缺少描述布局⽅向 orientation 属性。

- PxUsage:（Using 'px' dimension）：尽量不直接使⽤ px 描述，px 在不同像素密度⼤⼩的⼿机上表现不⼀致。推荐使⽤ dp 。

- InflateParams（Layout Inflation without a Parent）：在inflate⼀个布局 的时候，⽗布局尽量不要传null，会导致⽗布局的布局参数失效。 Deprecated（Using deprecated resources）：废弃⽂件、资源等问题。

- DefaultLocale（Implied default locale in case conversion）：调⽤ toLowerCase 或 toUpperCase 明确转换后的字符集 ⽤ toUpperCase(Locale.US) 或 toUpperCase(Locale.getDefault()) 例如：在⼟⽿其地区，⼤写字⺟替换，i替换后不是I。

- SimpleDateFormat（Implied locale in date format）：调⽤ SimpleDateFormat ⽅法时注意第⼆个时区的⼊参。

## Correctness:Message（正确性：消息）

- StringFormatInvalid（Invalid format string ）：1.如果format模板字符串 是⾮法的，调⽤的String.format会产⽣运⾏时错误。 2.字符串本身含 有'%'的需要使⽤'%%'来避免替换 。 ps:推荐getString(R.string.A,B)这样的 的重载⽅法来替换。

- StringFormatMatches（String.format string doesn't match the XML format string）：如果⼀个格式化string资源有多个翻译版本，所有的翻译 都将在同⼀个数字编号参数位置使⽤同⼀种类型。如果使⽤的是Java的字 符串格式化，那么确保字符串匹配写在格式化的string资中。

- MissingTranslation（Incomplete translation）:不完整的翻译，翻译不 全。

- Typos（Spelling error）：拼写错误。

- ExtraTranslation（Extra translation ExtraTranslation）:缺少默认本地语⾔ 翻译。

- PluralsCandidate（Potential Plurals）：可能存在复数的情况。 举个例⼦：

```xml
<string name="try_again">Try again in %d seconds.</string>
```

推荐以下代码以适应复数的情景。

```xml
<plurals name="try_again">
    <item quantity="one">Try again in %d second</item> 
    <item quantity="other">Try again in %d seconds</item>
</plurals>
```

- StringFormatCount（Formatting argument types incomplete or inconsistent）：字符串格式化参数不完整或者类型不⼀致。

## Security（安全）

- JavascriptInterface（Missing @JavascriptInterface on methods）：与 Js交互的⽅法上缺少@JavascriptInterface注解。

- HardwareIds（Hardware Id Usage）：避免使⽤硬件标识符 推荐根据需 求使⽤对应的唯⼀标识符 可参考官⽅建议。 https://developer.android.com/training/articles/user-data-ids.html 

- SetJavaScriptEnabled（Using setJavaScriptEnabled）：如果你不确定 你的app是否要⽀持JS调⽤，最好不要调⽤SetJavaScriptEnabled⽅法。 

- TrustAllX509TrustManager（Insecure TLS/SSL trust manager）：不安 全的TLS/SSL信任管理器。

- WrongConstant（Incorrect constant）：确保当调⽤⽅法时只允许⼀组特 定的常数作为参数。

- ExportedContentProvider（Content provider does not require permission）：ContentProvider默认是暴露出去的，系统的任何应⽤能够 访问，如果要保护隐私数据，可以在Manifest⽂件中指定export属性为 false不暴露数据，或者提供⼀个权限来准许其他应⽤访问。

- ExportedReceiver（Receiver does not require permission）：同 ContentProvider⼀样，Receiver的export属性默认为true。如果要保护隐 私数据，可以在Manifest⽂件中指定export属性为false不暴露数据，或者 提供⼀个权限来准许其他应⽤访问。 

- ExportedService（Exported service does not require permission）： export属性默认为true。如果要保护隐私数据，可以在Manifest⽂件中指 定export属性为false不暴露数据，或者提供⼀个权限来准许其他应⽤访 问。 

- AllowBackup（AllowBackup/FullBackupContent Problems）： allowBackup属性决定了应⽤是否能够备份和恢复数据。

## Performance（性能）

- DrawAllocation（Memory allocations within drawing code）：避免在绘 制或布局过程中分配内存。频繁地申请对象会导致流畅的Ui被GC中断。通 常的做法是在绘制和布局之前申请需要的对象，在之后的绘图操作中重⽤ 这些对象。

- Recycle（Missing recycle() calls）：许多资源⽐如TypedArrays、 VelocityTrackers等应该在使⽤之后显式地调⽤回收⽅法。

- ObsoleteLayoutParam（Obsolete layout params）：在⼀个布局⽂件 中，某些属性是未定义的(常发⽣于更改⽗布局)。

- ObsoleteSdkInt（Obsolete SDK_INT Version Check）：此检查标志版本 检查是不必要的，因为MinSDKVersion（或周围已知的API级别）已经⾄ 少⾼达检查版本。

- StaticFieldLeak（Static Field Leaks）：静态字段可能存在内存泄漏。

- UseCompoundDrawables（Node can be replaced by a TextView with compound drawables）:线性布局中⼀个TextView和⼀个ImageView的组 合 可以⽤TextView的drawableTop, drawableLeft drawablePadding属性 代替。

- HandlerLeak（Handler reference leaks）：Handler存在内存泄漏。 Handler作为内部类，持有外部类的引⽤。

- MergeRootFrame（FrameLayout can be replaced with \<merge\> tag）：FrameLayout帧布局可以⽤ \<merge\> 标签代替

- UseSparseArrays（HashMap can be replaced with SparseArray）： HashMap可以⽤安卓特有的SparseArray代替，SparseArray⽐HashMap 更省内存。

- UseValueOf（Should use valueOf instead of new）：应该使⽤valueof⽅ 法⽽不是每次都new。例如，使⽤Integer.valueOf(42)⽽不是 Integer(42)，因为公共的整型对象会共⽤同⼀个实例。 DisableBaselineAlignment（Missing baselineAligned attribute）：当使 ⽤线性布局LineLayout来分配嵌套布局之间的空间⽐例 ，应该关闭基线对 ⻬齐属性baselineAligned以使布局计算速度更快。

- InefficientWeight（Inefficient layout weight）：当线性布局LineLayout中 只有⼀个简单的组件分配权重，将他的宽或⾼设置成0dp是更⾼效的，因 为这样它⾸先就不会测量⾃⼰的⼤⼩。

- NestedWeights（Nested layout weights）：布局权重会要求组件测量两 次，当⼀个⾮零权重的线性布局在另⼀个⾮零权重的线性布局中嵌套，测 量次数将呈⼏何数字增⻓长。

- Overdraw（Painting regions more than once ）：过度绘制问题。 UselessParent（Useless parent layout）：⼀个带有孩⼦节点但没有兄弟 姐妹节点的布局，它既不是scrollview或者根布局，并且也没有背景，那 么它是可以移除的。 

- UnusedResources（Unused resources ）：未使⽤资源。 UselessLeaf（Useless leaf layout）：⼀个没有孩⼦节点且没有设置背景 的布局是可以被移除的，这样能减少布局层级。 TooManyViews（Layout has too many views）：在⼀个布局中使⽤太多 的View会影响性能，考虑使⽤⼀些技巧减少View的数量。最⼤数量默认是 80，你也可以⾃定义ANDROID_LINT_MAX_VIEW_COUNT环境变量来修改 这个默认配置。

- UnusedNamespace（Unused namespace）：存在未使⽤的命名空间。

## Usability:Typography（可⽤性：排版）

- TypographyEllipsis（Ellipsis string can be replaced with ellipsis character）：字符串 "..."应该⽤专⽤的省略符(…, …)。 TypographyFractions（Fraction string can be replaced with fraction character ）：某些字符串, ⽐如 1/2, 和 1/4, 应该⽤专⻔门的字符来表示， ⽐如 ½ (½) 和 ¼ (¼)。

- TypographyDashes（Hyphen can be replaced with dash ）：特殊字符 需⽤编码代替：“–”需要⽤“–” （&#8211）；“—”需要⽤“—” （&#8212） ，“-”这个是连字符 “–”代表范围 “—”是破折号。在有范围 和破折的⽂案场景，请不要⽤连字符 这使你的APP更优雅。

## Usability:Icons（可⽤性：图标）

- IconColors（Icon colors do not follow the recommended visual style）：图标没有遵守推荐的设计⻛风格。[详情](https://developer.android.com/guide/practices/ui_guidelines/icon_design.html)

- IconDipSize（Icon density-independent size validation）：图标不同分 辨率密度⽂件夹放置有误。

- IconDuplicatesConfig（Identical bitmaps across various configurations）：相同像素⼤⼩的图标放在了不同分辨率密度⽂件夹 下。

- IconLocation（Image defined in density-independent drawable folder）：图标如果真的是密度⽆关的，推荐放在drawable-nodpi⽂件夹 下。

- IconDensities（Icon densities validation）：图标对于不同分辨率密度的 ⽂件夹没有覆盖完全。Lint默认不检测低密度分辨率⽂件夹ldpi，如果想要 检测，可以设置环境变量ANDROID_LINT_INCLUDE_LDPI=true。

- IconDuplicates（Duplicated icons under different names Usability ）： 图标相同，仅仅是使⽤了不同的命名。

## Usability（可⽤性）

- ButtonOrder（Button order）：对话框左侧按钮应该是代表取消含义的，右侧按钮应该是代表确认含义的。

- ButtonStyle（Button should be borderless）：按钮应该是⽆边界的。参考[点击](https://material.io/guidelines/components/buttons.html#buttonsdropdown-buttons)

- GoogleAppIndexingWarning（ Missing support for Firebase App Indexing）：将URL添加到⾕歌索引中，以获得安装和流量。从⾕歌搜索 到你的应⽤程序。

- TextFields（Missing inputType or hint）：缺少 inputType 属性或者 hint 属性，这两个属性是引导⽤户输⼊程序期望的值类型。

- ViewConstructor（Missing View constructors for XML inflation）：⾃定义 View 不能被布局编辑器识别。

- SmallSp（Text size is too small）：避免使⽤⽐12sp⼩的字体⼤⼩。

- Accessibility（可访问性）

- ClickableViewAccessibility（Accessibility in Custom Views）：当⼀个 View 重写了 onTouchEvent ⽅法或者使⽤了OnTouchListener⽽没有实现 performClick ⽅法，那么当View被点击是⽆法响应的。

- ContentDescription（Image without contentDescription）：图⽚没有设置 ContentDescription 属性（为视⼒有障碍的⼈增加对图⽚控件的解释）。

- LabelFor（Missing labelFor attribute）：⽂本字段 View 缺少 LabelFor 属性（为视⼒有障碍的⼈增加对⽂本控件的解释）。

## Internationalization（国际化）

- ByteOrderMark（Byte order mark inside files）：我们期望⽂件是UTF-8 编码，所以字节顺序标记 BOM（Byte Order Mark）在 UTF-8 编码⽂件中是不需要的，它会导致基础资源名和翻译不可⽤。

- SetTextI18n（TextView Internationalization）：不要使⽤ + 连接多个⽂本块，这会导致不能准确翻译 多使⽤ %s%d 占位符。

- HardcodedText（Hardcoded text）：硬编码问题。

- RelativeOverlap（Overlapping items in RelativeLayout）：如果不设置 toRightof toLeftof，相对布局中的item会覆盖试图。

- Internationalization:Bidirectional Text（国际 化：双向⽂字）

- RtlSymmetry（Padding and margin symmetry ）：设置 padding 或 margin 左右要对称。

- RtlHardcoded（Using left/right instead of start/end attributes）：例如 layout_marginLeft 等属性没有⼀并⽤上 layout_marginStart（为了照顾从右向左书写的语⾔）。
