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

### 回到桌面
```java
Intent i = new Intent(Intent.ACTION_MAIN);
i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
i.addCategory(Intent.CATEGORY_HOME);
startActivity(i);
```

### 获取正在运行的进程
```java
// 只能获取自己应用
ActivityManager activityManager = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
List<ActivityManager.RunningAppProcessInfo> appProcesses = activityManager.getRunningAppProcesses();
if (appProcesses == null) {
    tv_msg.setText("NONE");
}
for (ActivityManager.RunningAppProcessInfo appProcess : appProcesses) {
    tv_msg.append("processName=" + appProcess.processName + "\t, importance=" + appProcess.importance + "\n");
}
```

```java
// 只能获取自己和桌面应用
ActivityManager am = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
List<ActivityManager.RunningTaskInfo> infoList = am.getRunningTasks(100);
for (ActivityManager.RunningTaskInfo info : infoList) {
    tv_msg.append("base=" + info.baseActivity.getPackageName() + "/" + info.baseActivity.getClassName()
            + "\ntop=" + info.topActivity.getPackageName() + "/" + info.topActivity.getClassName()
            + "\n\n");
}
```

```java
UsageStatsManager mUsageStatsManager = (UsageStatsManager) getSystemService(Context.USAGE_STATS_SERVICE);
long time = System.currentTimeMillis();
// We get usage stats for the last 10 seconds
List<UsageStats> stats = mUsageStatsManager.queryUsageStats(UsageStatsManager.INTERVAL_DAILY, time - 1000 * 10, time);

// Sort the stats by the last time used
if (stats != null) {
    SortedMap< Long, UsageStats > mySortedMap = new TreeMap< Long, UsageStats >();
    int index = 0;
    for (UsageStats usageStats: stats) {
        mySortedMap.put(usageStats.getLastTimeUsed(), usageStats);
    }
    for (UsageStats item : mySortedMap.values()) {
        tv_msg.append(index++ + ".packageName=" + item.getPackageName()
                + ", getLastTimeUsed=" + formatTime(item.getLastTimeUsed())
                + ", getFirstTimeStamp=" + formatTime(item.getFirstTimeStamp())
                + ", getLastTimeVisible=" + formatTime(item.getLastTimeVisible())
                + ", getTotalTimeInForeground=" + item.getTotalTimeInForeground()
                + ", getTotalTimeVisible=" + item.getTotalTimeVisible()
                + "\n\n");
    }
}
```

```java
UsageStatsManager mUsageStatsManager = (UsageStatsManager) getSystemService(Context.USAGE_STATS_SERVICE);
long time = System.currentTimeMillis();
// We get usage stats for the last 10 seconds
UsageEvents usageEvents = mUsageStatsManager.queryEvents(time - 500_000, time);
index = 0;
while (usageEvents.hasNextEvent()) {
    UsageEvents.Event event = new UsageEvents.Event();
    usageEvents.getNextEvent(event);
    Log.d("TestActivity", index++ + ".packageName=" + event.getPackageName()
            + ", getTimeStamp=" + formatTime(event.getTimeStamp())
            /*
             1-MOVE_TO_FOREGROUND
             2-MOVE_TO_BACKGROUND
             23-ACTIVITY_STOPPED
             */
            + ", getEventType=" + event.getEventType()
            + ", getShortcutId=" + event.getShortcutId()
            + ", getAppStandbyBucket=" + event.getAppStandbyBucket()
            + ", getConfiguration=" + event.getConfiguration()
            + "\n\n");
}
```

```java
PackageManager localPackageManager = getPackageManager();
List localList = localPackageManager.getInstalledPackages(0);
for (int i = 0; i < localList.size(); i++) {
    PackageInfo localPackageInfo1 = (PackageInfo) localList.get(i);
    String str1 = localPackageInfo1.packageName.split(":")[0];
    if (((ApplicationInfo.FLAG_SYSTEM & localPackageInfo1.applicationInfo.flags) == 0)
            && ((ApplicationInfo.FLAG_UPDATED_SYSTEM_APP & localPackageInfo1.applicationInfo.flags) == 0)
            && ((ApplicationInfo.FLAG_STOPPED & localPackageInfo1.applicationInfo.flags) == 0)) {
        Log.e("TestActivity",str1);
    }
}
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

## EditText
### 光标自定义
修改颜色、宽度

通过padding（一般为负值），top和bottom可修改高度，left还可以修改光标与文字的间距。
shape_input_cursor.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="#FF00C853" />
    <size android:width="2dp" />
    <padding
        android:top="-5dp"
        android:bottom="-5dp" 
        android:left="-5dp"/>
</shape>
```
```xml
<EditText
    android:textCursorDrawable="@drawable/shape_input_cursor"
    ... />
```
