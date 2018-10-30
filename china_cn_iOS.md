# Mapbox China plugin for iOS
当前版本：v0.0.1
- 支持中国用户
- 坐标系转化为GCJ-02
- 海外的很多地图服务，在国内访问时经常受限，或者存在访问慢的问题。Mapbox 很早之前就已经推出了Mapbox.cn 地图服务，任何用户都可以在国内（或通过中国的移动运营商）非常快速的访问 Mapbox 的地图。现在Mapbox 发布了这个Android China 插件，专门绑定了Mapbox Android SDK，并且自动配置了Mapbox.cn 的地图节点。此外，插件中的一个依赖项允许用户对地图上的图标进行坐标偏移，从而保证所有的图标都正确地显示在Mapbox.cn 地图上。

Mapbox China for iOS 为使用 Mapbox China 地图提供了基础。该插件包含使用 Mapbox China 样式的便捷方法，将 WGS-84 坐标转换为 GCJ-02 的坐标系，以及提供 GCJ-02 坐标位置更新的自定义位置管理器。

## 从这里开始
要访问 Mapbox China 样式，您需要一个特殊的access token。请填写 https://www.mapbox.cn/contact 上的表格，即可申请特殊的access token。没有此access token，您将无法访问 Mapbox China 矢量切片。

## 安装
通过安装适用于 iOS 的 Mapbox Maps SDK 和 Mapbox China 插件来设置 Mapbox China 插件，您可以按照我们的[安装指南](https://www.mapbox.com/install/ios/)中的说明安装 Maps SDK。

### CocoaPods
在您的Podfile中添加下面的语句：
```
pod 'MapboxChinaPlugin'
```
接下来通过在命令行中运行`pod update`来更新工程。

### Carthage
在您的Cartfile中添加下面的语句：
```
binary "https://www.mapbox.cn/mapbox-china-plugin/ios/Mapbox-iOS-China-Plugin.json" ~> 0.0.1
```
通过在终端中运行`carthage update`来更新工程。

## 设置工程
### 改变API端点
要接收 Mapbox China 矢量切片，请将`MGLMapboxAPIBaseURL`作为键添加到`Info.plist`中，并将`https://api.mapbox.cn`设置为其值。这会切换地图样式的API端点。

## Mapbox China样式
Mapbox目前提供三种中国政府认可的地图样式：Mapbox街景地图、浅色地图和暗黑地图。默认使用Mapbox街景地图。

该插件提供了用于更新地图样式URL的便捷方法。

| 样式名称 | URL字符串 | MGLStyle的便捷方法 |
| --- | --- | --- |
| 街景地图 | `mapbox://styles/mapbox/streets-zh-v1` | `mbcn_streetsChineseStyleURL` |
| 暗黑地图 | `mapbox://styles/mapbox/dark-zh-v1` | `mbcn_darkChineseStyleURL` |
| 浅色地图 | `mapbox://styles/mapbox/light-zh-v1` | `mbcn_lightChineseStyleURL` |

## 使用Mapbox China插件
### 添加插件
`-addToMapView：`方法将Mapbox China 插件添加到已初始化的`MGLMapView`中。此方法将地图的样式URL切换为中国样式，并配置使用GCJ-02坐标的位置管理器。
Swift
```Swift
import MapboxChinaPlugin
 
class ViewController: UIViewController, MGLMapViewDelegate {
 
override func viewDidLoad() {
super.viewDidLoad()
let mapView = MGLMapView(frame: view.bounds)
let chinaPlugin = MBXChinaPlugin()
chinaPlugin.add(to: mapView)
view.addSubview(mapView)
}
 
}
```
Objective C
```Objective C
import MapboxChinaPlugin;
 
@interface ViewController ()
 
@end
 
@implementation ViewController
 
- (void)viewDidLoad {
[super viewDidLoad];
MGLMapView *mapView = [[MGLMapView alloc] initWithFrame:self.view.bounds];
MBXChinaPlugin *chinaPlugin = [[MBXChinaPlugin alloc] init];
[chinaPlugin addToMapView:mapView];
}
 
@end
```

### 坐标系转换
默认情况下，Maps SDK使用WGS-84坐标。该插件可将WGS-84坐标转换为GCJ-02坐标，以符合中国政府的制图要求。目前，此插件支持转移`CLLocationCoordinate2D`和[`MGLShape`](https://www.mapbox.com/ios-sdk/api/4.5.0/Classes/MGLShape.html)对象。 转换由[`NSValueTransformer`](https://developer.apple.com/documentation/foundation/nsvaluetransformer)和`MGLConvertToDatumTransformerName`管理，后者是[`NSValueTransformerName`](https://developer.apple.com/documentation/foundation/nsvaluetransformername)。 通过将插件添加到地图视图或直接初始化`MBCNGCJCoordinateTransformer`，可以使`MGLConvertToDatumTransformerName`可用。
Swift
```Swift
let transformerName = NSValueTransformerName(rawValue: MGLConvertToDatumTransformerName)
let transformer = ValueTransformer(forName: transformerName)
```
Objective C
```Objective C
NSValueTransformer *transformer = [NSValueTransformer valueTransformerForName:MGLConvertToDatumTransformerName];
```
要移动注释，请移动该注释的坐标属性。这会将WGS-84坐标转换为GCJ-02坐标。这也可用于设置地图视图的中心坐标。
Swift
```Swift
let unshiftedCoordinate = CLLocationCoordinate2D(latitude: 31.22894, longitude: 121.45434)
 
// Use the value transformer to shift the coordinate.
guard let transformedValue = transformer.transformedValue(NSValue(mglCoordinate: unshiftedCoordinate)) as? NSValue else { return }
let annotation = MGLPointAnnotation()
 
// Convert the NSValue back to an CLLocationCoordinate2D. Set the annotation's coordinate property to that shifted value.
let shiftedCoordinate = transformedValue.mglCoordinateValue
annotation.coordinate = shiftedCoordinate
mapView.addAnnotation(annotation)
 
// Set the center coordinate for the map view to the transformed coordinate.
mapView.setCenter(shiftedCoordinate, zoomLevel: 15, animated: false)
```
Objective C
```Objective C
CLLocationCoordinate2D unshiftedCoordinate = CLLocationCoordinate2DMake(31.22894, 121.45434);
 
// Use the value transformer to shift the coordinate.
NSValue *transformedValue = [transformer transformedValue:[NSValue valueWithMGLCoordinate:unshiftedCoordinate]];
MGLPointAnnotation *annotation = [[MGLPointAnnotation alloc] init];
 
// Convert the NSValue back to an CLLocationCoordinate2D. Set the annotation's coordinate property to that shifted value.
CLLocationCoordinate2D shiftedCoordinate = [transformedValue MGLCoordinateValue];
annotation.coordinate = shiftedCoordinate;
[mapView addAnnotation:annotation];
 
// Set the center coordinate for the map view to the transformed coordinate.
[mapView setCenterCoordinate:shiftedCoordinate zoomLevel:15 animated:NO];
```
此方法也适用于`MGLShape`和`MGLFeature`对象，包括使用`GeoJSON`创建的对象。
```Swift
let originalShape = [
CLLocationCoordinate2D(latitude: 31.22869, longitude: 121.4534),
CLLocationCoordinate2D(latitude: 31.22894, longitude: 121.45319),
CLLocationCoordinate2D(latitude: 31.2287, longitude: 121.45279),
CLLocationCoordinate2D(latitude: 31.22844, longitude: 121.45301),
CLLocationCoordinate2D(latitude: 31.22869, longitude: 121.4534)
]
 
let shape = MGLPolygon(coordinates: originalShape, count: UInt(originalShape.count))
 
// Shift the shape's coordinates.
let transformedValue = transformer.transformedValue(shape) as! MGLPolygon
```
Objective C
```Objective C
CLLocationCoordinate2D originalShape[5] = {
CLLocationCoordinate2DMake(31.22869, 121.4534),
CLLocationCoordinate2DMake(31.22894, 121.45319),
CLLocationCoordinate2DMake(31.2287, 121.45279),
CLLocationCoordinate2DMake(31.22844, 121.45301),
CLLocationCoordinate2DMake(31.22869, 121.4534)
};
 
MGLPolygon *shape = [MGLPolygon polygonWithCoordinates:originalShape count:5];
 
// Shift the shape's coordinates.
MGLPolygon *transformedShape = [transformer transformedValue:shape];
```
