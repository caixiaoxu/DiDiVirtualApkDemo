# DiDiVirtualApkDemo
滴滴插件化框架VirtualApk  Demo

滴滴 VirtualApk git地址：https://github.com/didi/VirtualAPK

## 接入流程
- ### 1、新建两个项目，一个为宿主项目(PluginMain)，一个是子项目(PluginSub),(保证两个项目中的文件不能有重名)
- ### 2、修改两个项目gradle版本为3.0.0,路径-根目录/build.gradle,修改项目gradleg下载版本,路径-gradle/wrapper/gradle-wrapper.properties(对Gradle版本要求统一)：
<code>classpath 'com.android.tools.build:gradle:3.0.0'</code>
<code>distributionUrl=https\://services.gradle.org/distributions/gradle-4.6-all.zip</code>
------------------------------------------宿主项目(PluginMain)------------------------------------------
- ### 3、在PluginMain根目录下的build.gradle添加插件工具
<code>
	dependencies {
    	classpath 'com.didi.virtualapk:gradle:0.9.8.6'
	}
</code>
- ### 4、在PluginMain/app目录下的build.gradle顶部应用插件
<code>
	apply plugin: 'com.didi.virtualapk.host'
</code>
- ### 5、在PluginMain/app目录下的build.gradle导入库
<code>
	dependencies {
		compile 'com.didi.virtualapk:core:0.9.8'
	}
</code>
- ### 6、自定义的Application，并在attachBaseContext中初始化(注意要在AndroidManifest.xml中添加配置)
<code>
	@Override
	protected void attachBaseContext(Context base) {
	    super.attachBaseContext(base);
	    PluginManager.getInstance(base).init();
	}
</code>

- ### 7、在Activity中加载Apk文件，跳转(需添加读权限)
<code>
	//apk路径
	File plugin = new File(Environment.getExternalStorageDirectory(), "PluginSub.apk");
    if (plugin.exists()) {
        try {
            PluginManager.getInstance(this).loadPlugin(plugin);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
</code>
<code>
	Intent intent = new Intent();
    intent.setClassName("com.lsy.pluginsub","com.lsy.pluginsub.PluginMainActivity");
    startActivity(intent);
</code>
------------------------------------------子项目(PluginSub)------------------------------------------
- ### 8、在PluginSub根目录下的build.gradle添加插件工具
<code>
	dependencies {
    	classpath 'com.didi.virtualapk:gradle:0.9.8.6'
	}
</code>
- ### 9、在PluginSub/app目录下的build.gradle顶部应用插件
<code>
	apply plugin: 'com.didi.virtualapk.plugin'
</code>
- ### 10、在PluginSub/app目录下的build.gradle增加配置文件
<code>
	virtualApk {
	    packageId = 0x6f             // PackageId值，在[0x02, 0x7E]之间取值.
	    targetHost='../PluginMain/app' // 宿主项目路径
	    applyHostMapping = true      // 默认为true,混淆时候生成的映射表保持一致
	}
</code>
------------------------------------------编译运行------------------------------------------
- ### 11、打开终端，输入
<code>
	gradlew(./gradlew) clean assemblePlugin
	或者：
	gradle clean assemblePlugin
</code>
------------------------------------------编译报错及解决------------------------------------------
- 问题1：PackageId取值太小或太大
<code>
	> Failed to notify project evaluation listener.
   		> the packageId must be in [0x02, 0x7E].
   		> Cannot invoke method onProjectAfterEvaluate() on null object
</code>
解决方法：virtualApk配置中的packageId值，在[0x02, 0x7E]之间
- 问题2：没有host配置
<code>
	> Failed to notify project evaluation listener.
	   > Can't find /Users/Lsy/AndroidStudioProjects/DiDiVirtualApkDemo/PluginMain/app/build/VAHost/versions.txt, please check up your host application
	       need apply com.didi.virtualapk.host in build.gradle of host application 
	   > Cannot invoke method onProjectAfterEvaluate() on null object
</code>
解决方法：宿主项目需先Build Apk
- 问题3：
<code>
	> Failed to notify project evaluation listener.
	   > Can't using incremental dexing mode, please add 'android.useDexArchive=false' in gradle.properties of :app.
	   > Cannot invoke method onProjectAfterEvaluate() on null object
</code>
解决方法：在PluginSub根目录下gradle.properties中添加android.useDexArchive=false
- 问题4：buildToolsRevision版本过高
<code>
	> Required entry 'activity_plugin_main' but got 'abc_select_dialog_material', This is seems to unsupport the buildToolsRevision: 29.0.2.
</code>
解决方法：修改PluginSub/app目录下的build.gradle中的buildToolsVersion版本为：26.0.2
------------------------------------------运行报错及解决------------------------------------------
- 问题1：
<code>
	Caused by: android.content.pm.PackageParser$PackageParserException: Package /storage/emulated/0/PluginSub.apk has no certificates at entry AndroidManifest.xml
</code>
解决方法：子项目(PluginSub)需要签名后才能运行，宿主项目(PluginMain)是否签名都可以




