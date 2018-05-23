---
title: Building and Deploying a Rust library on Android using JNA
layout: text
---

# Building and Deploying a Rust library on Android

In our previous post we built a Rust library, and linked to it on [Android using JNI](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-21-rust-on-android.html). This post builds on that by showing how to build the same app using JNA instead of JNI.

# Why JNA?

With JNI, for every `extern "C"` function that we created for our FFI, we had to create a mirror function in Rust for JNI to access. This involved some complicated function name construction and forced us to duplicate large chunks of code. JNA does away with all of that. You create a single interface that declares your FFI functions in Java and that's about it. It removes a large chunk of the headache and it is what we have moved to using here at Mozilla for our Android FFI.

# Getting set up.

Follow the instructions in [our previous post](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-21-rust-on-android.html) for building and creating your own standalone toolchains and linking them to Rustup. Once you have your environment set up, head back here for what to do next.

# Building our library

Now we're all set up and we're ready to start. Let's create the `lib` directory. If you've already created a Rust project from following either or our previous posts, you don't need to do it again.

```
cargo new cargo
mkdir android
```

`cargo new cargo` sets up a brand new Rust project with its default files and directories in a directory called `rust`. The name of the directory is not important. In this directory is a file called `Cargo.toml`, which is the package manager descriptor file, and there is be a subdirectory, `src`, which contains a file called `lib.rs`. This will contain the Rust code that we will be executing.

Our Rust project here is a super simple Hello World library. It contains a function `rust_greeting` that takes a string argument and return a greeting including that argument. Therefore, if the argument is "world", the returned string is "Hello world".

Open `cargo/src/lib.rs` and enter the following code.

```rust
use std::os::raw::{c_char};
use std::ffi::{CString, CStr};

#[no_mangle]
pub extern "C" fn rust_greeting(to: *const c_char) -> *mut c_char {
    let c_str = unsafe { CStr::from_ptr(to) };
    let recipient = match c_str.to_str() {
        Err(_) => "there",
        Ok(string) => string,
    };

    CString::new("Hello ".to_owned() + recipient).unwrap().into_raw()
}
```

Let's take a look at what is going on here.

As we will be calling this library from non-Rust code, we will actually be calling it through a C bridge. `#[no_mangle]` tells the compiler not to mangle the function name as it usually does by default, ensuring our function name is exported as if it had been written in C.

`extern` tells the Rust compiler that this function will be called from outside of Rust and to therefore ensure that it is compiled using C calling conventions.

The string that `rust_greeting` accepts is a pointer to a C char array. We have to then convert the string from a C string to a Rust `str`. First we create a `CStr` object from the pointer. We then convert it to a `str` and check the result. If an error has occurred, then no arg was provided and we substitute `there`, otherwise we use the value of the provided string. We then append the provided string on the end of our greeting string to create our return string. The return string is then converted into a `CString` and passed back into C code.

In order to build our library, we need to tell the compiler what type of a library it should produce. You can specify this in the `Cargo.toml` file's `[lib]` section:

```
[lib]
crate-type = ["dylib"]
```

We are now ready to build our libraries. Unlike with iOS, there is no handy universal Android library that we can make so we have to create one for each of our target architectures. We can then create symlinks to them from the Android project. You will need to use absolute paths to your libraries here, not relative ones, otherwise Android Studio will not be able to follow the link. Navigate to your `cargo` directory and run the following commands:

```
cargo build --target aarch64-linux-android --release
cargo build --target armv7-linux-androideabi --release
cargo build --target i686-linux-android --release
```

Now let's create our Android project.

Open Android Studio and select `Start a New Android Project` from the options.

![Create new Android project](assets/2017-09-21-rust-on-android/new-project.png)

On the next screen, type a project name of `Greetings` into the `Application name` field, choose your `Company domain` and select the `android` directory we created earlier as the `Project location`. This will create your Android project inside `greetings/android/Greetings`. Click `Next`.

![Name project](assets/2017-09-21-rust-on-android/name-project.png)

On the next screen, make sure the `Phone and Tablet` option is selected. Click `Next`.

![Choose form factor](assets/2017-09-21-rust-on-android/form-factors.png)

Now we will be asked to choose a starting activity. Select the `Empty Activity` option and click `Next`.

![Choose starting activity](assets/2017-09-21-rust-on-android/add-activity.png)

Name your Activity and layout on the following screen, calling the activity `GreetingsActivity` and the layout `activity_greetings`. Click `Finish`.

![Name activity](assets/2017-09-21-rust-on-android/customize-activity.png)

We now have a build Rust library and an Android project. We now need to link the two. First, we have to add JNA to our project. Open your applications `build.gradle` file and add the following to the `dependencies` block. When prompted, resync gradle.

```
implementation 'net.java.dev.jna:jna:4.5.0'
```
Now, we have to create directories in which to place our Rust library. When we built our library, we created a single `.so` file for each architecture we cross compiled for. We now need to create a subdirectory for each of these architectures inside a special directory called `jniLibs`. Android knows to look for them here. Once those directories are created, we can create a symlink to ensure that the Android Studio version of the libraries are always linking to the most recently built version.

```
cd ../android/Greetings/app/src/main
mkdir jniLibs
mkdir jniLibs/arm64
mkdir jniLibs/armeabi
mkdir jniLibs/x86

ln -s <project_path>/greetings/cargo/target/aarch64-linux-android/release/libgreetings.so jniLibs/arm64/libgreetings.so
ln -s <project_path>/greetings/cargo/target/armv7-linux-androideabi/release/libgreetings.so jniLibs/armeabi/libgreetings.so
ln -s <project_path>/greetings/cargo/target/i686-linux-android/release/libgreetings.so jniLibs/x86/libgreetings.so
```

After we've linked out Rust library, we also need to add the `libjnidispatch.so` library for each architecture in the same directories. Right now, I am not sure where we get this from but I have links to pre-built versions of them here:

-[arm64](https://github.com/mozilla/mentat/raw/master/sdks/android/Mentat/library/src/main/jniLibs/arm64/libjnidispatch.so)
-[armeabi](https://github.com/mozilla/mentat/raw/master/sdks/android/Mentat/library/src/main/jniLibs/armeabi/libjnidispatch.so)
-[x86](https://github.com/mozilla/mentat/raw/master/sdks/android/Mentat/library/src/main/jniLibs/x86/libjnidispatch.so):  

As with iOS, when we had to create a C header file to link our iOS code with our Rust library, when using JNA we need to create a Java Interface to do the same thing.

Under your `com.mozilla.greetings` in Android Studio, create a new Java Interface called JNA.java.

```
package com.mozilla.greetings;

import com.sun.jna.Library;
import com.sun.jna.Native;
import com.sun.jna.NativeLibrary;

public interface JNA extends Library {
    String JNA_LIBRARY_NAME = "greetings";
    NativeLibrary JNA_NATIVE_LIB = NativeLibrary.getInstance(JNA_LIBRARY_NAME);

    JNA INSTANCE = (JNA) Native.loadLibrary(JNA_LIBRARY_NAME, JNA.class);
}
```

Library is the JNA base interface that derives all native library definitions from the instance of your library. In is inside this class that we will declare the headers that will link to our Rust library. We have only one external function to declare here. The `rust_greeting` function.

Add the following interface declaration to your `JNA` interface.

```
String rust_greeting(String pattern);
```

That's it. We can now create some code that will call it.


As with iOS, we're going to create a wrapper class to wrap the JNA binding and ensure that we have a more ergonmic Java API. In the project explorer on the left hand side of the studio window, ensure that `app > java > <domain>.greetings` is highlighted then go to `File > New > Java Class`. Name your class `RustGreetings` and click `OK`.

![Create new class](assets/2017-09-21-rust-on-android/new-class.png)

In your new class file, add the following code. Here we are defining the native interface to our Rust library and calling it `greeting`, with the same signature. The `sayHello` method simply makes a call to that native function.

```java
public class RustGreetings {

    public String sayHello(String to) {
        return JNA.INSTANCE.rust_greeting(to);
    }
}
```

We need to load our Rust library when the app starts, so add the following lines below the class declaration and before the `onCreate` method in `GreetingsActivity.java`.

```java
   static {
        System.loadLibrary("greetings");
    }
```

This looks for a library called `greetings.so` inside the jniLibs directory and picks the correct one for the current architecture.

Open `res/layout/activity-greetings.xml`. In the `Component Tree` panel, highlight the `TextField` and open the `Properties` panel. Change the `ID` in the `Properties` panel to `greetingField`. This is how we are going to refer to it from our Activity.

![Amend activity xml](assets/2017-09-21-rust-on-android/activity-greetings.png)

Reopen `GreetingsActivity.java` and amend the `onCreate` method to call our greetings function and set the text on the `greetingField` `TextField` to the response value.

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_greetings);

        RustGreetings g = new RustGreetings();
        String r = g.sayHello("world");
        ((TextView)findViewById(R.id.greetingField)).setText(r);
    }
```

Build and run the app. If this is your first time in Android Studio, you may need to set up a simulator. When choosing/creating your simulator pick one with API 26. When the app starts, `Hello world` will be printed on your screen.

You can find the code for this on [Github](https://github.com/fluffyemily/cross-platform-rust).
