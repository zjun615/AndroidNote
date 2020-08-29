# AndroidNote
A note for Android


## 知识点
### 调节屏幕亮度
**调节系统屏幕亮度**

AndroidManifest.xml
```xml
<uses-permission android:name="android.permission.WRITE_SETTINGS" tools:ignore="ProtectedPermissions" />
```
```java
/**
 * 系统亮度是否为自动亮度
 */
private boolean isAutoBrightness() {
    try {
        return Settings.System.getInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS_MODE)
                == Settings.System.SCREEN_BRIGHTNESS_MODE_AUTOMATIC;
    } catch (Settings.SettingNotFoundException e) {
        e.printStackTrace();
    }
    return false;
}

/**
 * 打开、关闭系统亮度
 * @param isAuto true-enable; false-disable, normal
 */
private void setAutoBrightness(boolean isAuto){
    Settings.System.putInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS_MODE,
            isAuto ? Settings.System.SCREEN_BRIGHTNESS_MODE_AUTOMATIC : Settings.System.SCREEN_BRIGHTNESS_MODE_MANUAL);
}

/**
 * 获取系统屏幕亮度
 */
private int getSystemBrightness() {
    try {
        return Settings.System.getInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS);
    } catch (Settings.SettingNotFoundException e) {
        e.printStackTrace();
        return -1;
    }
}

private void changeSystemBrightness(@IntRange(from = 0, to = 255) int brightness) {
    //修改系统屏幕亮度需要修改系统设置的权限
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        if (!Settings.System.canWrite(this)) {
            Intent intent = new Intent(Settings.ACTION_MANAGE_WRITE_SETTINGS);
            startActivityForResult(intent, REQ_CODE_WRITE_SETTINGS);
        } else {
            setSystemBrightness(brightness);
        }
    } else {
        //Android6.0以下的系统则直接修改亮度
        setSystemBrightness(brightness);
    }
}

/**
 * 设置系统亮度
 */
@RequiresPermission(Manifest.permission.WRITE_SETTINGS)
private void setSystemBrightness(@IntRange(from = 0, to = 255) int brightness){
    brightness = Math.min(Math.max(brightness, 0), 255);
    try {
        Settings.System.getInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS);
    } catch (Settings.SettingNotFoundException e) {
        e.printStackTrace();
    }
    boolean isAuto = isAutoBrightness();
    logD("isAuto = " + isAuto);
    if (isAuto) {
        setAutoBrightness(false);
    }
    Uri uri = Settings.System.getUriFor(Settings.System.SCREEN_BRIGHTNESS);
    Settings.System.putInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS, brightness);
    // 通知系统我们已经修改了屏幕亮度
    getContentResolver().notifyChange(uri, null);
}
```
**调节当前页面亮度**
```java
/**
 * @return -255：未设置，跟系统亮度一样。0-255：亮度值
 */
private int getWindowBrightness() {
    Window window = getWindow();
    WindowManager.LayoutParams lp = window.getAttributes();
    return (int) (lp.screenBrightness * 255);
}

/**
 * 设置当前界面亮度
 * @param brightness -255：跟系统亮度一样。0-255：亮度值
 */
private void setWindowBrightness(int brightness) {
    Window window = getWindow();
    WindowManager.LayoutParams lp = window.getAttributes();
    lp.screenBrightness = brightness / 255.0f;
    window.setAttributes(lp);
}
```

### 显示gif图片
#### 使用fresco
build.gradle
```
implementation 'com.facebook.fresco:fresco:1.10.0'
implementation 'com.facebook.fresco:animated-gif:1.10.0'
```
xml
```xml
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/sdv_gif"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```
java
```java
// 网络图片
// Uri uri = Uri.parse("https://www.baidu.com/img/dong_e8b80aecc2ee2ab14545e57e1ee7642b.gif");

// 本地图片
Uri uri = new Uri.Builder()
        .scheme(UriUtil.LOCAL_RESOURCE_SCHEME)
        .path(String.valueOf(R.drawable.gif_wallpaper_5))
        .build();

DraweeController draweeController = Fresco.newDraweeControllerBuilder()
        .setUri(uri)
        // 设置加载图片完成后是否直接进行播放
        .setAutoPlayAnimations(true)
        .build();

sdv_gif.setController(draweeController);
```

## 编译错误
###  Validation failed, exiting
AndroidManifest.xml
```xml
<application
        android:appComponentFactory="fixError" // 添加，任意字符都行
        tools:replace="android:appComponentFactory" // 解决属性重复问题
             >
            
```

### Duplicate class android.support.v4.app.INotificationSideChannel found in modules classes.jar
androidx与support的冲突，必须改成统一

**1.改成androidx**

gradle.properties
```
android.useAndroidX=true
android.enableJetifier=true
```
工程 —— Refactor —— Migrate to AndroidX

**2.改成support**

全部改成support，并三方库降低到不含androidx的版本

## Kotlin
### 1. AS引入
工程build.gradle
```
buildscript {    
    ext.kotlin_version = '1.2.51'    
    dependencies {        
        ...        
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"    
    }
}
```

module的build.gradle
```
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
}
```
