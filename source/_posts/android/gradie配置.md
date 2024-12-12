
Gradle 是一个基于 JVM 的构建工具，也是一款非常灵活强大的构建工具，支持 jcenter、maven、Ivy 仓库，支持传递性依赖管理 即 A 依赖 B,B 依赖 C,那么 A 也就可以依赖 C不用再单独去依赖,而不需要远程仓库或者是 pom.xml 和 ivy.xml 配置文件，抛弃了各种繁琐，基于 Groovy,build 脚本使用 Groovy 编写。

在我们的工程构建过程中通常会创建很多个 Module 来对我们的工程进行功能以及业务上的解耦（也就是模块化开发），这时候可能就会存在一个问题，就是每个 Module 以及 Module 中一些公用库的依赖可能会出现版本不统一的问题,包括使用的编译版本,SDK 的版本等，导致不能打包，这里可以使用 Gradle 统一配置文件来解决我们的问题 

首先我们来看一下，正常情况下我们的项目目录的 build.gradle 情况： 

# 我们来看一下项目根目录下的 build.gradle

```javascript
//构建脚本 
buildscript { 
   repositories { 
     //依赖的仓库 
    jcenter() 
  }

  dependencies { 
     
    //项目依赖的Gradle版本 
    classpath 'com.android.tools.build:gradle:2.2.3' 
 
    // NOTE: Do not place your application dependencies here; they belong 
    // in the individual module build.gradle files 
  } 
} 
 
allprojects { 
  repositories { 
    jcenter() 
  } 
} 
 
task clean(type: Delete) { 
  delete rootProject.buildDir 
} 
```

# 先看 app 下的 build.gradle

```javascript

// 声明是Android程序，
//com.android.application 表示这是一个应用程序模块
//com.android.library 标识这是一个库模块
//而这区别：前者可以直接运行，后着是依附别的应用程序运行
apply plugin: 'com.android.application'

android {
    signingConfigs {// 自动化打包配置
        release {// 线上环境
            keyAlias 'test'
            keyPassword '123456'
            storeFile file('test.jks')
            storePassword '123456'
        }
        debug {// 开发环境
            keyAlias 'test'
            keyPassword '123456'
            storeFile file('test.jks')
            storePassword '123456'
        }
    }
    compileSdkVersion 27//设置编译时用的Android版本
    defaultConfig {
        applicationId "com.billy.myapplication"//项目的包名
        minSdkVersion 16//项目最低兼容的版本
        targetSdkVersion 27//项目的目标版本
        versionCode 1//版本号
        versionName "1.0"//版本名称
        flavorDimensions "versionCode"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"//表明要使用AndroidJUnitRunner进行单元测试
    }
    buildTypes {// 生产/测试环境配置
        release {// 生产环境
            buildConfigField("boolean", "LOG_DEBUG", "false")//配置Log日志
            buildConfigField("String", "URL_PERFIX", "\"https://release.cn/\"")// 配置URL前缀
            minifyEnabled false//是否对代码进行混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'//指定混淆的规则文件
            signingConfig signingConfigs.release//设置签名信息
            pseudoLocalesEnabled false//是否在APK中生成伪语言环境，帮助国际化的东西，一般使用的不多
            zipAlignEnabled true//是否对APK包执行ZIP对齐优化，减小zip体积，增加运行效率
            applicationIdSuffix 'test'//在applicationId 中添加了一个后缀，一般使用的不多
            versionNameSuffix 'test'//在applicationId 中添加了一个后缀，一般使用的不多
        }
        debug {// 测试环境
            buildConfigField("boolean", "LOG_DEBUG", "true")//配置Log日志
            buildConfigField("String", "URL_PERFIX", "\"https://test.com/\"")// 配置URL前缀
            minifyEnabled false//是否对代码进行混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'//指定混淆的规则文件
            signingConfig signingConfigs.debug//设置签名信息
            debuggable false//是否支持断点调试
            jniDebuggable false//是否可以调试NDK代码
            renderscriptDebuggable false//是否开启渲染脚本就是一些c写的渲染方法
            zipAlignEnabled true//是否对APK包执行ZIP对齐优化，减小zip体积，增加运行效率
            pseudoLocalesEnabled false//是否在APK中生成伪语言环境，帮助国际化的东西，一般使用的不多
            applicationIdSuffix 'test'//在applicationId 中添加了一个后缀，一般使用的不多
            versionNameSuffix 'test'//在applicationId 中添加了一个后缀，一般使用的不多
        }
    }

    sourceSets {//目录指向配置
        main {
            jniLibs.srcDirs = ['libs']//指定lib库目录
        }
    }

    packagingOptions{//打包时的相关配置
        //pickFirsts做用是 当有重复文件时 打包会报错 这样配置会使用第一个匹配的文件打包进入apk
        // 表示当apk中有重复的META-INF目录下有重复的LICENSE文件时  只用第一个 这样打包就不会报错
        pickFirsts = ['META-INF/LICENSE']

        //merges何必 当出现重复文件时 合并重复的文件 然后打包入apk
        //这个是有默认值得 merges = [] 这样会把默默认值去掉  所以我们用下面这种方式 在默认值后添加
        merge 'META-INF/LICENSE'

        //这个是在同时使用butterknife、dagger2做的一个处理。同理，遇到类似的问题，只要根据gradle的提示，做类似处理即可。
        exclude 'META-INF/services/javax.annotation.processing.Processor'
    }

    productFlavors {
        wandoujia {}
        xiaomi {}
        _360 {}
    }

    productFlavors.all {
            //批量修改，类似一个循序遍历
        flavor -> flavor.manifestPlaceholders = [IFLYTEK_CHANNEL: name]
    }

    //程序在编译的时候会检查lint，有任何错误提示会停止build，我们可以关闭这个开关
    lintOptions {
        abortOnError false
        //即使报错也不会停止打包
        checkReleaseBuilds false
        //打包release版本的时候进行检测
    }

}

dependencies {
    //项目的依赖关系
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    //本地jar包依赖
    implementation 'com.android.support:appcompat-v7:27.1.1'
    //远程依赖
    implementation 'com.android.support.constraint:constraint-layout:1.1.2'
    testImplementation 'junit:junit:4.12'
    //声明测试用例库
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}

```



# 我们添加一个 Module 库,来看一下我们 Module 库下的 build.gradle
```javascript
apply plugin: 'com.android.library' 
android { 
  compileSdkVersion 23 
  buildToolsVersion "23.0.2" 
 
  defaultConfig { 
    minSdkVersion 13 
    targetSdkVersion 23 
    versionCode 1 
    versionName "1.0" 
     testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner" 
 
  } 
  buildTypes { 
    release { 
      minifyEnabled false 
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro' 
    } 
  } 
} 
 
dependencies { 
  compile fileTree(dir: 'libs', include: ['*.jar']) 
  androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', { 
    exclude group: 'com.android.support', module: 'support-annotations' 
  }) 
  compile 'com.android.support:appcompat-v7:25.0.0' 
  testCompile 'junit:junit:4.12' 
} 
```

这里我们来看一下和 app 目录下的 build.gradle 有什么区别： 

app 目录下的 build.gradle 是：apply plugin:com.android.application 

Module 库下的 build.gradle 是：apply plugin:com.android.library 

其它的就是版本的不一样了，要素是一样的，这里我们看到编译的 SDK 版本和编译的 Tools 版本以及支持 SDK 的最低版本等的版本号都是不一样的，这里我们就需要来统一，而我们总不能每次都来手动配置。

# 在根目录创建 config.gradle 来统一配置

config.gradle 里面的配置信息： 

```javascript

/** 
 * 在主项目的根目录下创建config.gradle文件 
 * 在这里单独处理统一依赖问题 
 * 注意需要在根目录的build.gradle中进行引入 
 */ 
ext { 
  android = [ 
      compileSdkVersion: 25, 
      buildToolsVersion: "25.0.2", 
      minSdkVersion  : 15, 
      targetSdkVersion : 25 
  ] 
 
  //Version 
  supportLibrary = "25.1.0" 
 
  //supportLibraries dependencies 
  supportDependencies = [ 
      supportAppcompat: "com.android.support:appcompat-v7:${supportLibrary}", 
      supportV4    : "com.android.support:support-v4:${supportLibrary}", 
      suppoutDesign  : "com.android.support:design:${supportLibrary}" 
  ] 
} 
```

然后我们需要在根目录的 build.gradle 中把 config.gradle 引入进来，这里特别注意是在根目录的 build.gradle 中引入。

引入的代码为： apply from: "config.gradle"  

接下来我们就可以在 Module 中引入使用了，如下： 
```javascript

apply plugin: 'com.android.library' 
 
//android配置 
def config = rootProject.ext.android 
 
//相关库依赖 
def librarys = rootProject.ext.supportDependencies 
 
android { 
  compileSdkVersion config.compileSdkVersion 
  buildToolsVersion config.buildToolsVersion 
 
  defaultConfig { 
    minSdkVersion config.minSdkVersion 
    targetSdkVersion config.targetSdkVersion 
    versionCode 1 
    versionName "1.0" 
 
    testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner" 
 
  } 
  buildTypes { 
    release { 
      minifyEnabled false 
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro' 
    } 
  } 
} 
 
dependencies { 
  compile fileTree(dir: 'libs', include: ['*.jar']) 
  androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', { 
    exclude group: 'com.android.support', module: 'support-annotations' 
  }) 
  testCompile 'junit:junit:4.12' 
 
  //在这里使用库的依赖 
  compile librarys.supportAppcompat 
  compile librarys.supportV4 
  compile librarys.suppoutDesign 
} 
```
到这里我们就成功的引入到了 Module 的 build.gradle 中，以后每个 Module 中的引入都是这样。
