## configurationDev
项目的主要作用是对build编译的项目进行集群管理，使目录上、项目间的lib、远程lib的统一管理。主要用于复杂、多变、模块化的多模块项目

项目中内置的include方法，替换了传统settings的配置。
更有addConfigExtensions和addCustomExtensions配置方法，通过指定项目类型和项目名进行全局统一默认配置

该项目把编译一个模块的类型从原本固定在编译器配置文件中转变到代码中，使开发者可以更灵活的定制项目配置和编译过程

下图是我个人项目中应用
![image](https://github.com/Daimhim/resources/blob/main/configurationDev/1C1ECC437D0D442B846EDDA2A7929978.png)

在model中的使用
![image](https://github.com/Daimhim/resources/blob/main/configurationDev/7F0FA3A6B6E24E7AAEE0AD9D3A939E2D.png)

## 通过git子模块功能吧configurationDev导入项目的根目录

在项目根目录执行，将configurationDev导入。尾端的configurationDev参数代表文件夹，通常不需要加
```
git submodule add https://github.com/Daimhim/configurationDev.git configurationDev
```
几个简单的子模块操作
```
git submodule update //更新子模块
git submodule update --remote //更新子模块为远程项目的最新版本
```
克隆包含子模块的项目，包含子模块
```
git clone https://github.com/Daimhim/StandByMe.git C --recursive
```
###### 注意，子项目在每次克隆之后先切换到指定分支，以免提交创建错误的分支

## 第一步：新建config.gradle

在根目录新建config.gradle文件

```
def rootPath = ""
try {
    def projectName = "StandByMe"
    def rootDir = new StringBuffer(rootProject.rootDir.toString())
    def index = rootDir.indexOf(projectName)
    rootPath = rootDir.substring(0, index+projectName.length())
} catch (MissingPropertyException e) {

}
apply from: "$rootPath/configurationDev/configuration.gradle"
// 以上都是模板代码

// 引入的项目
include("app", ":app", KOTLIN_MAIN_LIB, "org.daimhim.standbyme")

// 根据类型自定义配置
addConfigExtensions({
    project.dependencies {
        api androidxDependencies.appcompat
        api androidxDependencies.material
        api androidxDependencies.constraintlayout
        api androidxDependencies.swiperefreshlayout
        api androidxDependencies.lifecycle_livedata_ktx
        api androidxDependencies.lifecycle_viewmodel_ktx
        api androidxDependencies.multidex
        api androidxDependencies.navigation_fragment
        api androidxDependencies.navigation_ui
        api androidxDependencies.recyclerview
        api globalDependencies.timber
    }
},KOTLIN_MAIN_LIB)


```
#### 这段代码直接复制就行

```
def rootPath = ""
try {
    def projectName = "StandByMe"
    def rootDir = new StringBuffer(rootProject.rootDir.toString())
    def index = rootDir.indexOf(projectName)
    rootPath = rootDir.substring(0, index+projectName.length())
} catch (MissingPropertyException e) {

}
apply from: "$rootPath/configurationDev/configuration.gradle"
// 以上都是模板代码
```

#### 在父类中提供三个方法，分别是
+ include
+ addConfigExtensions
+ addCustomExtensions

#### include

主要作用是导入项目


```
include("app", ":app", KOTLIN_MAIN_LIB, "org.daimhim.standbyme")
```
+ app： 代表项目名，可以自定义。名字不能违反java变量名规范
+ :app： 项目的绝对路径，有几层就加几个:
+ KOTLIN_MAIN_LIB： 项目类型，目前支持13种类型，在最后统一展示
+ org.daimhim.standbyme：applicationid，除了application项目外，其他项目类型可不传

#### addConfigExtensions

主要作用是通过项目类型，提前全局统一配置

```
addConfigExtensions({
    project.dependencies {
        api globalDependencies.timber
    }
},KOTLIN_MAIN_LIB,KOTLIN_MAIN_LIB)
```
+ {}  接口，在内部可以使用使用project的闭包
+ KOTLIN_MAIN_LIB：项目类型，是可变参，可以传入多个类型

#### addCustomExtensions
与addConfigExtensions类似，主要作用是通过项目名称，提前全局统一配置

```
addCustomExtensions({
    project.dependencies {
        implementation globalDependencies.retrofit
    }
},"home")
```

## 第二步：修改settings

settings.gradle

#### 复制就行
```
rootProject.name = "StandByMe"
apply from: "config.gradle"
ext.modules.each{ k, v ->
    println "${v.libType} ${v.name} ${v.include}"
    switch (v.libType){
        case ext.JAVA_MAIN_LIB:
        case ext.KOTLIN_MAIN_LIB:
            if (IS_MAIN_DEV == "true"){
                include(v.include)
            }
            break
        default:
            include(v.include)
            break
    }
}
```
修改成自己的项目名
```
rootProject.name = "StandByMe"
```
## 第三步：修改模块

#### 模块的build.gradle

##### 普通lib

```
apply from: "../configurationDev/build_config.gradle"

android {
    defaultConfig {
        versionCode 1
        versionName "1.0"
    }
}

dependencies {
    implementation project(path: basementDependencies.snail_core)
}
```
##### 可运行的app模块

```
apply from: "../configurationDev/build_config.gradle"

android {

    defaultConfig {
        versionCode 1
        versionName "1.0"
    }

}

dependencies {
    implementation 'androidx.vectordrawable:vectordrawable:1.1.0'
}
```
其中 ../ 根据项目距离根目录的层级所决定

#### 根目录的gradle.properties
```
# true 合并模式 app false 各自开发模式
IS_MAIN_DEV = false
```


## 项目所支持的module类型
类型 | 含义
---|---
JAVA_MAIN_LIB  |  ： 纯app
KOTLIN_MAIN_LIB  |  ： 纯app
JAVA_LIB  |  ： 纯java lib
KOTLIN_LIB  |  ： 纯kotlin lib
JAVA_SDK  |  ： 纯java lib带android sdk
KOTLIN_SDK  |  ： 纯kotlin lib带Android sdk
BUILD_SRC  |  ： IDEA特性项目
GROOVY_LIB  |  ： 纯Groovy gradle插件项目
JAVA_MODEL_LIB  |  ： 纯java 业务lib
JAVA_MODEL_TRANSFORM_LIB  |  ： 纯java 可独立业务lib
KOTLIN_MODEL_LIB  |  ： 纯kotlin 业务lib
KOTLIN_MODEL_TRANSFORM_LIB  |  ： 纯kotlin 可独立业务lib
OTHER_LIB  |  ：  其它

以下除了语言区别外，根据根目录的gradle.properties的IS_MAIN_DEV动态切换
```
JAVA_MODEL_LIB
JAVA_MODEL_TRANSFORM_LIB
KOTLIN_MODEL_LIB
KOTLIN_MODEL_TRANSFORM_LIB
```

## 项目内置的包

```
//Android原生依赖
    androidxDependencies = [
            'appcompat'              : 'androidx.appcompat:appcompat:1.2.0',
            'material'               : 'com.google.android.material:material:1.3.0',
            'constraintlayout'       : 'androidx.constraintlayout:constraintlayout:2.0.4',
            'swiperefreshlayout'     : 'androidx.swiperefreshlayout:swiperefreshlayout:1.0.0',
            'lifecycle_livedata_ktx' : 'androidx.lifecycle:lifecycle-livedata-ktx:2.3.1',
            'lifecycle_viewmodel_ktx': 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.3.1',
            'multidex'               : 'androidx.multidex:multidex:2.0.1',
            'navigation_fragment'    : 'androidx.navigation:navigation-fragment:2.3.5',
            'navigation_ui'          : 'androidx.navigation:navigation-ui:2.3.5',
            'recyclerview'           : 'androidx.recyclerview:recyclerview:1.2.1',
    ]
    //Android 基础模块
    kotlinAndroidDependencies = [
            'core_ktx': 'androidx.core:core-ktx:1.3.2',
    ]
    // kotlin 基础模块
    kotlinJavaDependencies = [
            'kotlin_stdlib': "org.jetbrains.kotlin:kotlin-stdlib:1.4.32",
    ]
    def camerax_version = '1.1.0-alpha04'
    def j_push_version = '4.0.6'
    //统一的lib 版本管理 除apk模块以外 其他模块依赖时请使用 编译模式
    globalDependencies = [
            'coil'                     : "io.coil-kt:coil:1.2.0",
            'timber'                   : 'com.jakewharton.timber:timber:4.7.1',
            'gson'                     : 'com.google.code.gson:gson:2.8.6',
            'kotlinpoet'               : 'com.squareup:kotlinpoet:1.7.2',
            'javapoet'                 : 'com.squareup:javapoet:1.13.0',
            'retrofit'                 : 'com.squareup.retrofit2:retrofit:2.9.0',
            'converter_gson'           : 'com.squareup.retrofit2:converter-gson:2.9.0',
            'okhttp'                   : 'com.squareup.okhttp3:okhttp:3.12.13',
            'logging_interceptor'      : "com.squareup.okhttp3:logging-interceptor:3.12.13",
            'flexbox'                  : 'com.google.android:flexbox:2.0.1',
            // CameraX core library
            'camera_core'              : "androidx.camera:camera-core:$camerax_version",
            'camera_camera2'           : "androidx.camera:camera-camera2:$camerax_version",
            'camera_lifecycle'         : "androidx.camera:camera-lifecycle:$camerax_version",
            'camera_extensions'        : 'androidx.camera:camera-extensions:1.0.0-alpha24',
            'camera_view'              : 'androidx.camera:camera-view:1.0.0-alpha24',
            //极光推送
            'jcore'                    : 'cn.jiguang.sdk:jcore:2.8.0',
            'jpush'                    : "cn.jiguang.sdk:jpush:$j_push_version",//极光推送
            'huawei_push'              : 'com.huawei.hms:push:5.1.1.301',
            'huawei_plugin'            : "cn.jiguang.sdk.plugin:huawei:$j_push_version",
            'xiaomi_plugin'            : "cn.jiguang.sdk.plugin:xiaomi:$j_push_version",
            'oppo_plugin'              : "cn.jiguang.sdk.plugin:oppo:$j_push_version",
            'vivo_plugin'              : "cn.jiguang.sdk.plugin:vivo:$j_push_version",
            //https://github.com/Nightonke/BoomMenu
            'boommenu'                 : 'com.nightonke:boommenu:2.0.3',
            //数据库
            'room_runtime'             : "android.arch.persistence.room:runtime:1.1.1",
            'room_compiler'            : "android.arch.persistence.room:compiler:1.1.1",
            //地图
            'amap_3dmap'               : 'com.amap.api:3dmap:7.7.0',
            'amap_location'            : 'com.amap.api:location:5.0.0',
            'amap_search'              : 'com.amap.api:search:7.3.0',
            //统计
            'mtj_sdk_circle'           : 'com.baidu.mobstat:mtj-sdk-circle:4.0.2.2',
            //bugly
            'crashreport'              : 'com.tencent.crashreport:crashreport:3.3.7',
            'nativecrashreport'        : 'com.tencent.bugly:nativecrashreport:3.8.0',
            //跑马灯，可以垂直方向跑
            'marqueeview'              : 'com.sunfusheng:MarqueeView:1.4.1',
            //https://github.com/EverythingMe/overscroll-decor
            // 滚动视图 用于RecyclerView, ListView, GridView, ScrollView
            'overscroll_decor'         : 'io.github.everythingme:overscroll-decor-android:1.1.1',
            //https://github.com/leolin310148/ShortcutBadger
            //桌面角标
            'shortcut_badger'          : "me.leolin:ShortcutBadger:1.1.22@aar",
            //https://github.com/Baseflow/PhotoView
            //图片缩放控件
            'photoview'                : "com.github.chrisbanes.photoview:library:1.2.4",
            //https://github.com/AnderWeb/discreteSeekBar
            //拖动进度条，有进度显示泡泡
            'discrete_seekbar'         : 'org.adw.library:discrete-seekbar:1.0.1',
            //https://github.com/yangpeixing/YImagePicker
            //微信UI样式图片和视频选择
            'yimagepicker'             : 'com.ypx.yimagepicker:androidx:3.1.4',
            //https://gitee.com/z8806c/ImagePicker
            //图片选择器
            'imagePicker'              : 'com.gitee.harisucici:ImagePicker:0.0.2',
            //glide 高斯模糊
            'glide_transformations'    : 'jp.wasabeef:glide-transformations:3.0.1',
            //侧滑库
            'swipe_reveal_layout'      : 'com.chauthai.swipereveallayout:swipe-reveal-layout:1.4.0',
            //解决LiveData数据倒灌
            'livedataplus'             : 'org.daimhim.livedataplus:livedataplus:1.0.1',
            //阿里云热修复
            'alicloud_android_hotfix'  : 'com.aliyun.ams:alicloud-android-hotfix:3.2.15',
            'tagsoup'                  : 'org.ccil.cowan.tagsoup:tagsoup:1.2.1',
            // https://mvnrepository.com/artifact/org.ccil.cowan.tagsoup/tagsoup css解析
            ////解析HTML
            'jsoup'                    : 'org.jsoup:jsoup:1.12.1',
            //扫描车牌ui
            'scanner'                  : 'com.shouzhong:Scanner:1.1.3',
            // 车牌识别库so
            'scanner_license_plate_lib': 'com.shouzhong:ScannerLicensePlateLib:1.0.3',
            //浏览器控件
            'web_view_lib'             : 'cn.yc:WebViewLib:1.4.0',
            //阿里云视频上传
            'aliyun_video_upload'      : 'com.aliyun.video.android:upload:1.6.0',
            //DK 播放器 必选，内部默认使用系统mediaplayer进行解码
            'dkplayer_java'            : 'com.github.dueeeke.dkplayer:dkplayer-java:3.2.6',
            //DK 播放器UI框架
            'dkplayer_ui'              : 'com.github.dueeeke.dkplayer:dkplayer-ui:3.2.6',
            //ijkplayer进行解码
            'player_ijk'               : 'com.github.dueeeke.dkplayer:player-ijk:3.2.6',
            // exoplayer-core
            'exoplayer_core'           : 'com.google.android.exoplayer:exoplayer-core:2.13.3',
            // extension-okhttp
            'extension_okhttp'         : 'com.google.android.exoplayer:extension-okhttp:2.13.3',
            //ARouter  https://github.com/alibaba/ARouter
            'arouter_api'              : 'com.alibaba:arouter-api:1.5.1',
            'arouter_compiler'         : 'com.alibaba:arouter-compiler:1.5.1',
            'auto_service_annotations' : 'com.google.auto.service:auto-service-annotations:1.0-rc6',
            'auto_service'             : 'com.google.auto.service:auto-service:1.0-rc6',
            'android'                  : 'com.google.android:android:4.1.1.4',
            'mmkv_static'              : 'com.tencent:mmkv-static:1.2.10',
            'java_websocket'           : "org.java-websocket:Java-WebSocket:1.5.1",
    ]
    //Android 单元和UI测试
    androidTest = [
            'junit'         : 'junit:junit:4.13.2', //testImplementation
            'androidx_junit': 'androidx.test.ext:junit:1.1.2', //androidTestImplementation
            'espresso_core' : 'androidx.test.espresso:espresso-core:3.3.0', //androidTestImplementation
    ]
```

使用内置的包
```
dependencies {
    // 引用自定义的本地扩展项目
    implementation project(path: basementDependencies.provider)
    implementation globalDependencies.gson
    api globalDependencies.converter_gson
}
```
也可以在config中自定义

```
ext {
    //基础模块
    basementDependencies = [
            'provider'  : modules['provider'].include,
            'compatible': modules['compatible'].include,
            'assistant' : modules['assistant'].include,
            'snail_sdk' : modules['snail-sdk'].include,
            'snail_core' : modules['snail-core'].include,
    ]
    outsiderDependencies = [
            'okhttp' : 'com.squareup.okhttp3:okhttp:3.12.13',
    ]
}
```
