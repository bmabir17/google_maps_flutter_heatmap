# Google Maps for Flutter (Developers Preview)

[![pub package](https://img.shields.io/pub/v/google_maps_flutter_heatmap.svg)](https://pub.dartlang.org/packages/google_maps_flutter_heatmap)
![Flutter CI](https://github.com/bmabir17/google_maps_flutter_heatmap/workflows/Dart%20CI/badge.svg?branch=master)

A Flutter plugin that provides a [Google Maps](https://developers.google.com/maps/) widget with heatmap support.

## Developers Preview Status
The plugin relies on Flutter's new mechanism for embedding Android and iOS views.
As that mechanism is currently in a developers preview, this plugin should also be
considered a developers preview.

Known issues are tagged with the [platform-views](https://github.com/flutter/flutter/labels/a%3A%20platform-views) and/or [maps](https://github.com/flutter/flutter/labels/p%3A%20maps) labels.

To use this plugin on iOS you need to opt-in for the embedded views preview by
adding a boolean property to the app's `Info.plist` file, with the key `io.flutter.embedded_views_preview`
and the value `YES`.

The API exposed by this plugin is not yet stable, and we expect some breaking changes to land soon.


## Usage

To use this plugin, add `google_maps_flutter_heatmap` as a [dependency in your pubspec.yaml file](https://flutter.io/platform-plugins/).

## Getting Started

Get an API key at <https://cloud.google.com/maps-platform/>.

### Android

Specify your API key in the application manifest `android/app/src/main/AndroidManifest.xml`:

```xml
<manifest ...
  <application ...
    <meta-data android:name="com.google.android.geo.API_KEY"
               android:value="YOUR KEY HERE"/>
```

### iOS

Specify your API key in the application delegate `ios/Runner/AppDelegate.m`:

```objectivec
#include "AppDelegate.h"
#include "GeneratedPluginRegistrant.h"
#import "GoogleMaps/GoogleMaps.h"

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [GMSServices provideAPIKey:@"YOUR KEY HERE"];
  [GeneratedPluginRegistrant registerWithRegistry:self];
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}
@end
```

Or in your swift code, specify your API key in the application delegate `ios/Runner/AppDelegate.swift`:

```swift
import UIKit
import Flutter
import GoogleMaps

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?
  ) -> Bool {
    GMSServices.provideAPIKey("YOUR KEY HERE")
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```
Opt-in to the embedded views preview by adding a boolean property to the app's `Info.plist` file
with the key `io.flutter.embedded_views_preview` and the value `YES`.

### Both


You can now add a `GoogleMap` widget to your widget tree.

The map view can be controlled with the `GoogleMapController` that is passed to
the `GoogleMap`'s `onMapCreated` callback.

### Sample Usage

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:google_maps_flutter_heatmap/google_maps_flutter_heatmap.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Google Maps Demo',
      home: MapSample(),
    );
  }
}

class MapSample extends StatefulWidget {
  @override
  State<MapSample> createState() => MapSampleState();
}

class MapSampleState extends State<MapSample> {
  Completer<GoogleMapController> _controller = Completer();
  final Set<Heatmap> _heatmaps = {};
  static final CameraPosition _kGooglePlex = CameraPosition(
    target: LatLng(37.42796133580664, -122.085749655962),
    zoom: 14.4746,
  );
  LatLng _heatmapLocation = LatLng(37.42796133580664, -122.085749655962);
  static final CameraPosition _kLake = CameraPosition(
      bearing: 192.8334901395799,
      target: LatLng(37.43296265331129, -122.08832357078792),
      tilt: 59.440717697143555,
      zoom: 19.151926040649414);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: GoogleMap(
        mapType: MapType.hybrid,
        initialCameraPosition: _kGooglePlex,
        heatmaps: _heatmaps,
        onMapCreated: (GoogleMapController controller) {
          _controller.complete(controller);
        },
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _goToTheLake,
        label: Text('To the lake!'),
        icon: Icon(Icons.directions_boat),
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _addHeatmap,
        label: Text('Add Heatmap'),
        icon: Icon(Icons.add_box),
      ),
    );
  }
  void _addHeatmap(){
    setState(() {
      _heatmaps.add(
        Heatmap(
          heatmapId: HeatmapId(_heatmapLocation.toString()),
          points: _createPoints(_heatmapLocation),
          radius: 20,
          visible: true,
          gradient:  HeatmapGradient(
            colors: <Color>[Colors.green, Colors.red], startPoints: <double>[0.2, 0.8]
          )
        )
      );
    });
  }
  //heatmap generation helper functions
  List<WeightedLatLng> _createPoints(LatLng location) {
    final List<WeightedLatLng> points = <WeightedLatLng>[];
    //Can create multiple points here
    points.add(_createWeightedLatLng(location.latitude,location.longitude, 1));
    points.add(_createWeightedLatLng(location.latitude-1,location.longitude, 1)); 
    return points;
  }

  WeightedLatLng _createWeightedLatLng(double lat, double lng, int weight) {
    return WeightedLatLng(point: LatLng(lat, lng), intensity: weight);
  }

  Future<void> _goToTheLake() async {
    final GoogleMapController controller = await _controller.future;
    controller.animateCamera(CameraUpdate.newCameraPosition(_kLake));
  }
}
```

See the `example` directory for a complete sample app.

[![Find and sponsor issues](https://issuehunt.io/static/embed/issuehunt-button-v1.svg)](https://issuehunt.io/r/bmabir17/google_maps_flutter_heatmap)
