---
layout: post
title: Split Packages on Java 9 Modules
tags: java-9 modules jpms
github_comments_issueid: 3
repository: Split-Package
excerpt: In this post I am going to describe the problem of split package on Java 9 Module system and propose solutions to 3 possible cases (Move, Upgrade and Unsplit).
---

Java 9 finally introduced a proper module system, [The Java Platform Module System (JPMS)](http://openjdk.java.net/projects/jigsaw/spec), formerly known as Project Jigsaw.  
One of its main goals is to _"make it easier for developers to construct and maintain libraries and large applications"_,
and it suppose to solve some of the issues with conflicting libraries and versions, a.k.a. [Jar Hell](https://dzone.com/articles/what-is-jar-hell).  
You can read [here](http://blog.joda.org/2017/04/java-9-modules-jpms-basics.html) about JPMS basics and [here](https://guides.gradle.org/building-java-9-modules) about the way to manage them with gradle.

One of the restrictions when working with JPMS is that a package can not be exported from more one module ([video](https://www.youtube.com/watch?v=gtcTftvj0d0&feature=youtu.be&t=16m26s)).  
For example, if one module (dependency.api) contains the class _io.github.msayag.lib.Something_, another module (dependency.impl)can contain the class _io.github.msayag.lib.KindOfSomething_ because they are both in the package _io.github.msayag.lib_.  

![Split Package](https://raw.githubusercontent.com/msayag/msayag.github.io/master/_posts/2017-12-3-SplitPackage/split_package_1.png)

![IntelliJ Error](https://raw.githubusercontent.com/msayag/msayag.github.io/master/_posts/2017-12-3-SplitPackage/split_package_2.png) 

## Solution 1: Move
The problematic modules are under your control
The solution is this case is usually simple: change the package of the conflicting classes.
For example, move _KindOfSomething_ to package _io.github.msayag.lib.impl_

## Solution 2: Upgrade
A package might be split between 2 libraries that are out of your control.  
For example, I have a project that uses log4j and thus require the libraries _log4j.api_ and _log4j.core.  
Unfortunately, the used version is 2.7, which is old and doesn't support module system.  

![Log4J 2.7 Conflict](https://raw.githubusercontent.com/msayag/msayag.github.io/master/_posts/2017-12-3-SplitPackage/log4j-2.7_1.png)

![Log4J 2.7 Split Package](https://raw.githubusercontent.com/msayag/msayag.github.io/master/_posts/2017-12-3-SplitPackage/log4j-2.7_2.png)

One solution may be to upgrade the library to a newer version that do support the modules (log4j 2.10.0).

![Log4J 2.10.0](https://raw.githubusercontent.com/msayag/msayag.github.io/master/_posts/2017-12-3-SplitPackage/log4j-2.10.0.png)

## Solution 3: Unsplit
If such version can not be used, because of the project's limitation or if such a version does not exist, a more complicated solution might have to be applied.
Since the problem is with classes from different jars, perhaps it can be worked around by merging the conflicting jars into a bigger one that contains the content of both.
To do so, extract the 2 original jars to the same location and pack them back together as a new jar or module.

```
mkdir log4j-bridge
cd log4j-bridge
jar -xvf /path/to/log4j-api-2.7.jar
jar -xvf /path/to/log4j-core-2.7.jar
jar -c -f log4j-bridge-2.7.jar *
```
