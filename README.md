# DiDiVirtualApkDemo
滴滴插件化框架VirtualApk  Demo
# 预览效果
空位

滴滴 VirtualApk git地址：https://github.com/didi/VirtualAPK
Demo git地址：https://github.com/caixiaoxu/DiDiVirtualApkDemo

## 接入流程
1. 新建两个项目，一个为宿主项目(PluginMain)，一个是子项目(PluginSub),**(保证两个项目中的文件不能有重名**)
2. 修改两个项目的gradle **(对Gradle版本要求统一)**
* 修改gradle版本为3.0.0,路径-根目录/build.gradle：
<code>classpath 'com.android.tools.build:gradle:3.0.0'</code>
* 修改项目gradleg下载版本,路径-gradle/wrapper/gradle-wrapper.properties
<code>distributionUrl=https\://services.gradle.org/distributions/gradle-4.6-all.zip</code>

---
3. 在PluginMain根目录下的build.gradle添加插件工具
```
	dependencies {
    	classpath 'com.didi.virtualapk:gradle:0.9.8.6'
	}
```
4. 在PluginMain/app目录下的build.gradle顶部应用插件
```
	apply plugin: 'com.didi.virtualapk.host'
```
5. 在PluginMain/app目录下的build.gradle导入库
```
	dependencies {
		compile 'com.didi.virtualapk:core:0.9.8'
	}
```
6. 自定义的Application，并在attachBaseContext中初始化 **(注意要在AndroidManifest.xml中添加配置)**
```
	@Override
	protected void attachBaseContext(Context base) {
	    super.attachBaseContext(base);
	    PluginManager.getInstance(base).init();
	}
```

7. 在Activity中加载Apk文件，跳转 **(需添加读权限)**
```
	//apk路径
	File plugin = new File(Environment.getExternalStorageDirectory(), "PluginSub.apk");
    if (plugin.exists()) {
        try {
            PluginManager.getInstance(this).loadPlugin(plugin);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
```
	Intent intent = new Intent();
    intent.setClassName("com.lsy.pluginsub","com.lsy.pluginsub.PluginMainActivity");
    startActivity(intent);
```
---
8. 在PluginSub根目录下的build.gradle添加插件工具
```
	dependencies {
    	classpath 'com.didi.virtualapk:gradle:0.9.8.6'
	}
```
9. 在PluginSub/app目录下的build.gradle顶部应用插件
```
	apply plugin: 'com.didi.virtualapk.plugin'
```
10. 在PluginSub/app目录下的build.gradle增加配置文件
```
	virtualApk {
	    packageId = 0x6f             // PackageId值，在[0x02, 0x7E]之间取值.
	    targetHost='../PluginMain/app' // 宿主项目路径
	    applyHostMapping = true      // 默认为true,混淆时候生成的映射表保持一致
	}
```
---
11. 打开终端，输入
```
	gradlew(./gradlew) clean assemblePlugin
	或者：
	gradle clean assemblePlugin
```
12. 修改打包后的文件名，放置到宿主文件中配置的路径，运行
---
-----------------编译报错及解决-----------------
- 问题1：PackageId取值太小或太大
```
	> Failed to notify project evaluation listener.
   		> the packageId must be in [0x02, 0x7E].
   		> Cannot invoke method onProjectAfterEvaluate() on null object
```
- 解决方法：virtualApk配置中的packageId值，在[0x02, 0x7E]之间
- 问题2：没有host配置
```
	> Failed to notify project evaluation listener.
	   > Can't find /Users/Lsy/AndroidStudioProjects/DiDiVirtualApkDemo/PluginMain/app/build/VAHost/versions.txt, please check up your host application
	       need apply com.didi.virtualapk.host in build.gradle of host application 
	   > Cannot invoke method onProjectAfterEvaluate() on null object
```
- 解决方法：宿主项目需先Build Apk
- 问题3：
```
	> Failed to notify project evaluation listener.
	   > Can't using incremental dexing mode, please add 'android.useDexArchive=false' in gradle.properties of :app.
	   > Cannot invoke method onProjectAfterEvaluate() on null object
```
- 解决方法：在PluginSub根目录下gradle.properties中添加android.useDexArchive=false
- 问题4：buildToolsRevision版本过高
```
	> Required entry 'activity_plugin_main' but got 'abc_select_dialog_material', This is seems to unsupport the buildToolsRevision: 29.0.2.
```
- 解决方法：修改PluginSub/app目录下的build.gradle中的buildToolsVersion版本为：26.0.2

---
-----------------运行报错及解决-----------------
- 问题1：
```
	Caused by: android.content.pm.PackageParser$PackageParserException: Package /storage/emulated/0/PluginSub.apk has no certificates at entry AndroidManifest.xml
```
- 解决方法：子项目(PluginSub)需要签名后才能运行，宿主项目(PluginMain)是否签名都可以


