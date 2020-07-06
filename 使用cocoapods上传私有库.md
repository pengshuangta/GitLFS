# 使用cocoapods上传私有库
[TOC]
## 参考文献

> [(原创)组件化：发布CocoaPods私有/公有库](https://www.jianshu.com/p/649f0001925d)

> [iOS-使用CocoaPods创建私有仓库（一）](https://www.jianshu.com/p/1d6fa46e31f9)

> [CocoaPods 动/静态库混用封装组件化](https://www.jianshu.com/p/544df88b6a1e)

## 总结思路

> 开发者上传,重要的思路点总结

    一般开发者会创建两个仓库：
        源码仓库，用来进行库的实现和集成测试，并且实现podspec用于测试，在这个仓库的测试完成后，上传到远程仓库，并打上tag，所有podspec内书写的版本都会让cocoapods去寻找你指定源码地址内对应的tag版本。
        索引仓库，用来进行podspec文件的管理，将源码仓库测试完成的podspec单独创建一个仓库进行管理，在测试通过后上传到远程仓库，不需要进行tag的管理。最后是将podspec上传到cocoapods，进行索引的映射，索引仓库的地址，也是客户app在Podfile中进行指定的地址，pod install后，其实原理就是cocoapods会去你指定的地址寻找你指定名称的索引库，然后根据索引库中指定的地址去下载源文件，进行集成和依赖。
        
## 使用心得 
        
### 源码仓库流程

#### 创建源码仓库，并clone到本地文件夹中

#### 创建podspec脚手架,`pod lib + 名字`

#### 在脚手架中,代码和资源在创建的项目中的pods -> Development Pods中编写。

说明1： 用该方法创建的项目，默认遵守了MIT License协议
说明2： 注意层级，podfile文件在Example文件夹下，podspec文件在项目根目录下

#### 编辑podspec

```ObjC

#
# Be sure to run `pod lib lint ZlfTestPod003.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see https://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'ZlfTestPod003'
  s.version          = '0.0.13'
  s.summary          = 'A short description of ZlfTestPod003.'

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://gitlab.oifitech.com/zlf_SDK/zlftestpod03.git'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { '厦门声连网信息科技有限公司' => 'zlf@soundbus.cn' }
  s.source           = { :git => 'https://gitlab.oifitech.com/zlf_SDK/zlftestpod03.git', :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  # s.source_files = 'ZlfTestPod003/Classes/**/*'

  s.source_files = 'ZlfTestPod003/ZlfTestPod003/Home/Classes/*.{h,m}'

  s.requires_arc = true

  s.ios.deployment_target  = '9.0'
  # s.ios.vendored_frameworks = 'ZlfTestPod003/**/Classes/*.framework'
  
  # s.resource_bundles = {
  #   'ZlfTestPod003' => ['ZlfTestPod003/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'

  s.subspec 'Home' do |ss|
  	ss.source_files = 'ZlfTestPod003/ZlfTestPod003/Home/Classes/*.{h,m}'
  end

  s.subspec 'Footprint' do |ss|
  	ss.ios.vendored_frameworks = 'ZlfTestPod003/ZlfTestPod003/Footprint/Classes/*.framework'
  	ss.dependency 'AFNetworking', '~> 2.3'
  end

end

```

##### podspec文件关键字说明

```ObjC

#  基本信息的配置
name：框架名
version：当前版本（注意，是当前版本，假如你后续更新了新版本，需要修改此处）
summary：简要描述，在pod search的时候会显示该信息。
description：详细描述
homepage：页面链接
license：开源协议
author：作者
platform：支持最低ios版本
swift_version : swift对应的版本

# 源文件的配置
source：源码git地址
source_files：源文件（可以包含.h和.m）
subspec：子库
public_header_files：头文件(.h文件)
resource_bundles：资源文件（配置的文件会放到你自己指定的bundle中）

# 依赖的配置
frameworks：依赖的系统框架
vendored_frameworks：依赖的非系统框架
libraries：依赖的系统库
vendored_libraries：依赖的非系统的静态库
dependency：依赖的三方库

```

##### 本地验证

```

pod lib lint --use-libraries --allow-warnings --sources='https://gitlab.oifitech.com/zlf_SDK/zlftestpod03.git,https://github.com/CocoaPods/Specs'

```

##### 远程验证

```

pod spec lint --use-libraries --allow-warnings --sources='https://gitlab.oifitech.com/zlf_SDK/zlftestpod03.git,https://github.com/CocoaPods/Specs'

```

#### 上传git仓库并打tag

```

// 回到项目根目录下
git add .
git commit -m "此次内容"
git pull origin master --allow-unrelated-histories
git push
git tag 0.0.1
git push origin --tags

```

### 索引仓库流程

#### 创建索引仓库，并clone到本地文件夹中

#### 将`podspec`文件挪到仓库中

#### 注意事项

> `podspec`文件中的版本指定对应的是源码仓库在git中的tag号

#### 上传到git(不需要tag)

```

// 回到项目根目录下
git add .
git commit -m "此次内容"
git pull origin master --allow-unrelated-histories
git push

```

#### 上传到cocoapods

```

pod repo push ZlfTestPod003 ZlfTestPod003.podspec --use-libraries --allow-warnings --sources='https://gitlab.oifitech.com/zlf_SDK/zlftestpod03.git,https://github.com/CocoaPods/Specs'

```

### 项目中的集成

```

source 'https://gitlab.oifitech.com/zlf_SDK/zlftestpodspec003.git'
source 'https://github.com/CocoaPods/Specs.git'

platform :ios, '9.0'

#use_frameworks!

target 'Test57' do
  pod 'ZlfTestPod003'
end


```

> 注：私有库需要说明source地址,其中`https://github.com/CocoaPods/Specs.git`是官方库地址,因为我们在pospec中指定了framework的位置，所以不要use_frameworks

### 关于子库

#### 目录结构

> 一般在我们的源码库下区分子库，然后再分Classes

![370c32d5887176277bb2ccfcc6beefa5](使用cocoapods上传私有库.resources/2E9B9B77-30F0-4A25-AAFA-788AF22481E8.png)

#### pod子库

> 使用`pod 'ZlfTestPod003'`的方式进行集成，会导入所有的子库

> 使用`pod 'ZlfTestPod003/Home'`的方式进行集成，将会集成某个子库内容，根据子库中的设置来集成代码

#### 注意事项

> 's.source_files'主要目标是.h.m文件，'s.ios.vendored_frameworks'主要目标是.framework文件

> 子库不会自动依赖父库的依赖，除非在`s.dependency`中引用父类


        
    