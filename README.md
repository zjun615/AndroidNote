# AndroidNote
A note for Android


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
