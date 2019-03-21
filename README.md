AOP的build中的配置：
import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main


buildscript {

    repositories {
    
        mavenCentral()
        
    }
    dependencies {
    
        classpath 'org.aspectj:aspectjtools:1.8.9'
        
        classpath 'org.aspectj:aspectjweaver:1.8.9'
    }
}




apply plugin: 'com.android.application'

android {

    compileSdkVersion 26
    
    defaultConfig {
    
        applicationId "com.example.tong.intentapplication"
        
        minSdkVersion 15
        
        targetSdkVersion 26
        
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
final def log = project.logger

final def variants = project.android.applicationVariants


variants.all { variant ->
    if (!variant.buildType.isDebuggable()) {
        log.debug("Skipping non-debuggable build type '${variant.buildType.name}'.")
        return;
    }

    JavaCompile javaCompile = variant.javaCompile
    javaCompile.doLast {
        String[] args = ["-showWeaveInfo",
                         "-1.8",
                         "-inpath", javaCompile.destinationDir.toString(),
                         "-aspectpath", javaCompile.classpath.asPath,
                         "-d", javaCompile.destinationDir.toString(),
                         "-classpath", javaCompile.classpath.asPath,
                         "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]
        log.debug "ajc args: " + Arrays.toString(args)

        MessageHandler handler = new MessageHandler(true);
        new Main().run(args, handler);
        for (IMessage message : handler.getMessages(null, true)) {
            switch (message.getKind()) {
                case IMessage.ABORT:
                case IMessage.ERROR:
                case IMessage.FAIL:
                    log.error message.message, message.thrown
                    break;
                case IMessage.WARNING:
                    log.warn message.message, message.thrown
                    break;
                case IMessage.INFO:
                    log.info message.message, message.thrown
                    break;
                case IMessage.DEBUG:
                    log.debug message.message, message.thrown
                    break;
            }
        }
    }
}



dependencies {

    implementation fileTree(include: ['*.jar'], dir: 'libs')
    
    //noinspection GradleCompatible
    
    implementation 'com.android.support:appcompat-v7:26.1.0'
    
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    
    testImplementation 'junit:junit:4.12'
    
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
    

    compile 'org.aspectj:aspectjrt:1.8.1'
}
