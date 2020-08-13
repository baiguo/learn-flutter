# FQ(常见问题总结)

## 1.app_plugin_loader.gradle' as it does not exist

更新 flutter sdk flutter/bin/flutter upgrade


## 2.flutter run卡在Running Gradle task 'assembleDebug'
**原因**
运行时会卡在Running 'gradle assembleDebug, 因为Gradle的Maven仓库在国外, 可以使用阿里云的镜像地址。

**解决办法**
修改项目下 build.gradle 和 flutter 安装目录flutter/packages/flutter_tools/gradle/flutter.gradle 两个文件中 buildscript 和allprojects 中的

'google()'
'jcenter()'

