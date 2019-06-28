TinkerPatch接入
=================
使用
--------
#第一步 注册并添加App
参考[平台使用说明](http://www.tinkerpatch.com/Docs/start)


#第二步 添加 gradle 插件依赖
```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // TinkerPatch 插件
        classpath "com.tinkerpatch.sdk:tinkerpatch-gradle-plugin:1.2.13"
    }
}
```
#第三步 集成 TinkerPatch SDK
```
dependencies {
    // 若使用annotation需要单独引用,对于tinker的其他库都无需再引用
        annotationProcessor("com.tencent.tinker:tinker-android-anno:1.9.13")
        compileOnly("com.tinkerpatch.tinker:tinker-android-anno:1.9.13")
        implementation("com.tinkerpatch.sdk:tinkerpatch-android-sdk:1.2.13")
}
```
#第四步 添加TinkerPatch配置
####为了简单方便，我们将 TinkerPatch 相关的配置都放于 tinkerpatch.gradle 中, 我们需要将其引入：
```
apply from: 'tinkerpatch.gradle'
```
####另外注意下面：
```
defaultConfig {
        applicationId "com.cmcc.hebao"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion

        multiDexEnabled true
        multiDexKeepProguard file("tinkerMultidexKeep.pro") //keep specific classes using proguard syntax
        javaCompileOptions { annotationProcessorOptions { includeCompileClasspath = true } }

        //版本名后面添加一句话，意思就是flavor dimension 它的维度就是该版本号，这样维度就是都是统一的了
        flavorDimensions "default"
        // 在使用robolectric时需要打开
//        testHandleProfiling true
//        testFunctionalTest true

//        ndk {
//            // 设置支持的SO库架构
//            abiFilters 'armeabi' //, 'x86', 'armeabi-v7a', 'x86_64', 'arm64-v8a'
//        }
        //recommend
        dexOptions {
            jumboMode = true
        }

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
```
#第五步 初始化
```
public class SampleApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        // 我们可以从这里获得Tinker加载过程的信息
        tinkerApplicationLike = TinkerPatchApplicationLike.getTinkerPatchApplicationLike();

        // 初始化TinkerPatch SDK, 更多配置可参照API章节中的,初始化SDK
        TinkerPatch.init(tinkerApplicationLike)
            .reflectPatchLibrary()
            .setPatchRollbackOnScreenOff(true)
            .setPatchRestartOnSrceenOff(true)
            .setFetchPatchIntervalByHours(3);

        // 每隔3个小时(通过setFetchPatchIntervalByHours设置)去访问后台时候有更新,通过handler实现轮训的效果
        TinkerPatch.with().fetchPatchUpdateAndPollWithInterval();
    }
}
```   
    
#第六步 使用步骤（以UAT Release为例）
#####1、修改tinkerpatch.gradle中的appVersion（必须保证每个基准包的appVersion唯一性），执行assembleUATRelease任务，会在build/bakPath目录生成，
![avatar](./1.png)
*上图文件需要备份，热修复是需要作为补丁的基准包

####2、当出现bug时，修复完bug后，需要配置好基准包的目录 如下：
```
/**
 * TODO: 请按自己的需求修改为适应自己工程的参数
 */
def bakPath = file("${buildDir}/bakApk/")
def baseInfo = "mocam20_int_0513-8.3.66-0625-14-20-02"
def variantName = "uat-release"
```
####接下来执行tinkerPatchUATRelease任务，会在以下生成：
![avatar](./2.png)
“patch_signed_7zip.apk” 就是我们需要用到的补丁包。
#第七步 上传补丁包
去 [TinkerPatch](http://www.tinkerpatch.com/Apps/detail/id/11982)后台添加App版本，版本号和tinkerpatch.gradle里面设置的保持一致，发布后立即生效，然后打开app会自动进行下载并合成，成功后需要重启进程才能生效。
#版本回退
如果发布的补丁有问题需要回退，只需要把TinkerPatch后台对应的补丁包删除
