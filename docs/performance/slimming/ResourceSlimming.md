# 缩包（apk瘦身）

## apk构成

* res资源文件夹；
* assets文件夹；
* lib库文件夹；
* classes.dex源码；
* 编译生成的二进制资源文件resources.arsc；
* AndroidManifest.xml清单文件；
* 依赖关系配置文件project.properties；
* 代码混淆配置文件proguard.cfg；
* 签名信息文件META-INF等。

## 方向

1. 图片资源压缩
2. 移除无用资源 如lint 添加shrinkResources设置项 [谷歌官方说明](https://developer.android.com/studio/build/shrink-code)
3. 资源混淆 [微信资源混淆工具AndResGuard](https://github.com/shwenzhang/AndResGuard)
4. [ThinR插件](https://github.com/meili/ThinRPlugin) 在编译时将除R$styleable.class以外的所有R.class删除掉，并且在引用的地方替换成对应的常量，从而达到缩减包大小和减少dex个数的效果
5. 减少语言支持 点击resources.arsc可以看到默认支持80多种语言
6. 架构支持减少
7. 代码精简 如精简support库/使用更小的库/删除冗余库等
8. 动态下发 如so、图片等
9. 支持插件化

## 图片资源篇

### 建议

* 用代码代替图片(xml描述) shape RotateDrawable layer-list selector
* 使用9.png图
* 如果是纯色的icon，那么用svg
* 如果是两种以上颜色的icon，用webp
* 如果webp无法达到效果，选择png
* 如果图片没有alpha通道，可以考虑jpg
* 减少适配图片套数，hdpi、mdpi、xhdpi、xxhdpi、xxxhdpi等
* 1x1的透明图片覆盖覆盖第三方库里的大图

能不用图片的就不用图片（用代码实现），如果要用图片则优先使用9图

### 压缩图片工具

1.[ImageOptim](https://imageoptim.com)
开源免费、提供了客户端
最好自定义命令行实现：

* 是否过滤点9图
* 针对压缩后视觉效果差异过大的设置白名单

2.[tinypng](https://tinypng.com/)
付费，有免费版但有限制，[TinyPng Gradle插件](https://github.com/meili/TinyPIC_Gradle_Plugin)

pngcrush,pngquant,zopflipng

### webp

[官网地址](https://developers.google.com/speed/webp/docs/cwebp)

#### 压缩效果

全仓库压缩 默认压缩率 可缩小20M
ManagerCounterRankSettingModule release包减少4.09M
ManagerFinancePayModule release包减少2.4M
ManagerChargeModule  release包减少40kb

#### 注意事项

[官方兼容性说明](https://developer.android.com/guide/appendix/media-formats.html)

* WebP的编码时间为PNG的5倍以上，解码速度与PNG差不多，甚至更快；
* WebP编码时占用内存比PNG高25%，解码时比PNG低30%；
* Android 4.0+开始支持WebP图片，但是带透明度的WebP图片是在4.2.1+之后才支持的，在4.2.1之前则无法显示；

ImageMagick 命令查看图片有没有 Alpha 值：

```shell
dentify -format '%A'
```
