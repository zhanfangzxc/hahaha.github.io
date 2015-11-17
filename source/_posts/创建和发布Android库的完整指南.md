title: 创建和发布Android库的完整指南
date: 2015-11-05 11:18:33
tags:
- Android

---

原文地址:[https://medium.com/android-news/the-complete-guide-to-creating-an-android-library-46628b7fc879#.fpcy9pvc7](https://medium.com/android-news/the-complete-guide-to-creating-an-android-library-46628b7fc879#.fpcy9pvc7)

我一直很惊讶于那些Android开发社区中存在的许多又去的第三方开源库没很长时间依赖，我都希望贡献一些自己的开源库，但是我并不知道要怎么做。查询了很多关于如何发布Android开源库的文章之后，我仍然发现在很多不同地方查询到的信息中仍然有许多遗失的细节，所以我今天说明一下我自己的实现过程。

首先，我使用的是以Gradle为官方构建系统的Android Studio，来创建我左右的Android项目，下载[Android Studio](https://developer.android.com/intl/zh-cn/sdk/index.html)的最新版本。

### 术语

在我们开始讲解之前先来了解一些术语：

**Project**——Android Studio中创建的项目代表一个完整的Android App,Android Studio中的项目包含一个或多个模块，就像Eclipse中工作空间的概念。

**Module**——一个模块是一个应用中的一个组建，可以独立的构建，测试或者调试。模块中包括你的应用相关的源代码和资源文件，Android Studio中的一个就像Eclipse中的一个工程。

**AAR**——是一个Android库的二进制版本，一个库项目的主要输出就是一个.aar的包，他是一个编译代码(就像一个jar文件或者是.so文件)和资源文件(manifest,res,assets)的组合.

**Maven中央仓库**——一个由Maven组织提供的仓库，它包括大量常用的开源库，搜索Maven可以浏览Maven中央仓库中的内容，[Gradle,Please]()是另外一个允许你搜索Maven仓库的工具，Gradle使用的是jCenter协议，如果你再你的repositories节点([Notes about jCenter]())写包含jCenter()的话,Maven中央仓库通常被称为Maven中央或中央仓库。

**Sonatype**——Snoatype's开源软件存储库主机主要是用来为项目所有者和贡献者提供发布他们的组件到中央仓库的服务。

**GPG**——GNU隐私保护(GPG,也GnuPG),GNU项目的自由选择PGP加密软件符合OpenPGP(RFC4880)标准。使用GPG可以加密(解密)文件包含敏感数据,如电子受保护的健康信息(ePHI)由健康保险流通与责任法案(HIPAA)隐私和安全规则。更多GPG,请参阅GNU隐私保护的网站。(此段话全部使用翻译软件翻译)

### 准备你的Android库

我将会使用我的[Tresle]()库作为一个示例，这里需要做一些修改才能发不到中央仓库

- 将库的核心代码和库的使用示例的代码分开，我将这个库分成两个模块，一个是**library**，一个是**sample**。

- 在Sample模块的**build.gradle**中，一定要包含以下内容

		apply plugin: 'com.android.application'

		dependencies {
			compile project(':library')
		}
		
- 在library模块的**build.gradle**文件中，要包含下面的内容

		apply plugin: 'com.android.library'
		apply from: 'maven-push.gradle'
		
- 在library模块，添加一个**gradle.properties**文件，然后包含如下代码

		POM_NAME=ProjectName
		POM_ARTOFACT_ID=projectname
		POM_PACKAGING=aar
		
- 在library模块中，添加一个**maven-push.gradle**文件，然后添加如下代码

		/*
		 * Copyright 2013 Chris Banes
		 *
		 * Licensed under the Apache License, Version 2.0 (the "License");
		 * you may not use this file except in compliance with the License.
		 * You may obtain a copy of the License at
		 *
		 *     http://www.apache.org/licenses/LICENSE-2.0
		 *
		 * Unless required by applicable law or agreed to in writing, software
		 * distributed under the License is distributed on an "AS IS" BASIS,
		 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
		 * See the License for the specific language governing permissions and
		 * limitations under the License.
		 */

		apply plugin: 'maven'
		apply plugin: 'signing'

		def isReleaseBuild() {
		    return VERSION_NAME.contains("SNAPSHOT") == false
		}

		def getReleaseRepositoryUrl() {
		    return hasProperty('RELEASE_REPOSITORY_URL') ? RELEASE_REPOSITORY_URL
		            : "https://oss.sonatype.org/service/local/staging/deploy/maven2/"
		}

		def getSnapshotRepositoryUrl() {
		    return hasProperty('SNAPSHOT_REPOSITORY_URL') ? SNAPSHOT_REPOSITORY_URL
		            : "https://oss.sonatype.org/content/repositories/snapshots/"
		}

		def getRepositoryUsername() {
		    return hasProperty('NEXUS_USERNAME') ? NEXUS_USERNAME : ""
		}

		def getRepositoryPassword() {
		    return hasProperty('NEXUS_PASSWORD') ? NEXUS_PASSWORD : ""
		}

		afterEvaluate { project ->
		    uploadArchives {
		        repositories {
		            mavenDeployer {
		                beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }

		                pom.groupId = GROUP
		                pom.artifactId = POM_ARTIFACT_ID
		                pom.version = VERSION_NAME

		                repository(url: getReleaseRepositoryUrl()) {
		                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
		                }
		                snapshotRepository(url: getSnapshotRepositoryUrl()) {
		                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
		                }

		                pom.project {
		                    name POM_NAME
		                    packaging POM_PACKAGING
		                    description POM_DESCRIPTION
		                    url POM_URL

		                    scm {
		                        url POM_SCM_URL
		                        connection POM_SCM_CONNECTION
		                        developerConnection POM_SCM_DEV_CONNECTION
		                    }

		                    licenses {
		                        license {
		                            name POM_LICENCE_NAME
		                            url POM_LICENCE_URL
		                            distribution POM_LICENCE_DIST
		                        }
		                    }

		                    developers {
		                        developer {
		                            id POM_DEVELOPER_ID
		                            name POM_DEVELOPER_NAME
		                        }
		                    }
		                }
		            }
		        }
		    }

		    signing {
		        required { isReleaseBuild() && gradle.taskGraph.hasTask("uploadArchives") }
		        sign configurations.archives
		    }

		    //task androidJavadocs(type: Javadoc) {
		    //source = android.sourceSets.main.allJava
		    //}

		    //task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
		    //classifier = 'javadoc'
		    //from androidJavadocs.destinationDir
		    //}

		    task androidSourcesJar(type: Jar) {
		        classifier = 'sources'
		        from android.sourceSets.main.java.sourceFiles
		    }

		    artifacts {
		        archives androidSourcesJar
		    }
		}
		
- 修改你的项目根目录下的**.gitigore**文件

		# [Android] ========================
		# Built application files
		*.apk
		*.ap_

		# Files for the Dalvik VM
		*.dex

		# Java class files
		*.class

		# Generated files
		bin/
		gen/

		# Gradle files
		.gradle/
		build/

		# Local configuration file (sdk path, etc)
		local.properties

		# Proguard folder generated by Eclipse
		proguard/

		# Log Files
		*.log


		## Directory-based project format:
		.idea/

		## File-based project format:
		*.ipr
		*.iws

		## Plugin-specific files:

		# IntelliJ
		out/

		# mpeltonen/sbt-idea plugin
		.idea_modules/

		# JIRA plugin
		atlassian-ide-plugin.xml

		# Crashlytics plugin (for Android Studio and IntelliJ)
		com_crashlytics_export_strings.xml


		# [Maven] ========================
		target/
		pom.xml.tag
		pom.xml.releaseBackup
		pom.xml.versionsBackup
		pom.xml.next
		release.properties


		# [Gradle-Android] ========================

		# Ignore Gradle GUI config
		gradle-app.setting

		# Gradle Signing
		signing.properties
		trestle.keystore

		# Mobile Tools for Java (J2ME)
		.mtj.tmp/

		# Package Files #
		*.jar
		*.war
		*.ear

		# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
		hs_err_pid*

		# Misc
		/.idea/workspace.xml
		.DS_Store
		/captures
		**/*.iml
		*.class

- 修改项目根目录下的**gradle.properties**

		# Project-wide Gradle settings.

		# IDE (e.g. Android Studio) users:
		# Gradle settings configured through the IDE *will override*
		# any settings specified in this file.

		# For more details on how to configure your build environment visit
		# http://www.gradle.org/docs/current/userguide/build_environment.html

		# Specifies the JVM arguments used for the daemon process.
		# The setting is particularly useful for tweaking memory settings.
		# Default value: -Xmx10248m -XX:MaxPermSize=256m
		# org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

		# When configured, Gradle will run in incubating parallel mode.
		# This option should only be used with decoupled projects. More details, visit
		# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
		# org.gradle.parallel=true

		VERSION_NAME=0.0.1
		VERSION_CODE=1
		GROUP=com.github.github_username

		POM_DESCRIPTION=A library that does X, Y, and Z
		POM_URL=https://github.com/github_username/ProjectName
		POM_SCM_URL=https://github.com/github_username/ProjectName
		POM_SCM_CONNECTION=scm:git@github.com:github_username/ProjectName.git
		POM_SCM_DEV_CONNECTION=scm:git@github.com:github_username/ProjectName.git
		POM_LICENCE_NAME=The Apache Software License, Version 2.0
		POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
		POM_LICENCE_DIST=repo
		POM_DEVELOPER_ID=github_username
		POM_DEVELOPER_NAME=GitHub FullName
		
- 添加一个**README.md**文件来告诉其他开发者你的library是干什么的以及他的用法，如果你需要添加截屏的话，我推荐你使用[Screenr](https://play.google.com/store/apps/details?id=de.toastcode.screener)生成截屏。

### 设置GPG

如果你的电脑上没有安装GPG的话，那么你需要下载它，在MacOSX上，[下载命令](https://gist.github.com/IanVaughan/1932175)

如果你从来没有使用过GPG，那么第一步是创建GPG keys:

	$ gpg --gen-key
	
如果你存在解决不了的问题，你可以参考[Creating an encryption key](https://kb.iu.edu/d/awio#key)

接下来，你可以通过下面的命令找到你的key:

	$ gpg --list-keys
	
第一行看起来像XXXXX/YYYYYYY <date>,记住，'YYYYYYYY'部分，就是你的key ID。

现在，发布你的keys:

	$ gpg --keyserver hkp://keyserver.ubuntu.com --send-keys YYYYYYYY
	$ gpg --keyserver hkp://gpg.mit.edu --send-keys YYYYYYYY
	
你也可以使用其他的key服务器，你可能想确保你的keys是否已经发布：

	$ gpg --keyserver hkp://pgp.mit.edu --search-keys
	johndoe@example.com #使用你的email
	
你的库已经在[Gradle,Please](http://gradleplease.appspot.com/)中(别人可以轻松使用你的library)，上传你的project到[Maven Central](http://search.maven.org/),最简单的方法就是通过[Sonatype](http://central.sonatype.org/pages/ossrh-guide.html)

### Sonatype

1. 在[Sonatype](https://issues.sonatype.org/secure/Signup!default.jspa)创建一个账号。

2. 一旦登陆，创建一个[新的issue](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)

![https://d262ilb51hltx0.cloudfront.net/max/800/1*kV8pnaeikyXsgse0bWCeUA.png](https://d262ilb51hltx0.cloudfront.net/max/800/1*kV8pnaeikyXsgse0bWCeUA.png)

我已经为我的项目创建一个GitHub协议，所以我在新的issue中填写的字段如下:

**Group id**:com.github.<github_username>

**Project URL**:https://github.com/<github_username>/<project_name>

**SCM url**: https://github.com/<github_username>/<project_name>.git

**Username**:<sonatype_username>

**Already Synced to Central**:No

注意:那些括号只是占位符，你需要用适当的值替换

当你提交issue的时候，issue的细节就相当于上面的截图，提交之后，会花费两天处理你的issue，你将会收到一个关于你的配置已经准备好，你的库可以发布的确认信息。

不要部署直到你收到一封邮件证明你的问题已解决，

如果你的项目已经在中央仓库，那么就需要确保如何迁移到OSSRH。

修改你本地机器上的 ～/.gradle/gradle.properties:

	NEXUS_USERNAME=sonatype_username
	NEXUS_PASSWORD=sonatype_password
	
	signing.keyId=gpg_key_id 
	signing.password=gpg_password
	signing.secretKeyRingFile=/Users/username/.gnupg/secring.gpg
	org.gradle.daemon=true

当你发布这个库时是通过gradle.properties来验证的，所以确保你提供正确的nexus用户名和密码(这里是Sonatype的用户名和密码)，否则你会得到一个未经授权的401错误。

注意：如果你已经发布你的库了没那么你就不需要在JIRA上创建新的issue，因为对一每一个顶级的groupId来说只用创建一个JIRA，你现在已经有权限向你的groupId或者任何sub-groups部署新的工作(此处省略一部分，尼玛，太不好翻译了!!!)

### 发布

一旦你已经准备好发布你的library，打开Android Studio右边的Gradle视图，在**Tasks > upload**点击**UploadArchives**,将会上传你的库到[Sonatype Staging Repositories](https://oss.sonatype.org/#stagingRepositories);

去[Sonatype Staging Repositories](https://oss.sonatype.org/#stagingRepositories)确保你的**Sonatype**账号已经登陆，寻找你的暂存的库，他应该在列表的最后，选中他，然后点击close按钮，关闭意味着你准备发布，如果close操作成功，你将会看到一个Release按钮被激活，你需要刷新页面，点击release按钮，参照[帮助文档](https://books.sonatype.com/nexus-book/reference/staging-repositories.html)，如果你的库已经发布，那么你就可以返回JIRA然后发布评论来提升你的库，之后你会收到一个Sonatype的一个回应，几分钟之后你的库就可用了，然后几个小时之后就会被同步到Maven中央，再过几个小事就会显示在Gradle中。




