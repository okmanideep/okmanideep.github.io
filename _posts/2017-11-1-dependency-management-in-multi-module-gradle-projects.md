---
layout: post
cover: false
title: Dependency management in multi-module gradle projects
description: 'Simple things you can do to improve your gradle/android dependency management'
date:   2017-11-02 00:20:00
tags: android gradle
class: 'post-template'
subclass: 'post tag-android tag-gradle'
categories: 'okmanideep'
navigation: True
disqus: 20171102002000
logo: 'logo_black.png'
---

These are the issues I will be addressing here:

+ Make it easy to read or update versions of the dependencies
+ Maintain consistency for dependency versions across all modules
+ Minimise the effort required to declare dependencies when extracting or adding a new module

## Step 1
Declare your dependencies and versions at the root of the project.

`$PROJECT_ROOT/dependencies.gradle`

```groovy
ext.versions = [:]

def versions = [:]

versions.support_lib = '26.0.2'
versions.gson = '2.8.0'

ext.versions = versions

ext.deps = [:]
def deps = [:]

def support_lib = [:]
support_lib.app_compat = "com.android.support:appcompat-v7:$versions.support_lib"
support_lib.recycler_view = "com.android.support:recyclerview-v7:$versions.support_lib"
deps.support_lib = support_lib

deps.gson = "com.google.code.gson:gson:$versions.gson"

ext.deps = deps
```

Attaching `versions` and `deps` to extra user properties `ext` will allow you to access those variables all across the project

## Step 2
Import the definitions in the root project's `buildScript` block

`$PROJECT_ROOT/build.gradle`

```groovy
buildScript {
  apply from: 'dependencies.gradle'

  //left out for brevity...
}
```

## Final step
Declare your module dependencies using the variables defined

`$PROJECT_ROOT/my-module/build.gradle`

```groovy
//left for brevity...
dependecies {
  implementation deps.support_lib.app_compat
  implementation deps.support_lib.recycler_view
  implementation deps.gson
}
```

