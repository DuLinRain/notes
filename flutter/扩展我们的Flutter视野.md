# 扩展我们的Flutter视野
几乎任何应用程序都离不开网络，Flutter同样如此。在本文中，我将通过创建一个简单的，从服务端获取JSON数据的应用例子来讨论在**Flutter中的网络**。每个应用都必须具有**可访问性（ accessibility ）功能**，以迎合广大用户的需求，我们将在可访问性选项中对此进行介绍。 在最后一节中，我们将讨论**本地化**以使您的应用程序具有在全球范围内发展的能力，支持多种语言。

在本文中，我们将讨论如下几个主题：

- Flutter中的网络。
- Flutter中的可访问性。
- Flutter app的国际化。

### 一、Flutter中的网络
#### 1.1 使用packages
与许多平台一样，Flutter支持使用开发人员为Flutter和Dart生态系统提供的共享软件包。 通过使开发人员快速构建应用程序，而不用担心从头开发代码，从而促进了开发。 某些最常用的软件包包括但不限于：发出网络请求（HTTP）； 使用设备API，例如设备信息（device_info）； 查找信息并控制相机，包括支持相机源预览和捕获的图像（相机）； 使用GPS坐标（地理定位器）查找并使用设备的位置； 并使用第三方平台SDK（例如Firebase）。 您可以在https：/// pub中找到Flutter支持的软件包的完整列表。 dartlang.org/packages。

##### 1.1.1 给应用添加包依赖
一旦确定了要包括的软件包集，请按照以下步骤包括依赖性。 出于本示例的目的，我们选择了应用程序的HTTP包。 该软件包包含一组高级函数和类，可帮助开发人员在使用应用程序时消耗HTTP资源，并且与平台无关。 它同时支持命令行和浏览器

1. 创建依赖关系：打开位于应用程序文件夹内的pubspec.yaml文件，并在依赖关系下添加http:。所有软件包的版本号都在其pubspec.yaml文件中指定。 软件包的当前版本显示在软件包名称的旁边。 当您提到Plugin_Name_1：时，它被解释为Plugin_Name_1：any。 这表明可以使用任何版本的软件包。 建议使用特定版本，以确保应用程序在更新时不会中断。
2. 在已添加依赖性的地方安装软件包。 您可以通过运行flutter软件包get命令来安装它。 如果您使用的是Android Studio / IntelliJ，还可以单击pubspecs.yaml顶部的功能区中的Package Get选项。 如果您使用的是VS代码，请单击pubspec.yaml顶部操作区域右侧的“获取软件包”。
3. 在Dart代码中包含相应的import语句。 在这种情况下，它是导入包：http / http.dart。 如果您错过了任何东西，可以随时使用Pub上打包页面上的Installation选项卡选项进行交叉检查。
4. 此时，最好停止并重新启动应用程序，以避免在使用软件包时出现诸如MissingPluginException之类的错误。

##### 1.1.2 更新已有的包

在pubspec.yaml文件中添加软件包后，首次运行flutter软件包get（在IntelliJ中为Packages Get）时，Flutter将保存在pubspec.lock锁定文件中找到的版本。 要升级程序包，您可以运行Flutter程序包升级（IntelliJ中的升级依赖项）。 使用此命令，Flutter将检索软件包的最高可用版本。 如果您在pubspec.yaml中指定了范围约束，它将根据约束的指定获取更新。

### 二、Flutter中的可访问性

Making your app accessible to many users could be a great initiative. That also includes
people with disabilities, such as blindness, hearing, voice, or motor impairment. As per the
reports on disability by the World Health Organization, there are over 100 million users
across the globe who face physical challenges in their daily routine. Technology can be
revolutionary in helping people, and that's when building apps catering to their specific
needs can aid them well

使许多用户可以访问您的应用程序可能是一个不错的计划。 这也包括残疾人，例如失明，听力，声音或运动障碍。 根据世界卫生组织关于残疾的报告，全球有超过1亿用户在日常生活中面临身体挑战。 技术在帮助人们方面可能是革命性的，那就是构建满足他们特定需求的应用程序可以很好地帮助他们

Not all the users use the app in a specifically defined manner, so, a focus on accessibility will not only help users to download and use the app, but will also propagate to a new level of users.
Google provides an app to check for accessibility support that is available as accessibility scanner on Google Play at https:/​/​play.​google.​com/​store/​apps/​details?​id=​com.google.​android.​apps.​accessibility.​auditor This app enables you to find the accessibility provide that a developer can do within the app. For iOS, XCode provides Accessibility Inspector.
Flutter supports three components for accessibility support:

并非所有用户都以特定定义的方式使用该应用程序，因此，将重点放在可访问性上不仅会帮助用户下载和使用该应用程序，还将传播到新的用户级别。
Google提供了一个用于检查可访问性支持的应用程序，该应用程序可通过https:////play.google.com/store/apps/details?id=com.google.cn作为Google Play上的可访问性扫描程序使用。 android.apps.accessibility.auditor这个应用程序使您能够找到开发人员可以在应用程序中执行的可访问性。 对于iOS，XCode提供了辅助功能检查器。
Flutter支持以下三个可访问性支持组件：

#### 2.1 大字体
With age, not many can see the content the way they used to in their youth. Some face issues in reading the text clearly, especially when developers consider using the default size without taking into considering factors such as screen size and orientation. One of the quickest ways to do this is to ensure that the text scales in their accessibility options consider the device specifications of the consumers

随着年龄的增长，没有多少人能像他们年轻时那样看到内容。 有些人在清晰阅读文本时会遇到问题，特别是当开发人员考虑使用默认大小而不考虑屏幕大小和方向等因素时。 最快的方法之一是确保其可访问性选项中的文本缩放考虑到消费者的设备规格
#### 2.2 读屏
For those who are visually impaired, this accessiblity option can come in handy. It enables users to receive spoken feedback about the content of the screen. You could turn on VoiceOver in iOS, or TalkBack in an Android application on your device to navigate around your app. For example, when using TalkBack, users perform actions using gestures, and each action is complimented with an audible output that allows users to know that their gesture trigger was successful. There are three types of gestures in TalkBack: basic gestures, back-and-forth gestures, and angle gestures. Note that the users should use single gestures, even finger pressure, and a steady speed to have a seamless experience

对于视力障碍者，此可访问性选项会派上用场。 它使用户可以接收有关屏幕内容的语音反馈。 您可以在iOS设备上打开VoiceOver，或在设备上的Android应用程序中打开“话语提示”，以在应用程序中导航。 例如，在使用“话语提示”时，用户使用手势执行操作，并且每个操作都带有一个可听的输出，该输出允许用户知道其手势触发已成功。 “话语提示”中有三种手势：基本手势，来回手势和角度手势。 请注意，用户应使用单个手势，均匀的手指压力和稳定的速度来获得无缝的体验

#### 2.3 屏幕对比度
Specifying background and foreground colors with sufficient color contrast enables better readability for the users. This ratio ranges from 1 to 21, where 21 means the highest.
指定具有足够色彩对比度的背景色和前景色可为用户提供更好的可读性。 该比率范围是1到21，其中21表示最高。

The W3C recommends the following:
At least 4.5:1 for smaller text (below 18 point regular, or 14 point bold)
At least 3.0:1 for larger text (18 point and above regular, or 14 point and above
bold)

Accessibility is an important feature and should not be neglected. It ensures that the app is
open to a larger audience, enabling the chances for better application usage. It is equally
important to test the accessibility options before rolling out to the masses

可访问性是一项重要功能，不应忽略。 它确保了该应用程序可以吸引更多的受众，从而有机会更好地使用该应用程序。 在推广到大众之前，测试可访问性选项同样重要。

### Flutter app的国际化

As the name suggests, if your app will be by the international audience, you will have to think of providing locale support for the specific language of the target. That means you’ll need to write the app in a way that your app renders the values like text and layouts depending on each language or locale that the app supports. Flutter has made it simple by providing support by classes and widgets. Flutter supports the global localization classes for about 24 languages.

顾名思义，如果您的应用程序将受到国际受众的欢迎，那么您将不得不考虑为目标的特定语言提供语言环境支持。 这意味着您需要编写应用程序，使应用程序根据应用程序支持的每种语言或语言环境呈现诸如文本和布局之类的值。 Flutter通过提供类和小部件的支持使其变得简单。 Flutter支持大约24种语言的全球本地化课程。

### 总结
We first discussed how networking plays an important role in the apps, along with sample code for setting up and running a local server for fetching JSON code. This section was followed by understanding why accessibility is important, and what improvements developers can provide to support accessibility in the app. The next section showed how to make app support internationalization.
In the next chapter, we will discuss how to use platform powers to build apps

我们首先讨论了网络如何在应用程序中发挥重要作用，以及用于设置和运行用于获取JSON代码的本地服务器的示例代码。 在此部分之后，您将了解为什么可访问性很重要，以及开发人员可以提供哪些改进来支持应用程序中的可访问性。 下一节将介绍如何使应用程序支持国际化。
在下一章中，我们将讨论如何使用平台功能来构建应用