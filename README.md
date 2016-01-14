# Paperwork
[![Build Status](https://travis-ci.org/zsoltk/paperwork.svg?branch=master)](https://travis-ci.org/zsoltk/paperwork)

Generate build info for your Android project without breaking incremental compilation

### The problem
You want to include the git hash of the last commit and build time into your project, so that you can use their values in your crash reporting tool (for example).

The easiest way to do this is to generate them into your BuildConfig by adding these to your ```build.gradle```

```groovy
def gitSha = 'git rev-parse --short HEAD'.execute([], project.rootDir).text.trim()
def buildTime = new Date().format("yyyy-MM-dd'T'HH:mm:ss'Z'", TimeZone.getTimeZone("UTC"))

android {
    defaultConfig {
        buildConfigField "String", "GIT_SHA", "\"${gitSha}\""
        buildConfigField "String", "BUILD_TIME", "\"${buildTime}\""
    }
}
```

But this will break incremental builds, resulting in increased build times all the time.


### What this lib offers
Paperwork can generate this information (and more) for you and put it into a ```paperwork.json``` file inside your assets folder.
During runtime, you can access them lazy-loaded through getters like this:
```java
Paperwork paperwork = new Paperwork(context);
String gitSha = paperwork.get("gitSha");
String buildTime = paperwork.get("buildTime");
``` 

Many helpers are available to generate data for the most common scenarios. See the configuration below.  

Incremental builds are not broken, yay!

### Build time comparison
Measured three consecutive builds per type, running gradle daemon, hitting "Run 'app'" in Android Studio without touching anything else. Generated info: git hash and build time (using seconds) so that it has a new value every time.

* Without generating build info: *3.989s*, *3.915s*, *3.902s*
* Using BuildConfig fields: *14.843s*, *13.844s*, *13.194s*
* Using Paperwork: *4.356s*, *4.075s*, *4.042s*

### Download and setup
Add these dependencies to your ```build.gradle```:

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    
    dependencies {
        classpath 'hu.supercluster:paperwork-plugin:1.2.0'
    }
}

apply plugin: 'hu.supercluster.paperwork'

paperwork {
    // Configuration comes here, see next section for details
}
    
dependencies {
    compile 'hu.supercluster:paperwork:1.2.0'
}
```

Lastly, don't forget to add ```paperwork.json``` to your ```.gitignore``` file. 

### Configuration
Paperwork doesn't generate anything by default, you have to define whatever data you need in simple key-value pairings:

```groovy
paperwork {
    set: [
        someKey: "someValue", 
        foo:     "bar"
    ] 
}
```

All the data will be available at runtime by querying for your own defined keys:

```java
Paperwork paperwork = new Paperwork(context);
String data1 = paperwork.get("someKey");        // will return "someValue"
String data1 = paperwork.get("foo");            // will return "bar"
```

Paperwork comes with a handful of helper methods you can use to generate dynamic build-time data:
```groovy
paperwork {
    set: [
        // simple unix timestamp (ms)
        buildTime1: buildTime(),
        
        //  formatted date string
        buildTime2: buildTime("yyyy-MM-dd HH:mm:ss"),
        
        //  formatted date string for a given timezone
        buildTime3: buildTime("yyyy-MM-dd HH:mm:ss", "GMT"),
        
        // the current git SHA
        gitSha: gitSha(),
        
        // the last git tag (lightweight tags included)
        gitTag: gitTag(),
        
        // the last tag +
        // how many commits ahead of that tag are we now in the working tree +
        // current hash +
        // whether the working tree has uncommited changes
        // e.g. "v2.1.0-71-gb88c59a-dirty"
        gitInfo: gitInfo(),
        
        // runs a shell command and returns its output (doesn't have to be a script)
        shell: shell("scripts/test.sh"),
        
        // returns the value of an environment variable
        someEnv: env("USER")
    ] 
}
```

You can also change the default filename or generate the file somewhere else:

```groovy
paperwork {
    filename = 'src/main/assets/paperwork.json'
}
```

Note however, that in order for it to be available in Paperwork runtime,
it has to be in the assets folder, and if the filename is not
paperwork.json, you have to inject its name in the constructor:

```java
Paperwork paperwork = new Paperwork(context, "paperwork.json");
```


### Contributing

Contributions are welcome! Got a question, found a bug, have a new helper method idea? Submit an issue and discuss it!

I'd love to hear about your use case too, especially if it's not covered perfectly.


### License

    Copyright 2015 Zsolt Kocsi

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
