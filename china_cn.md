---
title: "China"
description: "Read official documentation on the Mapbox Android China Plugin which takes the Mapbox Map SDK for Android and configures it to maximize performance inside China."
sideNavSections:
  - title: "安装 China Plugin"
  - title: "压缩依赖包大小"
  - title: "使用正确的对象 Objects"
  - title: "China 地图样式"
  - title: "对图标进行偏移"
prependJs:
  - |
    import {
      CHINA_PLUGIN_VERSION
    } from '../../../constants';
---
# China
海外的很多地图服务，在国内访问时经常受限，或者存在访问慢的问题。Mapbox 很早之前就已经推出了Mapbox.cn 地图服务，任何用户都可以在国内（或通过中国的移动运营商）非常快速的访问 Mapbox 的地图。现在Mapbox 发布了这个Android China 插件，专门绑定了Mapbox Android SDK，并且自动配置了Mapbox.cn 的地图节点。此外，插件中的一个依赖项允许用户对地图上的图标进行坐标偏移，从而保证所有的图标都正确地显示在Mapbox.cn 地图上。

## 安装China Plugin
在开始使用Ching Plugin 插件进行开发之前，你需要在`build.gradle` 文件中添加适当的依赖项。China Plugin插件包含两种依赖项，但都已经包括了Mapbox Android Map SDK，该插件一般都已打包了Mapbox Android Map SDK 的最新版本。下面给出的两种依赖项都可以在二进制文件中找到。

### 选择正确的依赖项dependency
China Plugin插件提供两种依赖项可选（China 和Global），或者说 Flavors。两者最终会决定应用程序的终端用户的位置，无论他们是在国内还是国外。在国内使用Global 依赖项，会导致地图加载较慢，以及坐标精度低的问题。使用China 依赖项可以纠正这些问题，主要通过以下两种方式来实现的：第一个是获取地图服务的Mapbox API的节点。China 依赖项会使所有API请求和接收的地图切片服务都来自国内，从而使应用程序加载地图的速度非常快。第二点是China 依赖项包括[偏移模块（shift module）](#shifting-annotations)，允许你将所有的图标都精确地显示在地图上。这两种依赖项是可互换的，在这两者之间互换不需要更新代码，只要更改`build.gradle`文件中的依赖名即可。这可以让你非常灵活，在国内只需要单一应用程序即可，或一个应用程序包含两种依赖项——一个在国内使用，一个在国外使用。

### 添加依赖项dependency

1. 打开Android Studio。
2. 打开应用程序的`build.gradle` 文件。
3. 确保工程项目中`minSdkVersion` 的API 为14或更高。
4. 确定使用合适的依赖项（China 或Global 依赖项），并将其添加到您的依赖项中。
5. 单击同步按钮，从而根据Gradle文件来更新依赖项。

```groovy
repositories {
  maven { url "https://mapbox.bintray.com/mapbox" }
}

dependencies {
  // 使用China 依赖项
  implementation 'com.mapbox.mapboxsdk:mapbox-android-plugin-china:{{ CHINA_PLUGIN_VERSION }}'

  // 或者Global 依赖项
  implementation 'com.mapbox.mapboxsdk:mapbox-android-plugin-china-global:{{ CHINA_PLUGIN_VERSION }}'
}
```

## 压缩依赖包大小
Android 提供了几种解决方案来减少您的APK的大小。第一种是使用[ProGuard](https://developer.android.com/studio/build/shrink-code)，它会从应用程序及其依赖项（包括插件）中删除未使用的类和方法。ProGuard 包含在Android系统中，我们强烈建议遵循[官方文档](https://developer.android.com/studio/build/shrink-code)来设置ProGuard。

另一个选择是删除应用程序中用户不需要支持的架构Architectures。Mapbox SDK 有四种架构Architectures：

- `arm64-v8a`
- `armeabi-v7a`
- `x86`
- `x86_64`

以上所有这些文件都加起来会影响APK大小。例如，如果您的应用程序不需要支持`x86`，您可以删除`x86`和`x86_64`，从而节省一些空间。这是通过使用ABI 分裂完成的，它允许你为每个CPU 构建一个APK 文件，只包含相关的本地库。此过程在[Android Studio项目站点](http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits#TOC-ABIs-Splits)中进行了描述。如果您按照这种方式设置项目，除了Mapbox Map SDK 原生库之外，偏移模块（shift module）的原生代码也会被被选择性地分开。

## 使用正确的对象Objects
此外，本Android China 插件同时也封装了一些原有的地图类Classes，从而代替对应的一些常规的类。例如，在Android 的常规Map SDK中，一般可以在Activities 的布局文件中使用`MapView` 。如果您使用了China 插件，Studio 会通过lint 错误来提示您替换使用`ChinaMapView`，其实它封装了`MapView` 对象。其实，`ChinaMapView`仅设置了一些针对国内的默认参数，从而实现最优的效果。其中包括连接提供的地图瓦片的中国服务器等。在下面提供的图表中，我们列出了对象列表，这些通常会在Android Map SDK中使用到，以及对应到China 插件中的对象。

| Mapbox Map SDK for Android | China Plugin |
| --- | --- |
| `MapView` | `ChinaMapView` |
| `MapboxMapOptions` | `MapboxMapChinaOptions` |

如上所述，当您使用错误的地图类时，Android Lint 将尝试警告您。但不会在在每一个实例中都出现，所以我们仍然建议手动确认，并使用正确的对象。

## China 地图样式
Mapbox目前提供3个中国政府认证的China 地图样式，正好对应Mapbox 默认的Streets，Dark，以及Light这三种典型样式。在国内，这3个政府认证的China 地图样式的加载速度非常快。您可以在应用程序中手动设置样式，或者也可以使用插件中提供的常量来设置。下面的表格中列出了预设的一些常量，包括Java Constant，XML String），以及样式真实的URL地址。

| Java Constant | XML String | URL |
| --- | --- | --- |
| `ChinaStyle.MAPBOX_STREETS_CHINESE` | `mapbox_style_mapbox_streets_chinese` | `mapbox://styles/mapbox/streets-zh-v1` |
| `ChinaStyle.MAPBOX_LIGHT_CHINESE` | `mapbox_style_mapbox_dark_chinese` | `mapbox://styles/mapbox/light-zh-v1` |
| `ChinaStyle.MAPBOX_DARK_CHINESE` | `mapbox_style_mapbox_light_chinese` | `mapbox://styles/mapbox/dark-zh-v1` |

当使用这些SDK中的常量时，你将始终使用的Mapbox 最新版本的地图样式。相反，手动设置样式可以确保您对地图样式有更多的控制，并决定应用程序中的地图样式何时进行更新。

## 对图标进行偏移
为了符合中国政府对位置保密的要求，Mapbox 必须对一些默认的地图样式（以上3个地图样式）进行了偏移。这就意味着你还需要把显示在地图上的图标也进行偏移，从而使这些图标与偏移的地图样式相匹配。该模块其实是将WGS-84坐标转换为GCJ-02坐标。为了更好地理解为什么需要坐标转换，我们建议阅读这篇[Wiki条目](https://en.wikipedia.org/wiki/Restrictions_on_geographic_data_in_China#The_China_GPS_shift_problem)和[文章](http://www.travelandleisure.com/articles/digital-maps-skewed-china)。

该插件有一个`ShiftForChina`类，带有`String shift（double lon，double lat）`方法。 您可以将*unshifted*的经度和纬度坐标传递给`shift（）`方法。该方法返回表示`JSONObject`的`String`。使用此移位坐标`String`将数据添加到地图中。

```java
Location toLocation = new Location(fromLocation);
try {
  JSONObject jsonObject = new JSONObject(toJson);
  toLocation.setLongitude(jsonObject.getDouble("lon"));
  toLocation.setLatitude(jsonObject.getDouble("lat"));
} catch (JSONException jsonException) {
  jsonException.printStackTrace();
}
```
```Kotlin
val shiftedCoordinatesJson = shiftForChina.shift(unshiftedLong, unshiftedLat)
try {
	val jsonObject = JSONObject(shiftedCoordinatesJson)
	val shiftedLong = jsonObject.getDouble("lon")
	val shiftedLat = jsonObject.getDouble("lat")
	// You now have longitude and latitude values, which you can use however you'd like.
} catch (jsonException: JSONException) {
	jsonException.printStackTrace()
}
```

## 对位置进行偏移
[点击这里阅读如何在适用于Android的Mapbox核心库中使用`Mapbox LocationEngine`](https://www.mapbox.com/android-docs/core/overview/#locationengine)

当发生新的位置更新时，您需要手动将位置对象提供给插件的ShiftLocation类的shift（）方法。 此方法和类处理Location对象，而不是处理原始坐标值。 目前，最好的方法是创建自己的LocationEngine，扩展另一个并监听位置更新。 更新发生时，通过shift模块发送Location对象，并让locationEngine提供修改后的位置。

```Kotlin
// Called when the location has changed.
 
override fun onLocationChanged(location: Location) {
 
	if (needChinaShifted) {
	
	    shiftedDeviceLocation = ShiftLocation.shift(location)
	
	    // You now have longitude and latitude values, which you can use however you'd like.
	    
	}
  ```
  ```Java
  // Called when the location has changed.
 
@Override
public void onLocationChanged(Location location) {
 
	if (needChinaShifted) {
	    
	shiftedDeviceLocation = ShiftLocation.shift(location);
	    
	// You now have longitude and latitude values, which you can use however you'd like.
	
	}
}
```
}
