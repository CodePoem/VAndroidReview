# UI适配

基本概念：

- 屏幕分辨率：手机横纵方向上的像素点数。比如常见的720 \* 1280 ，1080 \* 1920，单位 px，1px=一个像素点。
- 屏幕尺寸：不是屏幕宽度，也不是长度，而是屏幕的对角线长度。比如常见的5.0尺寸，5.5尺寸，单位是英寸（inch），1英寸=2.54厘米。
- dpi：全称dots per ich，也就是每英寸的像素点数。***系统软件上的概念(参考客观存在不会改变的物理像素密度ppi，人为指定的一个值，这样保证了某一个区间内的物理像素密度在软件上都使用同一个值。)。***屏幕对角线像素点数 / 屏幕的对角线长度。
- px：像素
- sp：全称scale-independent pixel，也叫sip，独立缩放像素。
- dp：全称Density-independent pixel 也叫dip，即独立像素密度。

重要公式：

- dpi = √(屏幕宽度像素^2 + 屏幕高度像素^2) / 屏幕的对角线长度
- density = dpi / 160
- px = dp * density

在Android中，规定以160dpi为基准，1dp=1px；320dpi下，1dp = 2px，计算公式：px = dp * (dpi / 160)，屏幕密度越大，1dp对应 的像素点越多。像素密度文件夹 mdpi、hdpi、xhdpi、xxhdpi、xxxhdpi。

默认情况下，通过dp 和 sp 加上自适应布局和 weight 比例布局可以基本解决不同手机上适配的问题，这基本是最原始的Android适配方案。但由于国内手机厂商屏幕碎片化严重，导致决定 dpi 的两个因素（屏幕对角线像素点数和屏幕的对角线长度）千奇百怪（例如相同屏幕对角线像素点数，不同屏幕的对角线长度的设备），所以需要针对性的适配方案。

## 资源文件夹使用限定符

### 宽高限定符

```txt
├── src/main
│   ├── res
│   ├── ├──values
│   ├── ├──values-800x480
│   ├── ├──values-860x540
│   ├── ├──values-1024x600
│   ├── ├──values-1024x768
│   ├── ├──...
│   ├── ├──values-2560x1440
```

针对屏幕宽高创建不同values文件夹进行适配。

缺点：

- 需要精准匹配屏幕分宽高。

### 屏幕最小宽度限定符

```txt
├── src/main
│   ├── res
│   ├── ├──values
│   ├── ├──values-sw320dp
│   ├── ├──values-sw360dp
│   ├── ├──values-sw400dp
│   ├── ├──values-sw480dp
│   ├── ├──...
│   ├── ├──values-sw600dp
```

针对宽高大小小sp、dp的资源文件，创建不同屏幕分辨率限定符的values文件夹。

以设计图最小宽度（单位为 dp）作为基准值，生成所有设备对应的 dimens.xml 文件。（网上已有开源的插件可以使用）

优点：

- 有很好的容错机制，会自动寻找最接近的dpi。

缺点：

- 侵入性较大，替换其他方案需要改动各处引用。
- 那就是多个 dimen s文件可能导致apk变大。
- 个别手机屏幕分辨率需要单独添加适配。

## 动态计算子控件宽高

### 百分比布局

计算填入的百分比后动态计算 LayoutParams 中的宽高。

### AutoLayout

和百分比布局库思想类似，读取 AndroidManifest 中注明你的设计稿的尺寸动态计算 LayoutParams 中的宽高。

考虑：

- 因为需要在运行时会在 onMeasure 里面动态计算，自定义的控件可能会被影响或限制，可能特定的控件需要单独适配，还有一个比较重要的问题。

## 动态更改density

将公式

dpi = √(屏幕宽度像素^2 + 屏幕高度像素^2) / 屏幕的对角线长度 / 160

修改为

density = 当前设备屏幕总宽度 / 设计图总宽度

优点：

- 侵入性低，接入简单，可随时替换其他方案，试错成本低。

缺点：

- 全局修改，项目中的系统控件、三方库控件、等非我们项目自身设计的控件，它们的设计图尺寸并不会和我们项目自身的设计图尺寸一样当这个适配方案不分类型，将所有控件都强行使用我们项目自身的设计图尺寸进行适配时，这时就会出现问题，当某个系统控件或三方库控件的设计图尺寸和和我们项目自身的设计图尺寸差距非常大时，这个问题就越严重。