---
layout: post
title:  Android中maven仓库的使用
date: 2017-04-06 17:44:00.000000000 +09:00
tags: Android
---

### 本地maven仓库的搭建
* [下载nexus](http://www.sonatype.org/nexus/)，解压
* 在bin目录下执行./nexus start或./nexus stop启动关闭服务
* 浏览器中访问 http://0.0.0.0:8081  ，默认用户名密码为admin/admin123

### 如何在项目中使用maven仓库
1. 在project目录下的`build.gradle`中配置maven仓库的地址，如

	```
	maven {
        url 'http://0.0.0.0:8081/#browse/browse/assets:maven-snapshots/'
    }
    maven {
        url 'http://0.0.0.0:8081/repository/maven-releases/'
    }
	```

2. 和使用开源库一样，在module的`build.gradle`中通过`compile('GROUP_ID:ARTIFACT_ID:VERSION')`的配置获取library

	```
	//使用SNAPSHOT仓库时，因为调试期间的版本号不变，会出现无法更新到最新aar的问题，解决方法在module的build.gradle中配置以下
	compile('GROUP_ID:ARTIFACT_ID:VERSION'){ changing = true }
	configurations.all {
   	    resolutionStrategy { cacheChangingModulesFor 0, 'seconds' }
   	}
	```

### 如何上传library至Amaven仓库
1. 编写上传library的脚本，为了使用方便，脚本写在单独的gradle文件中
	
	```
	apply plugin: 'maven'
	
	//release发布仓库地址，发布到此仓库的代码必需是稳定版，发布一次一个版本号
	def MAVEN_RELEASE_PATH = 'http://0.0.0.0:8081/repository/maven-releases/'
	
	//snapshots快照仓库地址，开发调试过程中发布的版本，一个版本号可以发布多次，版本号的格式必须是版本号＋ -SNAPSHOT
	def MAVEN_SNAP_SHOT_PATH = def MAVEN_SNAP_SHOT_PATH = 'http://0.0.0.0:8081/repository/maven-snapshots/'

	uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: MAVEN_RELEASE_PATH) {
                authentication(userName: 'admin', password: '***')
            }
            snapshotRepository(url: MAVEN_SNAP_SHOT_PATH) {
                authentication(userName: 'admin', password: '***')
            }
            pom.project {
                groupId GROUP_ID
                artifactId ARTIFACT_ID
                version VERSION_NAME
                packaging 'aar'
            }	
    	}
	}
	```
	
2. 在library module的`build.gradle`中配置`apply from: rootProject.file('gradle/uploadArchives.gradle')`，使用上传脚本。并且添加如下配置信息

	```
	ext{
    	ARTIFACT_ID = 'anjuke-library' //library的名称
    	VERSION_NAME = '1.0.0' //发布到release仓库的必需是稳定版，并且请更新版本号
    	//VERSION_NAME = '1.0.0-SNAPSHOT' //发布到snapshots仓库的版本号必需是-SNAPSHOT结尾    
    	GROUP_ID = 'com.jeney.android'
	}
	```
3. 在library module目录下直接执行命令`gradle uploadArchives`，或者AS界面点击library module的Tasks中的uploadArchives.执行成功，build成功aar包就可以上传到maven仓库了。

	![image](http://oo0obca4y.bkt.clouddn.com/uploadArchives.png)
	
> 	建议开发的library按照github上的library开源项目的结构一样，包涵library以及sample代码，方便调试与发布。