# Demo SE2 android repository with Github Actions CI and Sonarcloud for analysis

[![SonarCloud](https://sonarcloud.io/images/project_badges/sonarcloud-white.svg)](https://sonarcloud.io/summary/new_code?id=SE2-sample-tut-project_demo-repo)
<br>
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=SE2-sample-tut-project_demo-repo&metric=coverage)](https://sonarcloud.io/summary/new_code?id=SE2-sample-tut-project_demo-repo)
### 

## How to get jacoco plugin and sonar scanner to work within modules:

1. you can use the plain 'jacoco' plugin for this to work
2. move the <b>sonarqube</b> plugin and its properties to your <b>build.gradle in a module (e.g. app/build.gradle)</b>
3. add a <b>jacocoTestReport</b> task that depends on <b>testDebugUnitTest</b> (you can see the template below)
4. add <b>testOptions</b> to the android configuration in the build.gradle and finalize the <b>jacocoTestReport</b> task (see template below)
5. make sure that the <b>xml.destination</b> from the reports attribute of the jacocoTestReport task <b>is the same</b> as the <b>sonar.coverage.jacoco.xmlReportPaths</b> property of the sonaqube task
6. with that configuration, it is sufficient to run the following command as the last part in your CI pipeline file: <b>run: ./gradlew build sonarqube --info</b>

```gradle
plugins {
    id 'com.android.application'
    
    ////////// 1. simple jacoco plugin //////////
    id 'jacoco'  
    
    ////////// 2. move sonarqube plugin into app/build.gradle //////////
    id 'org.sonarqube' version '3.3'   
    
}

android {
    compileSdk 32

    defaultConfig {
        applicationId "com.example.demo_repo"
        minSdk 30
        targetSdk 32
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    ////////// 4. add testOptions //////////
    testOptions {        
        unitTests.all {
            useJUnitPlatform()
            finalizedBy jacocoTestReport
        }
    }
}

////////// 3. add jacocoTestReport task like so //////////
task jacocoTestReport(type: JacocoReport, dependsOn: 'testDebugUnitTest') {   

    reports {
        xml.enabled true
        
        //////////  5. xml.destination must be same as sonaqube property  //////////
        xml.destination file("${project.projectDir}/build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml")  
    }

    def fileFilter = ['**/R.class', '**/R$*.class', '**/BuildConfig.*', '**/Manifest*.*', '**/*Test*.*', 'android/**/*.*']
    def debugTree = fileTree(dir: "${buildDir}/intermediates/javac/debug", excludes: fileFilter)
    def mainSrc = "${project.projectDir}/src/main/java"

    sourceDirectories.from = files([mainSrc])
    classDirectories.from = files([debugTree])
    executionData.from = files("${buildDir}/jacoco/testDebugUnitTest.exec")
}

////////// 2. move also sonarqube properties //////////
sonarqube {      
    properties {
        property "sonar.projectKey", "SE2-sample-tut-project_demo-repo"
        property "sonar.organization", "se2-sample-tut-project"
        property "sonar.host.url", "https://sonarcloud.io"
        property "sonar.java.coveragePlugin", "jacoco"
        
        ////////// 5. property same as xml.destination of jacocoTestReport //////////
        property "sonar.coverage.jacoco.xmlReportPaths", "${project.projectDir}/build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml"   
    }
}

dependencies {

    implementation 'androidx.appcompat:appcompat:1.3.0'
    implementation 'com.google.android.material:material:1.4.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'

    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'

    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
}
```
