# 主服务sdk仓库依赖配置说明

------
现在我们在jFrog仓库里面添加了主服务的SDK配置，以后对应分支的应用就可以像依赖一个标准的远程jar包一样引用主服务SDK了
如：   
compile(group: 'com.ubtechnic.cruzr', name: 'cruzr-sdk', version: '0.1.12')
## 开始配置
第一步
在pc端 C:\Users\xxx\\.gradle\gradle.properties 文件里面添加内容

```groovy
# Artifactory base info
artifactory_url=http://10.10.22.41:18083/artifactory
artifactory_username=填上开通账户名
artifactory_password=填上密码

# Artifactory cruzr team repositories
artifactory_cruzr_release=cruzr-release-local
artifactory_cruzr_snapshot=cruzr-snapshot-local

artifactory_virtual_release=libs-release
artifactory_virtual_snapshot=libs-snapshot
```
第二步
:bowtie:
:kissing_closed_eyes:
```groovy

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        //此行不用修改，保留项目原来的配置即可
        classpath 'com.android.tools.build:gradle:2.2.3'
        //添加插件，如果只是依赖jar包，不需要发布jar，就不用配置下面这一句
        classpath 'org.jfrog.buildinfo:build-info-extractor-gradle:4.6.2'
    }
}
//添加远程依赖配置
allprojects {
    repositories {
        // 产品团队的 release 仓库
        maven {
            url "${artifactory_url}/${artifactory_cruzr_release}"
            credentials {
                username = artifactory_username
                password = artifactory_password
            }
        }

        // 产品团队的 snapshot 仓库
        maven {
            url "${artifactory_url}/${artifactory_cruzr_snapshot}"
            credentials {
                username = artifactory_username
                password = artifactory_password
            }
        }

        // release 虚拟仓库
        maven {
            url "${artifactory_url}/${artifactory_virtual_release}"
            credentials {
                username = artifactory_username
                password = artifactory_password
            }
        }

        // snapshot 虚拟仓库（如果没有依赖外部仓库的 snapshot 库，则不用配置）
        maven {
            url "${artifactory_url}/${artifactory_virtual_snapshot}"
            credentials {
                username = artifactory_username
                password = artifactory_password
            }
        }
        // 外部仓库
        jcenter()
    }
}
```
## 在项目中依赖
### 正式(release)发布

```groovy
    compile(group: 'com.ubtechnic.cruzr', name: 'cruzr-sdk', version: '0.1.12')
```
### snapshot发布

为了在sdk调试过程中可以频繁发布，而避免sdk的版本号迅速上涨，所以这种情况请大家配置为snapshot形式的依赖，等sdk调试稳定之后再切回上面的正式发布依赖
```groovy
configurations.all {
    //保证每次刷新会依赖最新的sdk
    resolutionStrategy.cacheChangingModulesFor 30, 'seconds'
}
dependencies {
 ...
 compile(group: 'com.ubtechnic.cruzr', name: 'cruzr-sdk', version: '0.1.9-SNAPSHOT', changing: true)
 ...
 }
```
当然为了方便也可以使用变量控制
sdk version 0.1.4 及其以后的sdk版本
在项目模块下面的build.gradle里面添加
示例 apirunner里面的build.gradle 配置如下，请参考标注部分的更新即可
# 完整示例
```grovvy
apply plugin: 'com.android.application'
//★★★★★★★★★★★★★★★★★注意★★★★★★★★★★★★★★
def isSnapShot = true
def coreServiceSDKVersion = "0.1.4"
//★★★★★★★★★★★★★★★★★注意★★★★★★★★★★★★★★
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.1"

    defaultConfig {
        applicationId "com.ubtechinc.cruzr.apirunner"
        minSdkVersion 19
        targetSdkVersion 25
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

//★★★★★★★★★★★★★★★★★注意★★★★★★★★★★★★★★
if (isSnapShot) {
    configurations.all {
        resolutionStrategy.cacheChangingModulesFor 30, 'seconds'
    }
}
:smiley:
//★★★★★★★★★★★★★★★★★注意★★★★★★★★★★★★★★
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.0.0'
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:recyclerview-v7:25.0.0'
    compile 'io.reactivex:rxjava:1.2.10'
    compile 'io.reactivex:rxandroid:1.2.1'
    compile files('libs/rockvideo-1.2.jar')
    compile 'com.jakewharton.rxbinding:rxbinding:1.0.1'
//★★★★★★★★★★★★★★★★★注意★★★★★★★★★★★★★★
    if (isSnapShot) {
        compile(group: 'com.ubtechnic.cruzr', name: 'cruzr-sdk', version: "${coreServiceSDKVersion}-SNAPSHOT", changing: true)
    } else {
        compile(group: 'com.ubtechnic.cruzr', name: 'cruzr-sdk', version: "${coreServiceSDKVersion}")
//★★★★★★★★★★★★★★★★★注意★★★★★★★★★★★★★★
    }
}
```
