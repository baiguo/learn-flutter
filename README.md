# FQ(Flutter常见问题总结)

## 1.app_plugin_loader.gradle' as it does not exist

更新 flutter sdk flutter/bin/flutter upgrade


## 2.flutter run卡在Running Gradle task 'assembleDebug'
**原因**

运行时会卡在Running 'gradle assembleDebug, 因为Gradle的Maven仓库在国外, 可以使用阿里云的镜像地址。

**解决办法**

修改项目下 build.gradle 和 flutter 安装目录flutter/packages/flutter_tools/gradle/flutter.gradle 两个文件中 buildscript 和allprojects 中的


```
google()
jcenter()
```

改为阿里云镜像

```
maven { url 'https://maven.aliyun.com/repository/google' }
maven { url 'https://maven.aliyun.com/repository/jcenter' }
maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }
```


## 3. ios混合工程 Exception: MissingPluginException(No implementation found for method getAll on channel plugins.flutter.io/shared_preferences)
需要进行手动把flutter使用的模块注册下
```
@implementation FlutterNativePlugin
+ (void)registerWithRegistry:(NSObject<FlutterPluginRegistry>*)registry {
  [FLTDeviceInfoPlugin registerWithRegistrar:[registry registrarForPlugin:@"FLTDeviceInfoPlugin"]];
  [FluttertoastPlugin registerWithRegistrar:[registry registrarForPlugin:@"FluttertoastPlugin"]];
  [FLTImagePickerPlugin registerWithRegistrar:[registry registrarForPlugin:@"FLTImagePickerPlugin"]];
  [LaunchReviewPlugin registerWithRegistrar:[registry registrarForPlugin:@"LaunchReviewPlugin"]];
  [FLTPackageInfoPlugin registerWithRegistrar:[registry registrarForPlugin:@"FLTPackageInfoPlugin"]];
  [FLTPathProviderPlugin registerWithRegistrar:[registry registrarForPlugin:@"FLTPathProviderPlugin"]];
  [PermissionHandlerPlugin registerWithRegistrar:[registry registrarForPlugin:@"PermissionHandlerPlugin"]];
  [FLTSharedPreferencesPlugin registerWithRegistrar:[registry registrarForPlugin:@"FLTSharedPreferencesPlugin"]];
  [SqflitePlugin registerWithRegistrar:[registry registrarForPlugin:@"SqflitePlugin"]];
  [FLTURLLauncherPlugin registerWithRegistrar:[registry registrarForPlugin:@"FLTURLLauncherPlugin"]];
  [FLTWebViewFlutterPlugin registerWithRegistrar:[registry registrarForPlugin:@"FLTWebViewFlutterPlugin"]];
}
```
```
  FlutterViewController *flutterViewController = [[FlutterViewController alloc] init];
  self.mUINavigationController = [[UINavigationController alloc] init];
  [self.mUINavigationController pushViewController:flutterViewController animated:YES];
  [GeneratedPluginRegistrant registerWithRegistry:flutterViewController];
  [FlutterNativePlugin registerWithRegistry:flutterViewController];
  [FlutterNativePlugin registerWithRegistrar:[flutterViewController registrarForPlugin:@"FlutterNativePlugin"]];
```

## 4. android混合工程 找不到Flutter资源
这是因为需要把Flutter工程里面的资源拷贝到主anndorid目录中，打开andorid工程下面的bulid.gradle 加入自动拷贝脚本
```
  android.applicationVariants.all { variant ->
      // delete previous files first
      delete "${buildDir}/intermediates/merged_assets/${variant.dirName}"

      variant.mergeAssets.doLast {
          def sourceDir = "${buildDir}/../../../../build/jsb-default"
          def resDir = "${outputDir}/res"
          def flutterAssetsDir = "${buildDir}/../../../flutter_module/.android/Flutter/build/intermediates/flutter/${variant.dirName}/flutter_assets"
          copy {
              from "${flutterAssetsDir}"
              into  "${buildDir}/intermediates/merged_assets/${variant.dirName}/out/flutter_assets"
          }
      }
  }
```

## 5. android混合工程 Exception: MissingPluginException(No implementation found for method getAll on channel plugins.flutter.io/shared_preferences)
如果是flutter_module模式，则android同样需要和ios手动注册一下：
```
  implementation project(path: ':flutter')
  implementation project(path: ':device_info')
  implementation project(path: ':image_picker')
  implementation project(path: ':launch_review')
  implementation project(path: ':package_info')
  implementation project(path: ':path_provider')
  implementation project(path: ':permission_handler')
  implementation project(path: ':shared_preferences')
  implementation project(path: ':sqflite')
  implementation project(path: ':url_launcher')
  implementation project(path: ':webview_flutter')
```
```
  flutterEngine.getPlugins().add(new io.flutter.plugins.imagepicker.ImagePickerPlugin());
  com.iyaffle.launchreview.LaunchReviewPlugin.registerWith(shimPluginRegistry.registrarFor("com.iyaffle.launchreview.LaunchReviewPlugin"));
  flutterEngine.getPlugins().add(new io.flutter.plugins.packageinfo.PackageInfoPlugin());
  flutterEngine.getPlugins().add(new io.flutter.plugins.pathprovider.PathProviderPlugin());
  flutterEngine.getPlugins().add(new com.baseflow.permissionhandler.PermissionHandlerPlugin());
  flutterEngine.getPlugins().add(new io.flutter.plugins.sharedpreferences.SharedPreferencesPlugin());
  flutterEngine.getPlugins().add(new com.tekartik.sqflite.SqflitePlugin());
  flutterEngine.getPlugins().add(new io.flutter.plugins.urllauncher.UrlLauncherPlugin());
  flutterEngine.getPlugins().add(new io.flutter.plugins.webviewflutter.WebViewFlutterPlugin());
```

## 6.archive Bitcode 问题

Bitcode bundle could not be generated because '/***/.ios/Flutter/engine/Flutter.framework/Flutter' was built without full bitcode. All frameworks and dylibs for bitcode must be generated from Xcode Archive 

情形一：有时候时候我们需要将FlutterModule中的.ios工程中的Bitcode关掉就好了
情形二：本情况适用于在情形一无效的情况下

1、修改 ios和.ios 项目bitcode修改为no

2、终端 Flutter build ios --release

3、然后对.ios工程执行pod install，然后bitcode修改为no，然后archive

4、.ios的工程archive没问题的话，再对native 项目执行pod install，bitcode修改为no，然后archive

