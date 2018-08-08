# HotUpdateDemo
codepush demo
###前期环境配置
####1. 安装appcenter
```js
npm install -g appcenter-cli 
appcenter login  
```
####2. 创建app
```js
//groupName 可选，默认创建为账号拥有
appcenter apps create -d <appDisplayName> -o <operatingSystem>  -p <platform>
//举个栗子
appcenter apps create -d MyAndroid -o Android  -p React-Native

```
####3. 创建部署 生成秘钥
```js
appcenter codepush deployment add -a <ownerName>/<appName> Staging
appcenter codepush deployment add -a <ownerName>/<appName> Production
//举个栗子  
appcenter codepush deployment add -a MyGroup/MyAndroid Staging  // 这里的ownerName是你自己的账号名
```
####4. 查看部署
```js
appcenter codepush deployment list -a MyGroup/MyAndroid
```
###5.appcenter 其他的手动配置（自主开启）
```js
npm install appcenter appcenter-analytics appcenter-crashes --save
react-native link
// 可能需要配置git socket 代理
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
// 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
//查看配置
git config --list
```
###代码配置
####1.安装依赖
```js
npm install --save react-native-code-push  
react-native link react-native-code-push
```
###打包部署
需要注意  这里的版本号一般不变 要是变了android配置的版本号也要变
```js
// 查看部署情况
code-push deployment ls MyAndroid -k 
// 部署debug
appcenter codepush release-react -a zzyamor/MyAndroid -t 1.0.1 -m --description "更改版本号"
// 部署线上？realse
appcenter codepush release-react -a zzyamor/MyAndroid -d Production -t 1.0.1 -m --description "线上版本"
```
###android环境配置
- 配置`MainApplication.java`
```javascript 1.8
@Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
            new CodePush(BuildConfig.CODEPUSH_KEY, getApplicationContext(), BuildConfig.DEBUG),
            new AppCenterReactNativeCrashesPackage(MainApplication.this, getResources().getString(R.string.appCenterCrashes_whenToSendCrashes)),
            new AppCenterReactNativeAnalyticsPackage(MainApplication.this, getResources().getString(R.string.appCenterAnalytics_whenToEnableAnalytics)),
            new AppCenterReactNativePackage(MainApplication.this)
      );
    }
```
- 配置版本号`build.gradle(app)`
```javascript 1.8
// 第一处
defaultConfig {
        applicationId "com.hotupdatedemo"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
        versionName "1.0.1"
        ndk {
            abiFilters "armeabi-v7a", "x86"
        }
    }
// 第二处
...
splits{}
buildTypes {
        release {
            minifyEnabled enableProguardInReleaseBuilds
            proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
            buildConfigField "String", "CODEPUSH_KEY", '"'+properties.getProperty("code_push_key_production")+'"'
            signingConfig signingConfigs.release
        }
        debug {
            buildConfigField "String", "CODEPUSH_KEY", '"'+properties.getProperty("code_push_key_staging")+'"'
        }
    }
...
// 第三处  打包用的
...
defaultConfig{}
signingConfigs {
        release {
            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                storeFile file(MYAPP_RELEASE_STORE_FILE)
                storePassword MYAPP_RELEASE_STORE_PASSWORD
                keyAlias MYAPP_RELEASE_KEY_ALIAS
                keyPassword MYAPP_RELEASE_KEY_PASSWORD
            }
        }
    }
...
```
- 配置`local.properties`
```javascript 1.8
code_push_key_production=FSmzKrz9zp3vto458gGeFicsX4UqH1j_Ch8Hm
code_push_key_staging=i7kftMkk0585P0W4IvW3g_lEfZcZByYPR3Urm
```
###[最简单打包成APK](https://reactnative.cn/docs/signed-apk-android.html#content)
```javascript 1.8
adb install xxxx.apk
```

###最简单自动更新
```jsx harmony
import codePush from "react-native-code-push";
let codePushOptions = { checkFrequency: codePush.CheckFrequency.MANUAL };

class MyApp extends Component {
    onButtonPress() {
        codePush.sync({
            // false 不弹窗  自动更新
            updateDialog: true,
            installMode: codePush.InstallMode.IMMEDIATE
        });
    }

    render() {
        return (
            <View>
                <TouchableOpacity onPress={this.onButtonPress}>
                    <Text>Check for updates</Text>
                </TouchableOpacity>
            </View> 
        )
    }
}

MyApp = codePush(codePushOptions)(MyApp);
export default MyApp;
```
###bundle
```js
react-native bundle --platform android --entry-file index.js --bundle-output ./index.android.bundle --dev false
```
###使用APP Center打包build
```angular2html
// 需要更改.gitignore配置文件
// 复制release-key到android/app目录
```
![.gitignore](./gitignore.jpg)

###参考
- [配置APPCenter](https://appcenter.ms)
- [React Native 热更新Appcenter codepush,Android](https://www.jianshu.com/p/a09005ddf509)
- [React Native热更新部署/热更新-CodePush最新集成总结(新)](https://www.jianshu.com/p/9e3b4a133bcc)
- [Management CLI](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/cli#releasing-updates-react-native)



