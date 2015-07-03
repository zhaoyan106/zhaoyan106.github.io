---
layout: post
title: "项目插件化-CocoaPods的使用"
date: 2015-07-03 10:52:05 +0800
comments: true
categories: 
---
####一、CocoaPods的安装
######1、安装ruby环境,添加淘宝ruby镜像

	$ gem sources --remove https://rubygems.org/
	//等有反应之后再敲入以下命令
	$ gem sources -a http://ruby.taobao.org/
	
######2、查看是否设置成功:

	$ gem sources -l
	
######3、然后安装cocoapods:

	$ sudo gem install cocoapods

######4、查看cocoapods是否支持某个类库
	
	$ pod search 类库名,支持模糊查询(如:AFNetworking)
	
####二、CocoaPods的使用
######1、在项目根目录下新建一个“Podfile”的文件
`注:Podfile也可以放在任何位置,但是需要在Podfile顶部使用”xcodeproj”关键字指定工程的路径,如下:`

	xcodeproj"/User/Desktop/CocoaPodsDemo/CocoaPodsDemo.xcworkspace"
	platform:ios
	pod 'JSONKit',		  '~>1.4'
	pod 'Reachability',  '~>3.0.0'
	pod 'AFNetworking',	  '~>2.2.4'
	
######2、在项目podfile所在目录下运行(会在你当前项目中导入podfile所配置的库,所以要在项目目录下运行)
`执行pod install命令后,生成的文件将在Podfile所在的目录`

	$ pod install
	
注意上述命令运行完毕后终端输出的最后一段话,意思就是以后打开项目就用CocoaPodsDemo.xcworkspace 打开，而不是之前的.xcodeproj文件
####三、使用过程中遇到的一些问题
######1、引入第三方库后找不到头文件

在项目的Targe- Build Settings - Search Paths - User Header Searcj Paths中添加 ${SRCROOT} 值为 recursive

######2、如何删除cocopods

1、删除工程文件夹下的Podfile、Podfile.lock及Pods文件夹

2、删除xcworkspace文件

3、使用xcodeproj文件打开工程，删除Frameworks组下的Pods.xcconfig及libPods.a引用

4、在工程设置中的Build Phases下删除Check Pods Manifest.lock及Copy Pods Resources

####四、Podfile.lock文件的作用
在使用CocoaPods，执行完pod install之后，会生成一个Podfile.lock文件。这个文件看起来跟我们关系不大，实际上绝对不应该忽略它。该文件用于保存已经安装的Pods依赖库的版本

Podfile.lock文件最大得用处在于多人开发。当团队中的某个人执行完pod install命令后，生成Podfile.lock文件就记录下了当时最新Pods依赖库的版本，这时团队中的其它人check下来这份包含Podfile.lock文件的工程以后，再去执行pod install命令时，获取下来的Pods依赖库的版本就和最开始用户获取到的版本一致。

`如果没有Podfile.lock文件，后续所有用户执行pod install命令都会获取最新版本的SBJson，这就有可能造成同一个团队使用的依赖库版本不一致，这对团队协作来说绝对是个灾难！`

在这种情况下，如果团队想使用当前最新版本的SBJson依赖库，有两种方案：

1、更改Podfile，使其指向最新版本的SBJson依赖库；

2、执行pod update命令；

鉴于Podfile.lock文件对团队协作如此重要，我们需要将它添加到版本管理中。

####五、制作自己的Cocopods库
######1、在github上新建一个工程
license类型

正规的仓库都应该有一个license文件，Pods依赖库对这个文件的要求更严，是必须要有的。因此最好在这里让github创建一个，也可以自己后续再创建。我使用的license类型是MIT。
#####2、podspec创建
把项目clone到本地然后在根目录下新建MyPodDemo.podspec或者

	$ pod spec create MyPodDemo
	
podsepc的编写

	Pod::Spec.new do |s| 
	s.name = "MyPodDemo" 
	s.version = "0.0.1" 
	s.summary = "A short description of MyPodDemo." 
	s.description = <<-DESC A longer description of MyPodDemo in Markdown format. * Think: Why did you write this? What is the focus? What does it do? * CocoaPods will be using this to generate tags, and improve search results. * Try to keep it short, snappy and to the point. * Finally, don't worry about the indent, CocoaPods strips it! 
	DESC 
	s.homepage = "https://github.com/goingta/MyPodDemo" 
	s.license = "MIT" s.author = { "goingta" => "tangle1128@gmail.com" } 
	s.source = { :git => "https://github.com/goingta/MyPodDemo.git", :tag => "0.0.1" } s.source_files = "MyPodDemo/Src", "MyPodDemo/Src/**/*.{h,m}" 
	s.requires_arc = true # s.framework = "SomeFramework" # 
	s.frameworks = "SomeFramework", "AnotherFramework" 
	# s.library = "iconv" 
	# s.libraries = "iconv", "xml2" 
	# s.dependency "JSONKit", "~> 1.4" 
	# s.dependency "AFNetworking", "~> 2.2.4" end

属性解析

	name: 导入pod后的目录名 
	version: 当前版本号 
	deployment_target: 配置的target 
	prefix_header_file: 预编译头文件路径，将该文件的内容插入到Pod的pch文件内 
	source: 来源的具体路径，是http链接还是本地路径 
	requires_arc: 是否需要arc 
	source_files: 指定该目录下包含哪些文件 
	其他可选参数还包括：
	dependency: 指定依赖，如果依赖的库不存在或者依赖库的版本不符合要求将会报错 
	libraries: 指定导入的库，比如sqlite3 
	frameworks: 指定导入的framework 
	weak_frameworks: 弱链接，比如说一个项目同时兼容iOS6和iOS7，但某一个framework只在iOS7上有，这时候如果用强链接，那么在iOS7上运行就会crash，使用weak_frameworks可以避免这种情况。

整个podspec语法是一个嵌套结构从Pod::Spec.new do |s|到最后一个end是最大的循环，表示整个podspec导入的文件。中间每一个subspec到end结束是一个子目录，Pods会为每个subspec创建一个逻辑目录，相当于Xcode的group概念。|**|中间是subspec的名字，可以随便命名，但后面使用的名称必须一致。

通配符说明

	a{bb,bc}def.{h,m}表示四个文件abbdef.h abbdef.m abcdef.h abcdef.m
	
	*.{h,m,mm}表示所有的.h .m .mm文件
	Class/**/*.{h,m}表示Class目录下的所有.h .m文件

写完podspec文件后使用pod spec lint验证spec是否合格,有error则需要修改

配置非ARC文件
一个私有的pods库中某几个文件是在非ARC时代写的,如果要进行修改工程量浩大,于是乎要对这几个文件单独处理,这几个文件不使用arc其他文件使用arc,网上查了一些资料,只需要对source_file进行修改并排除那几个不使用ARC的文件就可以了,大致修改如下：

	Pod::Spec.new do |s| 
	s.name = "MyPodDemo" s.version = "0.0.1" 
	s.summary = "A short description of MyPodDemo. 
	s.homepage = "https://github.com/goingta/MyPodDemo" 
	s.license = "MIT" 
	s.author = { "goingta" => "tangle1128@gmail.com" } 
	s.source = { :git => "https://github.com/goingta/MyPodDemo.git", :tag => "0.0.1" } s.source_files = "MyPodDemo" 
	non_arc_files = 'MyPodDemo/NoArcFile1.{h,m}','MyPodDemo/NoArcFile2.{h,m}' 
	s.requires_arc = true 
	
	s.exclude_files = non_arc_files 
	s.subspec 'no-arc' do |sna| 
	sna.requires_arc = false 
	sna.source_files = non_arc_files 
	end

#####3、上传代码至github
#####4、私有库实现,编写podfile
如果由于某些原因我们编写的库不能公开,但是又想使用pods来进行管理,要怎么办呢?

首先我们要将我们刚刚在github上建的仓库改为Private(不然还用Public搞毛啊)

然后修改我们项目的podfile,与已加入Cocopods仓库的公有库相比我们只需要指明私有库低git地址,如下:

	platform :ios, '6.0' 
	pod 'MyPodDemo', :git => 'https://github.com/goingta/MyPodDemo.git' //私有库 
	pod 'CocoaLumberjack'//公有库
	

