# How to make Android apps without IDE from command line
A HelloWorld without Android Studio
Update: I’ve made a new course that explain how you can avoid Android Studio and Gradle, but still use IntelliJ iDE:
How to do Android development faster without Gradle
IntelliJ IDE, but not Gradle
[![Alt text](https://github.com/anonymansz/sdk-tools-linux/blob/main/src/1_RDLCtAqz0GWpMyrw3eSHBw.png)
In this tutorial, I will show you how you can build/compile an APK (an Android app) from your java code using terminal (on Linux) without IDE or in other words without Android Studio. At the end, I will also show you a script to automate the process. In this example, I will use Android API 19 (4.4 Kitkat) to make a simple HelloWorld. I want to say that I will do this tutorial without android command which is deprecated.

## Install Java
First, you need to install java, in my case, I install the headless version because I don’t use graphics (only command line):
~~~
sudo apt-get install openjdk-8-jdk-headless
~~~
## Install all SDK tools
Then download the last SDK tools of Android which you can find here:
Download Android Studio and SDK Tools | Android Studio
Download the official Android IDE and developer tools to build apps for Android phones, tablets, wearables, TVs, and…
developer.android.com

Or simply do:
~~~
wget https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip
~~~
I recommend to unzip it in the /opt directory inside another directory that we will call “android-sdk”:
~~~
mkdir -p /opt/android-sdk
unzip sdk-tools-linux-3859397.zip -d /opt/android-sdk
~~~
Now, we have to install platform tools (which contain ADB), an Android API and build tools.
In fact, if you are on Debian, you can avoid installing platform-tools package and only install ADB like that:
~~~
sudo apt-get install adb
~~~
## Code the application
In this example, I want to compile a simple HelloWorld. So, first, we need to make a project directory:
~~~
mkdir HelloAndroid
cd HelloAndroid
~~~
Then we have to make the files tree:
~~~
mkdir -p src/com/example/helloandroid
mkdir obj
mkdir bin
mkdir -p res/layout
mkdir res/values
mkdir res/drawable
~~~
If you use exernal libraries (.jar files), also make a folder for them:
~~~
mkdir libs
~~~

## You have an example here:

Make the file 'src/com/example/helloandroid/MainActivity.java' and put that inside
Make the 'strings.xml' file in the 'res/values folder'. It contains all the text that your application uses:
The 'activity_main.xml' is a layout file which have to be in 'res/layout'
You also have to add the file 'AndroidManifest.xml' at the 'root'

## Build the code
Now, I recommend to store the project path in a variable:
~~~
export PROJ=path/to/HelloAndroid
~~~
First, we need generate the R.java file which is necessary for our code:
~~~
cd /opt/android-sdk/build-tools/26.0.1/
./aapt package -f -m -J $PROJ/src -M $PROJ/AndroidManifest.xml -S $PROJ/res -I /opt/android-sdk/platforms/android-19/android.jar
~~~
-m instructs aapt to create directories under the location specified by -J
-J specifies where the output goes. Saying -J src will create a file like src/com/example/helloandroid/R.java
-S specifies where is the res directory with the drawables, layouts, etc.
-I tells aapt where the android.jar is. You can find yours in a location like android-sdk/platforms/android-<API level>/android.jar

Now, we have to compile the .java files:
~~~
cd /path/to/AndroidHello
javac -d obj -classpath src -bootclasspath /opt/android-sdk/platforms/android-19/android.jar src/com/example/helloandroid/*.java
If you have use an external, add it the classpath:
javac -d obj -classpath "src:libs/<your-lib>.jar" -bootclasspath /opt/android-sdk/platforms/android-19/android.jar src/com/example/helloandroid/*.java
~~~
The compiled .class files are in obj folder, but Android can’t read them. We have to translate them in a file called “classes.dex” which will be read by the dalvik Android runtime:
~~~
cd /opt/android-sdk/build-tools/26.0.1/
./dx --dex --output=$PROJ/bin/classes.dex $PROJ/obj
~~~
But if you use external libraries, do rather:
~~~
./dx --dex --output=$PROJ/bin/classes.dex $PROJ/*.jar $PROJ/obj
~~~
If you have the error UNEXPECTED TOP-LEVEL EXCEPTION, it can be because you use old build tools and DX try to translate java 1.7 rather than 1.8. To solve the problem, you have to specify 1.7 java version in the previous javac command:
~~~
cd /path/to/AndroidHello
javac -d obj -source 1.7 -target 1.7 -classpath src -bootclasspath /opt/android-sdk/platforms/android-19/android.jar src/com/example/helloandroid/*.java
~~~
The -source option specify the java version of your source files. Note that we can use previous versions of Java even we use OpenJDK 8 (or 1.8).
We can now put everything in an APK:
~~~
./aapt package -f -m -F $PROJ/bin/hello.unaligned.apk -M $PROJ/AndroidManifest.xml -S $PROJ/res -I /opt/android-sdk/platforms/android-19/android.jar
cp $PROJ/bin/classes.dex .
./aapt add $PROJ/bin/hello.unaligned.apk classes.dex
~~~
Be aware: until now, we used three AAPT commands, the first and the second one are similar but they don’t do the same. You have to copy the classes.dex file at the root of project like above! Otherwise, AAPT won’t put this file at right place in the APK archive (because an APK is like a .zip file).
The generated package can’t be installed by Android because it’s unaligned and unsigned.
If you want, you can check the content of the package like this:
~~~
./aapt list $PROJ/bin/hello.unaligned.apk
~~~
## Sign the package
To do so, we firstly create a new keystore with the command keytool given by Java:
keytool -genkeypair -validity 365 -keystore mykey.keystore -keyalg RSA -keysize 2048
Just answer the questions and put a password.
You can sign an APK like this:
./apksigner sign --ks mykey.keystore $PROJ/bin/hello.apk
Note that apksigner only exist since Build Tools 24.0.3.
## Align the package
It’s as simple as that:
./zipalign -f 4 $PROJ/bin/hello.unaligned.apk $PROJ/bin/hello.apk
Alignment increase the performance of the application and may reduce memory use.
## Test the application
To test the application, connect your smartphone with a USB cable and use ADB:
~~~
adb install $PROJ/bin/hello.apk
adb shell am start -n com.example.helloandroid/.MainActivity
~~~
But before run this command, I recommend to run this one:
~~~
adb logcat
~~~
If there is an error during installation or running, you see it with that command.
Voila! Here’s the result:

## Make a script
If you don’t want to run all these steps every time you would like to compile your app, make a script! Here’s mine:

To use it, run:
~~~
cd /path/to/AndroidHello
./build.sh test
~~~
[![Alt text](https://github.com/anonymansz/sdk-tools-linux/blob/main/src/1_BkYm27ojlbrl4wdxQl6E5Q.png)
## Notes
You can remove “test” if you just want to compile without testing.
[This script only compile and run the app on the phone.](https://authmane512.medium.com/how-to-build-an-apk-from-command-line-without-ide-7260e1e22676)
