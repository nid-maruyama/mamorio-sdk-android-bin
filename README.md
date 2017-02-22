# MAMORIOSDK設定手順
### プロジェクトのbuild.gradleにMAMORIOリポジトリを追加する
```javascript
allprojects {
    repositories {
        maven {url "https://raw.githubusercontent.com/otoshimono/mamorio-sdk-android-bin/master/"}
    }
}
```

### アプリのbuild.gradleにMAMORIOリポジトリを追加する
```javascript
dependencies {
    compile 'jp.mamorio:mamorioSDK:1.0'
}
```
