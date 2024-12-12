

每个应用项目都必须在项目源代码集的根目录下有一个 AndroidManifest.xml 文件，且文件名精确无误。清单文件会向 Android 构建工具、Android 操作系统和 Google Play 描述有关应用的基本信息。

清单文件必须声明以下内容（还有许多其他内容）：

+ 应用的组件，包括所有 activity、服务、广播接收器和 content provider。每个组件都必须定义一些基本属性，如该组件的 Kotlin 或 Java 类的名称。组件还会声明一些功能，如它可以处理哪些设备配置，以及描述如何启动组件的 intent 过滤器。请阅读下一部分，详细了解应用组件。
+ 应用为了访问系统或其他应用的受保护部分而需要具备的权限。清单文件还会声明其他应用想要访问此应用的内容时必须具备的所有权限。
+ 应用所需的硬件和软件功能，这些功能会影响哪些设备可以从 Google Play 安装该应用。

如果您使用 Android Studio 构建应用，系统会为您创建清单文件，并且会在您构建应用时添加大部分基本清单元素，尤其是在使用代码模板时。

# 结构图

```xml
<?xmlversion="1.0"encoding="utf-8"?>
<manifest>
    <uses-sdk/> 
    <uses-configuration/> 
    <uses-feature/>  
 
    <uses-permission/>
    <permission/>
    <permission-tree/>
    <permission-group/>
    <instrumentation/> 
 
    <supports-screens/>
 
    <application> 
       <activity> 
           <intent-filter>
               <action/> 
               <category/> 
           </intent-filter> 
      </activity>
       <activity-alias> 
           <intent-filter></intent-filter> 
           <meta-data/> 
      </activity-alias> 
       <service> 
           <intent-filter></intent-filter> 
           <meta-data/> 
       </service>
       <receiver>
           <intent-filter></intent-filter> 
           <meta-data/> 
       </receiver> 
       <provider> 
           <grant-uri-permission/>
           <meta-data/> 
       </provider> 
       <uses-library/> 
    </application>  
 
</manifest>
```

# Manifest:属性

AndroidManifest.xml 文件的根元素。它必须包含 `<application>` 元素并指定 xmlns:android 和 package 属性。

```xml
<manifest  xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.somnus.yunyi"
          android:sharedUserId="string"
          android:sharedUserLabel="string resource"
          android:versionCode="integer"
          android:versionName="string"
          android:installLocation=["auto" | "internalOnly" | "preferExternal"] >
</manifest>
```

  + xmlns:android
  定义 Android 命名空间。此属性始终设置为 "http://schemas.android.com/apk/res/android"。

  + package
  APK 清单文件中 package 属性的值将代表应用的通用唯一应用 ID。如果是 Android 应用，软件包名称的格式会遵循完整 Java 语言样式。各个软件包名称部分只能以字母开头。
  
  请注意不要更改 package 值，因为从本质上说，这样做会创建一个新应用。旧版应用的用户不仅不会收到更新，而且还无法在新旧版本之间转移数据。

  + sharedUserId
  与其他应用共享的 Linux 用户 ID 的名称。 默认情况下，Android 会为每个应用分配其唯一用户 ID。 不过，如果针对两个或两个以上的应用将此属性设置为相同的值，只要这些应用的证书集完全相同，它们就会共享相同的 ID。具有相同用户 ID 的应用可以访问彼此的数据，如果需要的话，还可以在同一进程中运行。
  
  此常量自 API 级别 29 起已废弃。共享用户 ID 会在软件包管理器中导致具有不确定性的行为。因此，强烈建议您不要使用它们，并且我们在未来的 Android 版本中可能会移除它们。

  + sharedUserLabel
  此常量自 API 级别 29 起已废弃。
  
  + versionCode
  内部版本号，是给设备程序识别版本(升级)用的必须是一个interger值代表app更新过多少次，此数字仅用于确定某个版本是否比另一个版本更新：数字越大表示版本越新。此数字不会显示给用户。

  + versionName
  这个名称是给用户看的，你可以将你的APP版本号设置为1.1版，后续更新版本设置为1.2、2.0版本等等。除了向用户显示之外，该字符串没有其他用途。

  + installLocation
  应用的默认安装位置。接受以下关键字字符串：
    + internalOnly：应用仅安装在设备内部存储空间中。如果设置此值，则应用一定不会安装在诸如 SD 卡等外部存储空间中。如果内部存储空间已满，系统不会安装应用。如果您没有定义 android:installLocation，这会是默认行为。
    + auto：应用可以安装在外部存储空间中，但默认情况下，系统会将应用安装在内部存储空间中。如果内部存储空间已满，系统会将应用安装在外部存储空间中。安装后，用户可以通过系统设置将应用移至内部或外部存储空间。
    + preferExternal：应用更倾向于安装在外部存储空间中。无法保证系统会遵循该请求。如果外部媒体不可用或已满，则应用可能会安装在内部存储空间中。安装后，用户可以通过系统设置将应用移至内部或外部存储空间。
 
# Application:属性 

一个AndroidManifest.xml中必须含有一个Application标签，这个标签声明了每一个应用程序的组件及其属性(如icon,label,permission等)设置默认值。其他属性（如 debuggable、enabled、description 和 allowClearUserData）则为整个应用设置值，并且不会被组件替换。

```xml
<application android:allowTaskReparenting=["true" | "false"]
             android:allowBackup=["true" | "false"]
             android:allowClearUserData=["true" | "false"]
             android:allowCrossUidActivitySwitchFromBelow=["true" | "false"]
             android:allowNativeHeapPointerTagging=["true" | "false"]
             android:appCategory=["accessibility" | "audio" | "game" |
             "image" | "maps" | "news" | "productivity" | "social" | "video"]
             android:backupAgent="string"
             android:backupInForeground=["true" | "false"]
             android:banner="drawable resource"
             android:dataExtractionRules="string resource"
             android:debuggable=["true" | "false"]
             android:description="string resource"
             android:enabled=["true" | "false"]
             android:extractNativeLibs=["true" | "false"]
             android:fullBackupContent="string"
             android:fullBackupOnly=["true" | "false"]
             android:gwpAsanMode=["always" | "never"]
             android:hasCode=["true" | "false"]
             android:hasFragileUserData=["true" | "false"]
             android:hardwareAccelerated=["true" | "false"]
             android:icon="drawable resource"
             android:isGame=["true" | "false"]
             android:isMonitoringTool=["parental_control" | "enterprise_management" |
             "other"]
             android:killAfterRestore=["true" | "false"]
             android:largeHeap=["true" | "false"]
             android:label="string resource"
             android:logo="drawable resource"
             android:manageSpaceActivity="string"
             android:name="string"
             android:networkSecurityConfig="xml resource"
             android:permission="string"
             android:persistent=["true" | "false"]
             android:process="string"
             android:restoreAnyVersion=["true" | "false"]
             android:requestLegacyExternalStorage=["true" | "false"]
             android:requiredAccountType="string"
             android:resizeableActivity=["true" | "false"]
             android:restrictedAccountType="string"
             android:supportsRtl=["true" | "false"]
             android:taskAffinity="string"
             android:testOnly=["true" | "false"]
             android:theme="resource or theme"
             android:uiOptions=["none" | "splitActionBarWhenNarrow"]
             android:usesCleartextTraffic=["true" | "false"]
             android:vmSafeMode=["true" | "false"] >
    . . .
</application>
```
## allowTaskReparenting
默认值为 "false"。确定应用定义的 activity 是否可以从启动它们的任务移至对其具有亲和性的任务（当下次将该任务置于前台时）。如果它们可以移动，则设为 "true"；如果它们必须一直与启动它们的任务在一起，则设为 "false"。

## allowBackup
默认值为 "true"。确定应用是否参与备份和恢复基础架构。如果将此属性设为 "false"，则永远不会为该应用执行备份或恢复，即使是采用全系统备份方法也不例外（这种备份方法通常会通过 adb 保存所有应用数据）。

## allowClearUserData
此属性的默认值为 "true"。确定应用是否允许重置用户数据。这些数据包括标志（如用户是否看到了介绍性提示）以及用户可自定义的设置和偏好设置。

## allowCrossUidActivitySwitchFromBelow
指定任务中此优先级以下的 activity 是否也可启动其他 activity 或完成相应任务。


## allowNativeHeapPointerTagging
确定应用是否启用堆指针标记功能。此属性的默认值为 "true"。

## appCategory
应用的类别。类别可以在汇总电池使用、网络使用或磁盘使用等情况时，将多个应用进行有效分组。只为适合某个特定类别的应用定义此值。

必须是以下某个常量值。
accessibility	主要为无障碍应用的应用，例如屏幕阅读器。
audio	主要为与音频或音乐相关的应用，例如音乐播放器。
game	主要为游戏的应用。
image	主要为与图片或照片相关的应用，例如相机或图库应用。
maps	主要为与地图相关的应用，例如导航应用。
news	主要为新闻应用的应用，例如报纸应用、杂志应用或体育新闻应用。
productivity	主要为效率应用的应用，例如云端存储应用或工作场所应用。
social	主要为社交应用的应用，例如即时通讯应用、沟通应用、电子邮件应用或社交网络应用。
video	主要为与视频或电影相关的应用，例如流式视频应用。

## backupAgent
没有默认值，必须指定具体的名称。

实现应用的备份代理的类的名称，它是 BackupAgent 的子类。属性值是完全限定的类名称，例如 "com.example.project.MyBackupAgent"。不过，作为一种简写形式，如果名称的第一个字符是句点（例如 ".MyBackupAgent"），则会附加到 <manifest> 元素中指定的软件包名称。

## backupInForeground
表示即使此应用处于前台等效状态，也可以对其执行自动备份操作。默认值为 "false"，这表示当应用在前台运行时（如正在使用处于 startForeground() 状态的服务播放音乐的音乐应用），操作系统会避免对其进行备份。

## banner
一种可绘制资源，可为其关联项提供扩展图形横幅。它可以与 <application> 标记一起使用，为所有应用 activity 提供默认横幅；也可以与 <activity> 标记一起使用，为特定 activity 提供横幅。

## dataExtractionRules
应用可以将此属性设置为 XML 资源，在其中指定规则，以确定在备份或转移操作过程中可以从设备复制哪些文件和目录。

## debuggable
确定应用是否可以调试（即使在处于用户模式的设备上运行时）。如果可以调试，则设为 "true"，否则设为 "false"。默认值为 "false"。

## description
有关应用的用户可读文本，比应用标签更长且描述性更强。该值应设置为对字符串资源的引用。与标签不同，它不能是原始字符串。没有默认值。

## enabled
确定 Android 系统是否可以实例化应用的组件。如果可以实例化，则设为 "true"，否则设为 "false"。如果值为 "true"，则每个组件的 enabled 属性决定了是否启用该组件。如果值为 "false"，则会替换特定于组件的值，并且所有组件都处于停用状态。

## extractNativeLibs
从 AGP 4.2.0 开始，DSL 选项 useLegacyPackaging 取代了 extractNativeLibs 清单属性。请使用应用的 build.gradle 文件中的 useLegacyPackaging（而非清单文件中的 extractNativeLibs）来配置原生库压缩行为。

此属性指示软件包安装程序是否将原生库从 APK 提取到文件系统。如果设置为 "false"，则原生库以未压缩的形式存储在 APK 中。虽然您的 APK 可能较大，但应用加载速度更快，因为库是在应用运行时直接从 APK 加载。
extractNativeLibs 的默认值取决于 minSdkVersion 和您使用的 AGP 版本。在大多数情况下，默认行为很可能符合您的预期，您无需显式设置此属性。

## fullBackupContent
此属性指向一个包含自动备份功能的完整备份规则的 XML 文件。 这些规则决定了备份哪些文件。如需了解详情，请参阅自动备份的 XML 配置语法。
此属性是可选的。如果未指定，默认情况下，自动备份会涵盖应用的大部分文件。如需了解详情，请参阅备份的文件。

## fullBackupOnly
默认值为 "false"。 此属性指示是否在设备上使用自动备份（如果可用）。如果设为 "true"，则应用安装在搭载 Android 6.0（API 级别 23）或更高版本的设备上时会执行自动备份。在旧款设备上，应用会忽略此属性并执行键值对备份。

## gwpAsanMode
默认值为 "never"。此属性指示是否使用 GWP-ASan；GWP-ASan 是一种原生内存分配器功能，可帮助查找释放后使用和堆缓冲区溢出 bug。

## hasCode
确定应用是否包含任何 DEX 代码，即使用 Kotlin 或 Java 编程语言。 如果需要，则设为 "true"，否则设为 "false"。当 值为 "false"，则系统不会尝试加载任何应用 启动组件时调用的代码。默认值为 "true"。
如果应用包含原生 (C/C++) 代码，但没有 DEX 代码，这应该 设为 "false"。如果设为 "true"，在 APK 加载 不包含 DEX 代码，则应用可能无法加载。

## hasFragileUserData
确定是否在用户卸载应用时向用户显示保留应用数据的提示。默认值为 "false"。

## hardwareAccelerated
确定是否为此应用中的所有 activity 和视图启用硬件加速渲染。如果启用，则设为 "true"，否则设为 "false"。如果已将 minSdkVersion 或 targetSdkVersion 设为 "14" 或更高版本，则默认值为 "true"；否则，值为 "false"。

## icon
整个应用的图标，以及每个应用组件的默认图标。此属性应设置为对包含图片的可绘制资源（例如 "@drawable/icon"）的引用。没有默认图标。

## isGame
警告：此属性已废弃。请改用设置为 game 的 appCategory 属性。确定应用是否为游戏。系统可以将分类为游戏的应用归入一组，或者将它们与其他应用分开显示。默认值为 "false"。

## isMonitoringTool
表明此应用旨在监控其他人员。

没有默认值。开发者必须指定以下值之一：
值	说明
"parental_control"	应用的用途是家长控制，专为希望借助此类功能确保孩子安全的家长而设计。
"enterprise_management"	应用的用途是企业管理，专为希望对发放给员工的设备进行管理和跟踪的企业而设计。
"other"	应用的用途是未在此表中另行指定的用例。

## killAfterRestore
默认值为 "true"，这表示应用在全系统恢复期间处理完其数据后会终止。

## largeHeap
应用的进程是否使用大型 Dalvik 堆创建。此属性适用于为应用创建的所有进程。它只适用于加载到进程中的第一个应用。如果您使用共享用户 ID 来让多个应用使用一个进程，则它们必须全部以一致的方式使用此选项，以免产生不可预知的结果。

若要在运行时查询可用内存大小，请使用 getMemoryClass() 或 getLargeMemoryClass() 方法。

## label
整个应用的用户可读标签，以及每个应用组件的默认标签。此标签应设置为对字符串资源的引用，以便可以像界面中的其他字符串一样进行本地化。不过，为了方便您开发应用，也可以将其设为原始字符串。

## logo
整个应用的徽标，以及 activity 的默认徽标。此属性应设置为对包含图片的可绘制资源（例如 "@drawable/logo"）的引用。没有默认徽标。

## manageSpaceActivity
一个 Activity 子类的完全限定名称，系统可以启动该子类来让用户管理设备上的应用占用的内存。此 activity 还使用 `<activity> `元素进行声明。

## name
为应用实现的 Application 子类的完全限定名称。应用进程启动后，此类会在应用的所有组件之前进行实例化。

## networkSecurityConfig
指定包含应用的网络安全配置的 XML 文件的名称。该值是对包含相应配置的 XML 资源文件的引用。

## permission
客户端为了与应用进行交互而需要具备的权限的名称。使用此属性可以方便地设置适用于所有应用组件的权限。可以通过设置各个组件的 permission 属性来将其覆盖。

## persistent
确定应用是否需要始终保持运行状态。如果需要，则设为 "true"，否则设为 "false"。默认值为 "false"。应用通常不会设置此标志。持久性模式仅适用于某些系统应用。

## process
运行应用所有组件所在进程的名称。每个组件都可以通过设置自己的 process 属性来替换此默认值。

通过将此属性设为与其他应用共享的进程名称，您可以安排两个应用的组件在同一进程中运行，但前提是这两个应用还共享用户 ID 并使用同一证书进行了签名。

## restoreAnyVersion
指示应用已准备好尝试恢复任何备份的数据集，即使存储该备份的应用版本比当前安装在设备上的版本要高，也一样恢复。将此属性设为 "true" 可让备份管理器尝试恢复，即使版本不匹配表明数据不兼容也是如此。使用时应格外小心！
此属性的默认值为 "false"。

## requestLegacyExternalStorage

## requiredAccountType
指定应用正常运行所需的账号类型。 如果您的应用需要 Account，则此属性的值必须与您的应用使用的账号身份验证器类型（由 AuthenticatorDescription 定义，如 "com.google"）相对应。
默认值为 null，这表示应用可以在没有任何账号的情况下运行。

## resizeableActivity
指定应用是否支持多窗口模式。您可以在 <activity> 或 <application> 元素中设置此属性。

如果您将此属性设为 "true"，则用户可以在分屏模式和自由窗口模式下启动 activity。如果您将此属性设为 "false"，则无法针对多窗口环境测试或优化应用。在应用了兼容模式的情况下，系统可能仍会将 activity 置于多窗口模式中。

## restrictedAccountType
指定此应用所需的账号类型，并指明受限个人资料可以访问属于所有者用户的此类账号。如果您的应用需要 Account 并且受限个人资料可以访问主要用户的账号，则此属性的值必须与您的应用使用的账号身份验证器类型（由 AuthenticatorDescription 定义，如 "com.google"）相对应。


## supportsRtl
声明您的应用是否愿意支持从右到左 (RTL) 布局。

## taskAffinity
默认情况下，应用中的所有 Activity 具有相同的粘性。该粘性的名称与由 <manifest> 元素设置的软件包名称相同。


## testOnly
指示此应用是否仅用于测试目的。例如，它可能会在自身之外公开功能或数据，这样会导致安全漏洞，但对测试很有用。此类 APK 只能通过 adb 安装，您不能将其发布到 Google Play。

## theme
对样式资源的引用，用于为应用中的所有 activity 定义默认主题。各个 activity 可以通过设置自己的 theme 属性来替换默认值。

## uiOptions
有关 activity 界面的额外选项。必须是以下某个值：

## usesCleartextTraffic
指示应用是否打算使用明文网络流量，如明文 HTTP。 对于目标 API 级别为 27 或更低级别的应用，默认值为 "true"。对于以 API 级别 28 或更高级别为目标的应用，默认值为 "false"。

## vmSafeMode
指示应用是否希望虚拟机 (VM) 在安全模式下运行。默认值为 "false"。


# Activity:属性
声明实现应用部分可视化界面的 Activity（一个 Activity 子类）。所有 activity 都必须在清单文件中用 `<activity>` 元素表示。任何未通过该 API 声明的服务都不会被看到 并且永远不会运行

```xml
<activity android:allowEmbedded=["true" | "false"]
          android:allowTaskReparenting=["true" | "false"]
          android:alwaysRetainTaskState=["true" | "false"]
          android:autoRemoveFromRecents=["true" | "false"]
          android:banner="drawable resource"
          android:canDisplayOnRemoteDevices=["true" | "false"]
          android:clearTaskOnLaunch=["true" | "false"]
          android:colorMode=[ "hdr" | "wideColorGamut"]
          android:configChanges=["colorMode", "density",
                                 "fontScale", "fontWeightAdjustment",
                                 "grammaticalGender", "keyboard",
                                 "keyboardHidden", "layoutDirection", "locale",
                                 "mcc", "mnc", "navigation", "orientation",
                                 "screenLayout", "screenSize",
                                 "smallestScreenSize", "touchscreen", "uiMode"]
          android:directBootAware=["true" | "false"]
          android:documentLaunchMode=["intoExisting" | "always" |
                                  "none" | "never"]
          android:enabled=["true" | "false"]
          android:enabledOnBackInvokedCallback=["true" | "false"]
          android:excludeFromRecents=["true" | "false"]
          android:exported=["true" | "false"]
          android:finishOnTaskLaunch=["true" | "false"]
          android:hardwareAccelerated=["true" | "false"]
          android:icon="drawable resource"
          android:immersive=["true" | "false"]
          android:label="string resource"
          android:launchMode=["standard" | "singleTop" |
                              "singleTask" | "singleInstance" | "singleInstancePerTask"]
          android:lockTaskMode=["normal" | "never" |
                              "if_whitelisted" | "always"]
          android:maxRecents="integer"
          android:maxAspectRatio="float"
          android:multiprocess=["true" | "false"]
          android:name="string"
          android:noHistory=["true" | "false"]  
          android:parentActivityName="string" 
          android:persistableMode=["persistRootOnly" | 
                                   "persistAcrossReboots" | "persistNever"]
          android:permission="string"
          android:process="string"
          android:relinquishTaskIdentity=["true" | "false"]
          android:requireContentUriPermissionFromCaller=["none" | "read" | "readAndWrite" |
                                                         "readOrWrite" | "write"] 
          android:resizeableActivity=["true" | "false"]
          android:screenOrientation=["unspecified" | "behind" |
                                     "landscape" | "portrait" |
                                     "reverseLandscape" | "reversePortrait" |
                                     "sensorLandscape" | "sensorPortrait" |
                                     "userLandscape" | "userPortrait" |
                                     "sensor" | "fullSensor" | "nosensor" |
                                     "user" | "fullUser" | "locked"]
          android:showForAllUsers=["true" | "false"]
          android:stateNotNeeded=["true" | "false"]
          android:supportsPictureInPicture=["true" | "false"]
          android:taskAffinity="string"
          android:theme="resource or theme"
          android:uiOptions=["none" | "splitActionBarWhenNarrow"]
          android:windowSoftInputMode=["stateUnspecified",
                                       "stateUnchanged", "stateHidden",
                                       "stateAlwaysHidden", "stateVisible",
                                       "stateAlwaysVisible", "adjustUnspecified",
                                       "adjustResize", "adjustPan"] >   
    ...
</activity>
```

## allowEmbedded
表示该 activity 可作为另一个 activity 的嵌入式子项启动 activity，特别是在子项位于容器中的情况下， 例如，由另一个 activity 拥有的 Display。例如，用于 Wear 自定义通知的 activity 必须声明此属性，以便 Wear 在其位于另一进程内的上下文流中显示 activity。

此属性的默认值为 false。

## allowTaskReparenting
当下一次将启动 activity 的任务转至前台时，activity 是否能从该任务转移至与其有相似性的任务。"true" 表示可以转移，"false" 表示仍须留在启动它的任务处。

## configChanges
列出 activity 将自行处理的配置变更。在运行时发生配置变更时，默认情况下会关闭 activity 并将其重启，但使用该属性声明配置将阻止 activity 重启。相反，activity 会保持运行状态，并且系统会调用其 onConfigurationChanged() 方法。

下列字符串是该属性的有效值。若有多个值，则使用 | 进行分隔，例如 "locale|navigation|orientation"。

值	说明
"colorMode"	屏幕的颜色模式功能（色域或动态范围）已更改。

"density"	显示密度的更改，例如当用户指定 显示比例不同，或者不同显示屏当前处于活跃状态。


"fontScale"	字体缩放比例的更改，例如当用户选择新的全局字体大小时。
"fontWeightAdjustment"	字体粗细的增加量已更改。
"grammaticalGender"	语言的语法性别已更改。

"keyboard"	键盘类型的更改，例如当用户插入外接键盘时。
"keyboardHidden"	键盘无障碍功能的更改，例如当用户显示硬件键盘时。
"layoutDirection"	布局方向的更改，例如从 从左到右 (LTR) 到从右到左 (RTL)。

"locale"	语言区域的更改，例如当用户选择显示文本所用的新语言时。
"mcc"	当系统检测到用于更新 MCC 的 SIM 卡时，IMSI 移动设备国家/地区代码 (MCC) 发生的更改。
"mnc"	当系统检测到用于更新 MNC 的 SIM 卡时，IMSI 移动网络代码 (MNC) 发生的更改。
"navigation"	导航类型（轨迹球或方向键）的 TA 更改。通常不会出现这种情况。
"orientation"	屏幕方向的更改，例如用户旋转设备时。

"screenLayout"	屏幕布局的更改，例如在其他屏幕变为活动状态时。
"screenSize"	当前可用屏幕尺寸的更改。

该值表示目前可用尺寸相对于当前宽高比的变更，当用户在横向模式与纵向模式之间切换时，它便会发生变更。

"smallestScreenSize"	实体屏幕尺寸的更改。

该值表示与方向无关的尺寸变更，因此它只有在实际物理屏幕尺寸发生变更（如切换到外部显示器）时才会变化。对此配置所作变更对应 smallestWidth 配置的更改。

"touchscreen"	触摸屏的更改。通常不会出现这种情况。
"uiMode"	界面模式的更改，例如当用户将设备放到桌面或车载基座上时，或者夜间模式发生变化时。如需了解有关不同界面模式的更多信息，请参阅 UiModeManager。

## name
实现 activity 的类的名称，是 Activity 的子类。属性值通常是完全限定的类名称，例如 "com.example.project.ExtracurricularActivity"。不过，作为一种简写形式，如果名称的第一个字符是句点（例如 ".ExtracurricularActivity"），则会附加到 build.gradle 文件中指定的命名空间。
发布应用后，除非您已设置 android:exported="false"，否则请勿更改此名称。没有默认值。必须指定此名称。

## label
一种用户可读的 activity 标签。该标签会在系统向用户呈现 activity 时显示在屏幕上。此标签通常与 activity 图标一并显示。如果未设置此属性，则整个应用的标签集为 。请查看 `<application> `元素的 label 属性。

activity 的标签（无论是在此处设置，还是由 `<application>` 元素设置）同时也是 activity 所有 intent 过滤器的默认标签。如需了解详情，请参考 `<intent-filter>` 元素的 label 属性。

## windowSoftInputMode
Activity 的主窗口与包含屏幕软键盘的窗口之间的交互方式。该设置必须是下表所列的其中一项值，或一个 "state..." 值加上一个 "adjust..." 值的组合。在任一组中设置多个值（例如，多个 "state..." 值）均会产生未定义的结果。 各个值之间用竖线 (|) 分隔，如以下示例所示：

"stateUnspecified"	不指定软键盘的状态是隐藏还是可见。系统会选择合适的状态，或依赖主题中的设置。
这是对软键盘行为的默认设置。

"stateUnchanged"	当 activity 转至前台时保留软键盘最后所处的任何状态，无论是可见还是隐藏。
"stateHidden"	当用户选择 activity 时（换言之，当用户确实是向前导航到 activity，而不是因离开另一 activity 而返回时）隐藏软键盘。
"stateAlwaysHidden"	当 activity 的主窗口有输入焦点时始终隐藏软键盘。
"stateVisible"	当用户选择 activity 时（换言之，当用户确实是向前导航到 activity，而不是因离开另一 activity 而返回时）显示软键盘。
"stateAlwaysVisible"	当窗口获得输入焦点时，会显示软键盘。
"adjustUnspecified"	不指定 activity 的主窗口是否通过调整尺寸为软键盘腾出空间，或者是否通过平移窗口内容以在屏幕上显示当前焦点。 根据窗口的内容是否存在任何可滚动其内容的布局视图，系统会自动选择其中一种模式。如果存在这种视图，系统会调整窗口尺寸，前提是可通过滚动操作在较小区域内看到窗口的所有内容。
这是对主窗口行为的默认设置。

"adjustResize"	始终调整 activity 主窗口的尺寸，以为屏幕上的软键盘腾出空间。
"adjustPan"	不通过调整 activity 主窗口的尺寸为软键盘腾出空间。相反，窗口的内容会自动平移，使键盘永远无法遮盖当前焦点，以便用户始终能看到自己输入的内容。这通常不如调整窗口尺寸可取，因为用户可能需关闭软键盘才能进入被遮盖的窗口部分，并与之进行交互。

## launchMode
有关 activity 如何启动的说明。共有五种模式 与 activity 标志（FLAG_ACTIVITY_* 常量）协同工作 在 Intent 对象中，以确定 系统会调用 activity 来处理 intent：

"standard"
"singleTop"
"singleTask"
"singleInstance"
"singleInstancePerTask"

默认模式为 "standard"。

如下表所示，这些模式可分为两大类：
"standard" 和 "singleTop" activity 是一类，
"singleTask"、"singleInstance" 和 "singleInstancePerTask" activity 是另一类。

启动模式为 "standard" 或 "singleTop" 的 activity 可以多次实例化。

启动模式为 "singleTask" 的 activity 结合了 "singleInstance" 和 "singleInstancePerTask" 的行为：activity 可以多次实例化，并且可以位于具有相同 taskAffinity 的任务中的任意位置。但是，设备只能保留一个用于在 activity 任务的根位置查找 "singleTask" activity 的任务。

# intent-filter
指定 activity、服务或广播接收器可以响应的 intent 类型。intent 过滤器声明其父组件的功能：activity 或服务可执行哪些操作，以及接收器可处理哪些类型的广播。
它让组件可以接收所通告类型的 intent，同时过滤掉对组件没有意义的 intent。过滤器的大部分内容由它的 `<action>、<category> 和 <data>` 子元素进行描述。

```xml
<intent-filter android:icon="drawable resource"
               android:label="string resource"
               android:priority="integer" >
    ...
</intent-filter>
```
## priority
为父组件指定的优先级，以便系统据此处理过滤器所描述类型的 intent。此属性对 activity 和广播接收器都有意义。

## order
当多个过滤器匹配时过滤器的处理顺序。


# action
向 Intent 过滤器添加操作。 `<intent-filter>` 元素必须包含一个或多个 `<action>` 元素。如果 Intent 过滤器中没有 `<action>` 元素，则过滤器不接受任何 Intent 对象。 

```xml
<action android:name="string" />
```
## name
操作的名称。某些标准操作在 Intent 类中定义为 ACTION_string 常量。若要将其中一项操作分配给此属性，请在 ACTION_ 之后的 string 前面加上 android.intent.action.。 例如，对于 ACTION_MAIN，请使用 android.intent.action.MAIN；对于 ACTION_WEB_SEARCH，请使用 android.intent.action.WEB_SEARCH。


# meta-data：属性
一个组件元素可以包含任意数量的 `<meta-data>` 子元素。所有这些子元素的值收集到一个 Bundle 对象，并且可作为 PackageItemInfo.metaData 字段提供给组件。

```xml
<meta-data android:name="string"
           android:resource="resource specification"
           android:value="string" />
```
通过 value 属性指定普通值。若要将资源 ID 指定为值，请改为使用 resource 属性。例如，以下代码会将 @string/kangaroo 资源中存储的任何值分配给 zoo 名称：

```xml
<meta-data android:name="zoo" android:value="@string/kangaroo" />
```
另一方面，使用 resource 属性会为 zoo 分配资源的数字 ID，而不是资源中存储的值：

```xml
<meta-data android:name="zoo" android:resource="@string/kangaroo" />
```

强烈建议不要以多个单独 `<meta-data>` 条目的形式提供相关数据。相反，如果您有要与组件相关联的复杂数据，请将其存储为资源，并使用 resource 属性告知组件其 ID。

## name
该项的唯一名称。若要保持名称的唯一性，请使用 Java 样式的命名惯例，例如“com.example.project.activity.fred”。

## resource
对资源的引用。资源的 ID 是分配给该项的值。系统会使用 Bundle.getInt() 方法从元数据 Bundle 中检索 ID。

## value
分配给该项的值。下表列出了可以指定为值的数据类型以及组件可以用来检索这些值的 Bundle 方法：
类型	Bundle 方法
字符串：使用双反斜杠 (\\) 转义字符，例如 \\n 表示新行，\\uxxxxx 表示 Unicode 字符	getString()
整数：例如 100	getInt()
布尔值：true 或 false	getBoolean()
颜色：采用 #rgb、#argb、#rrggbb 或 #aarrggbb 格式	getInt()
浮点数：例如 1.23	getFloat()

# activity-alias：属性
activity 的别名，由 targetActivity 属性命名。目标必须与别名位于同一应用中，并在清单中的别名之前进行声明。
别名将目标 activity 呈现为独立的实体，并且可以具有自己的一组 intent 过滤器。这些 intent 过滤器（而不是目标 activity 本身的 intent 过滤器）决定了哪些 intent 可以通过别名激活目标，以及系统如何处理别名。
```xml
<activity-alias android:enabled=["true" | "false"]
                android:exported=["true" | "false"]
                android:icon="drawable resource"
                android:label="string resource"
                android:name="string"
                android:permission="string"
                android:targetActivity="string" >
    ...
</activity-alias>
```
## enabled
确定系统是否可以通过此别名实例化目标 activity。如果可以，则设为 "true"，否则设为 "false"。默认值为 "true"。

## exported
确定其他应用的组件是否可以通过此别名启动目标 activity。如果可以，则设为 "true"，否则设为 "false"。 如果设为 "false"，则只有与别名在同一应用中的组件或具有同一用户 ID 的应用的组件可以通过别名启动目标 activity。

## icon
通过别名呈现给用户时，目标 activity 的图标。

## label
通过别名呈现给用户时，别名的用户可读标签。

## name
别名的唯一名称。该名称类似于完全限定的类名称。但是，与目标 activity 的名称不同，别名名称是任意的，它不引用实际类。

## permission
客户端要使用别名启动目标 activity 或让其执行某项操作而必须具备的权限的名称。如果没有向 startActivity() 或 startActivityForResult() 的调用方授予指定的权限，目标 activity 不会激活。
此属性会取代为目标 activity 本身设置的任何权限。如果未设置此属性，则无需权限即可通过别名激活目标。

## targetActivity
可通过别名激活的 activity 的名称。此名称必须与清单中别名前面的 `<activity>` 元素的 name 属性匹配。

是为activity创建快捷方式的，如下实例：
 
```xml
<activity android:name=".shortcut">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
            </intent-filter>
</activity>
 
 <activity-alias android:name=".CreateShortcuts" android:targetActivity=".shortcut" android:label="@string/shortcut">
    <intent-filter>
             <action android:name="android.intent.action.CREATE_SHORTCUT" />
             <category android:name="android.intent.category.DEFAULT" />
     </intent-filter>
 </activity-alias>
```

# Service:属性
service与activity同级，与activity不同的是，它不能自己启动的，运行在后台的程序，如果我们退出应用时，Service进程并没有结束，它仍然在后台运行。比如听音乐，网络下载数据等，都是由service运行的。

service生命周期：Service只继承了onCreate(),onStart(),onDestroy()三个方法，第一次启动Service时，先后调用了onCreate(),onStart()这两个方法，当停止Service时，则执行onDestroy()方法，如果Service已经启动了，当我们再次启动Service时，不会在执行onCreate()方法，而是直接执行onStart()方法。

service与activity间的通信

Service后端的数据最终还是要呈现在前端Activity之上的，因为启动Service时，系统会重新开启一个新的进程，这就涉及到不同进程间通信的问题了(AIDL)，Activity与service间的通信主要用IBinder负责。

```xml
<service android:description="string resource"
         android:directBootAware=["true" | "false"]
         android:enabled=["true" | "false"]
         android:exported=["true" | "false"]
         android:foregroundServiceType=["camera" | "connectedDevice" |
                                        "dataSync" | "health" | "location" |
                                        "mediaPlayback" | "mediaProjection" |
                                        "microphone" | "phoneCall" |
                                        "remoteMessaging" | "shortService" |
                                        "specialUse" | "systemExempted"]
         android:icon="drawable resource"
         android:isolatedProcess=["true" | "false"]
         android:label="string resource"
         android:name="string"
         android:permission="string"
         android:process="string" >
    ...
</service>
```

## description
描述服务的用户可读字符串。此说明应设置为对字符串资源的引用，以便可以像界面中的其他字符串一样进行本地化。


## directBootAware
默认值为 "false"。

确定服务是否可感知直接启动，也就是说，它是否可以在用户解锁设备之前运行。

## enabled
确定系统是否可以实例化服务。如果可以实例化，则设为 "true"，否则设为 "false"。默认值为 "true"。

## exported
确定其他应用的组件是否可以调用服务或与之交互。如果可以，则设为 "true"，否则设为 "false"。当该值为 "false" 时，只有同一个应用或具有相同用户 ID 的应用的组件可以启动服务或绑定到服务。

默认值取决于服务是否包含 intent 过滤器。没有任何过滤器意味着该服务只能通过指定其确切的类名称进行调用。这意味着服务仅供应用内部使用，因为其他应用不知道其类名称。因此，在这种情况下，默认值为 "false"。 另一方面，如果存在至少一个过滤器，则意味着该服务会供外部使用，所以默认值为 "true"。

此属性并非是唯一限制向其他应用披露服务的方式。您还可以使用权限来限制可以与服务交互的外部实体。

## foregroundServiceType
阐明服务是满足特定用例要求的前台服务。例如，"location" 类型的前台服务表示应用正在获取设备的当前位置，目的通常是继续用户发起的操作，且该操作与设备位置相关。

## icon
表示服务的图标。此属性应设置为对包含图片定义的可绘制资源的引用。如果未设置此属性，则改用为整个应用指定的图标。请参阅 <application> 元素的 icon 属性。

## isolatedProcess
如果设置为 "true"，则此服务会在与系统其余部分隔离的特殊进程下运行。此服务自身没有权限，唯一与其通信的方式是通过 Service API 进行绑定和启动。

## label
服务的用户可读名称。如果未设置此属性，则改用整个应用的标签集。请参阅 <application> 元素的 label 属性。此标签应设置为对字符串资源的引用，以便可以像界面中的其他字符串一样进行本地化。不过，为了方便您开发应用，也可以将其设为原始字符串。

## name
实现服务的 Service 子类的名称。这是一个完全限定的类名称，例如 "com.example.project.RoomService"。不过，作为一种简写形式，如果名称的第一个字符是句点（例如 ".RoomService"），则会将其附加到 <manifest> 元素中指定的软件包名称。

## permission
实体启动服务或绑定到服务所需的权限的名称。如果没有向 startService()、bindService() 或 stopService() 的调用方授予此权限，该方法将不起作用，且系统不会将 Intent 对象传送给服务。

如果未设置该属性，则对服务应用由 `<application>` 元素的 permission 属性所设置的权限。如果二者均未设置，则服务不受权限保护。

## process
运行服务的进程的名称。通常，应用的所有组件都会在为应用创建的默认进程中运行。它与应用软件包的名称相同。`<application>` 元素的 process 属性可以为所有组件设置不同的默认值。不过，组件可以使用自己的 process 属性替换默认属性，从而允许您跨多个进程分布应用。如果为此属性分配的名称以英文冒号 (:) 开头，则系统会在需要时创建应用专用的新进程，并且服务会在该进程中运行。

如果进程名称以小写字符开头，则服务将在采用该名称的全局进程中运行，前提是它具有相应权限。这样，不同应用中的组件就可以共享进程，从而减少资源使用量。

# Receiver:属性

将广播接收器（BroadcastReceiver 子类）声明为应用的组件之一。广播接收器允许应用接收由系统或其他应用广播的 intent，即使应用的其他组件并没有运行也是如此。

# Provider:属性
声明 content provider 组件。content provider 是 ContentProvider 的子类，可提供对由应用管理的数据的结构化访问机制。应用中的所有 content provider 都必须在清单文件的 `<provider>` 元素中定义。否则，系统将不知道它们，也不会运行它们。

您只能声明属于您的应用的 content provider，请勿在应用中使用的其他应用中声明 content provider。

Android 系统根据授权方字符串（提供程序的内容 URI 的一部分）来存储对 content provider 的引用。例如，假设您想要访问用来存储专业医护人员相关信息的 content provider。为此，您可以调用 ContentResolver.query() 方法，它接受用来标识提供程序的 URI（以及其他参数）：

content://com.example.project.healthcareprovider/nurses/rn

content: 架构将 URI 标识为指向 Android Content Provider 的内容 URI。授权方 com.example.project.healthcareprovider 标识提供程序本身。Android 系统会在已知提供程序及其授权方的列表中查询该授权方。子字符串 nurses/rn 是一个路径，content provider 用它来标识提供程序数据的子集。

在 `<provider>`元素中定义提供程序时，请勿在 android:name 参数中添加架构或路径，只能添加授权方。

## authorities
一个或多个 URI 授权方的列表，这些 URI 授权方用于标识 content provider 提供的数据。列出多个授权方时，用分号将其名称分隔开来。为避免冲突，授权方名称应遵循 Java 样式的命名惯例。

# uses-library
指定应用必须与之关联的共享库。 此元素告知系统将库的代码添加到软件包的类加载器中。所有 android 软件包（例如 android.app、android.content、android.view 和 android.widget）都位于所有应用自动与之关联的默认库中。不过，某些软件包（例如 maps）位于未自动关联的不同库中。

`<uses-library>` 标记的顺序非常重要。它会影响应用加载时的类查询和解析顺序。某些库可能有重复的类，在这种情况下，位置在前的库优先。

此元素还会影响应用在特定设备上安装以及应用在 Google Play 上的可用性。如果此元素存在并且其 android:required 属性设置为 "true"，则 PackageManager 框架将不允许用户安装应用，除非用户设备上存在相应的库。

## name
库的名称。此名称由您使用的软件包的文档提供。例如，"android.test.runner"，这是个包含 Android 测试类的软件包。

## required
布尔值，指示应用是否需要 android:name 指定的库。
+ "true"：如果没有此库，则应用将无法正常运行。系统不允许将应用安装在没有此库的设备上。
+ "false"：如果此库存在，应用会使用此库，但必要时也可在没有此库的情况下运行。 系统允许安装应用，即使不存在此库也是如此。如果您使用 "false"，则需要在运行时检查有没有此库。

# supports-screens
支持多屏幕机制

# uses-configuration
指示应用所需的硬件和软件功能。例如，应用可能会指定它需要物理键盘或特定的导航设备（例如轨迹球）。该规范用于避免将应用安装在它无法正常运行的设备上。

# uses-sdk
Google Play 会利用在您的应用清单中声明的 `<uses-sdk>` 属性，从不符合其平台版本要求的设备上滤除您的应用。在设置这些属性前，请确保您了解 Google Play 过滤器。

# instrumentation
定义一些用于探测和分析应用性能等等相关的类，可以监控程序。在各个应用程序的组件之前instrumentation类被实例化








