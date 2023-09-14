# Flutter_doc_CokBK_Net_Fetch_data_from_internet
 https://docs.flutter.dev/cookbook/networking/fetch-data
Set up app links for Android
============================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Navigation](https://docs.flutter.dev/cookbook/navigation)
3.  [Set up app links for Android](https://docs.flutter.dev/cookbook/navigation/set-up-app-links)

Deep linking is a mechanism for launching an app with a URI. This URI contains scheme, host, and path, and opens the app to a specific screen.

A *app link* is a type of deep link that uses `http` or `https` and is exclusive to Android devices.

Setting up app links requires one to own a web domain. Otherwise, consider using [Firebase Hosting](https://firebase.google.com/docs/hosting) or [GitHub Pages](https://pages.github.com/) as a temporary solution.

[](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#1-customize-a-flutter-application)1\. Customize a Flutter application
--------------------------------------------------------------------------------------------------------------------------------------

Write a Flutter app that can handle an incoming URL. This example uses the [go_router](https://pub.dev/packages/go_router) package to handle the routing. The Flutter team maintains the `go_router` package. It provides a simple API to handle complex routing scenarios.

1.  To create a new application, type `flutter create <app-name>`:

    content_copy

    ```
     $ flutter create deeplink_cookbook

    ```

2.  To include `go_router` package in your app, add a dependency for `go_router` to the project:

    To add the `go_router` package as a dependency, run `flutter pub add`:

    content_copy

    ```
     $ flutter pub add go_router

    ```

3.  To handle the routing, create a `GoRouter` object in the `main.dart` file:

[](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#2-modify-androidmanifestxml)2\. Modify AndroidManifest.xml
---------------------------------------------------------------------------------------------------------------------------

1.  Open the Flutter project with VS Code or Android Studio.
2.  Navigate to `android/app/src/main/AndroidManifest.xml` file.
3.  Add the following metadata tag and intent filter inside the `<activity>` tag with `.MainActivity`.

    Replace `example.com` with your own web domain.

    content_copy

    ```
     <meta-data android:name="flutter_deeplinking_enabled" android:value="true" />
     <intent-filter android:autoVerify="true">
         <action android:name="android.intent.action.VIEW" />
         <category android:name="android.intent.category.DEFAULT" />
         <category android:name="android.intent.category.BROWSABLE" />
         <data android:scheme="http" android:host="example.com" />
         <data android:scheme="https" />
     </intent-filter>

    ```

    info Note: The metadata tag flutter_deeplinking_enabled opts into Flutter's default deeplink handler. If you are using the third-party plugins, such as [uni_links](https://pub.dev/packages/uni_links), setting this metadata tag will break these plugins. Omit this metadata tag if you prefer to use third-party plugins.

[](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#3-hosting-assetlinksjson-file)3\. Hosting assetlinks.json file
-------------------------------------------------------------------------------------------------------------------------------

Host an `assetlinks.json` file in using a web server with a domain that you own. This file tells the mobile browser which Android application to open instead of the browser. To create the file, get the package name of the Flutter app you created in the previous step and the sha256 fingerprint of the signing key you will be using to build the APK.

### [](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#package-name)Package name

Locate the package name in `AndroidManifest.xml`, the `package` property under `<manifest>` tag. Package names are usually in the format of `com.example.*`.

### [](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#sha256-fingerprint)sha256 fingerprint

The process might differ depending on how the apk is signed.

#### [](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#using-google-play-app-signing)Using google play app signing

You can find the sha256 fingerprint directly from play developer console. Open your app in the play console, under Release> Setup > App Integrity> App Signing tab:

![Screenshot of sha256 fingerprint in play developer console](https://docs.flutter.dev/assets/images/docs/cookbook/set-up-app-links-pdc-signing-key.png)

#### [](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#using-local-keystore)Using local keystore

If you are storing the key locally, you can generate sha256 using the following command:

content_copy

```
keytool -list -v -keystore <path-to-keystore>

```

### [](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#assetlinksjson)assetlinks.json

The hosted file should look similar to this:

content_copy

```
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.example.deeplink_cookbook",
    "sha256_cert_fingerprints":
    ["FF:2A:CF:7B:DD:CC:F1:03:3E:E8:B2:27:7C:A2:E3:3C:DE:13:DB:AC:8E:EB:3A:B9:72:A1:0E:26:8A:F5:EC:AF"]
  }
}]

```

1.  Set the `package_name` value to your Android application ID.

2.  Set sha256_cert_fingerprints to the value you got from the previous step.

3.  Host the file at a URL that resembles the following: `<webdomain>/.well-known/assetlinks.json`

4.  Verify that your browser can access this file.

[](https://docs.flutter.dev/cookbook/navigation/set-up-app-links#testing)Testing
--------------------------------------------------------------------------------

You can use a real device or the Emulator to test an app link, but first make sure you have executed `flutter run` at least once on the devices. This ensures that the Flutter application is installed.

![Emulator screenshot](https://docs.flutter.dev/assets/images/docs/cookbook/set-up-app-links-emulator-installed.png)

To test only the app setup, use the adb command:

content_copy

```
adb shell 'am start -a android.intent.action.VIEW\
    -c android.intent.category.BROWSABLE\
    -d "http://<web-domain>/details"'\
    <package name>

```

info Note: This doesn't test whether the web files are hosted correctly, the command launches the app even if web files are not presented.

To test both web and app setup, you must click a link directly through web browser or another app. One way is to create a Google Doc, add the link, and tap on it.

If everything is set up correctly, the Flutter application launches and displays the details screen:

![Deeplinked Emulator screenshot](https://docs.flutter.dev/assets/images/docs/cookbook/set-up-app-links-emulator-deeplinked.png)Fetch data from the internet
============================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Networking](https://docs.flutter.dev/cookbook/networking)
3.  [Fetch data from the internet](https://docs.flutter.dev/cookbook/networking/fetch-data)

Fetching data from the internet is necessary for most apps. Luckily, Dart and Flutter provide tools, such as the `http` package, for this type of work.

This recipe uses the following steps:

1.  Add the `http` package.
2.  Make a network request using the `http` package.
3.  Convert the response into a custom Dart object.
4.  Fetch and display the data with Flutter.

[](https://docs.flutter.dev/cookbook/networking/fetch-data#1-add-the-http-package)1\. Add the `http` package
------------------------------------------------------------------------------------------------------------

The [`http`](https://pub.dev/packages/http) package provides the simplest way to fetch data from the internet.

To add the `http` package as a dependency, run `flutter pub add`:

content_copy

```
$ flutter pub add http

```

Import the http package.

content_copy

```
import 'package:http/http.dart' as http;
```

Additionally, in your AndroidManifest.xml file, add the Internet permission.

content_copy

```
<!-- Required to fetch data from the internet. -->
<uses-permission android:name="android.permission.INTERNET" />

```

[](https://docs.flutter.dev/cookbook/networking/fetch-data#2-make-a-network-request)2\. Make a network request
--------------------------------------------------------------------------------------------------------------

This recipe covers how to fetch a sample album from the [JSONPlaceholder](https://jsonplaceholder.typicode.com/) using the [`http.get()`](https://pub.dev/documentation/http/latest/http/get.html) method.

content_copy

```
Future<http.Response> fetchAlbum() {
  return http.get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));
}
```

The `http.get()` method returns a `Future` that contains a `Response`.

-   [`Future`](https://api.flutter.dev/flutter/dart-async/Future-class.html) is a core Dart class for working with async operations. A Future object represents a potential value or error that will be available at some time in the future.
-   The `http.Response` class contains the data received from a successful http call.

[](https://docs.flutter.dev/cookbook/networking/fetch-data#3-convert-the-response-into-a-custom-dart-object)3\. Convert the response into a custom Dart object
--------------------------------------------------------------------------------------------------------------------------------------------------------------

While it's easy to make a network request, working with a raw `Future<http.Response>` isn't very convenient. To make your life easier, convert the `http.Response` into a Dart object.

### [](https://docs.flutter.dev/cookbook/networking/fetch-data#create-an-album-class)Create an `Album` class

First, create an `Album` class that contains the data from the network request. It includes a factory constructor that creates an `Album` from JSON.

Converting JSON by hand is only one option. For more information, see the full article on [JSON and serialization](https://docs.flutter.dev/data-and-backend/serialization/json).

content_copy

```
class Album {
  final int userId;
  final int id;
  final String title;

  const Album({
    required this.userId,
    required this.id,
    required this.title,
  });

  factory Album.fromJson(Map<String, dynamic> json) {
    return Album(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
    );
  }
}
```

### [](https://docs.flutter.dev/cookbook/networking/fetch-data#convert-the-httpresponse-to-an-album)Convert the `http.Response` to an `Album`

Now, use the following steps to update the `fetchAlbum()` function to return a `Future<Album>`:

1.  Convert the response body into a JSON `Map` with the `dart:convert` package.
2.  If the server does return an OK response with a status code of 200, then convert the JSON `Map` into an `Album` using the `fromJson()` factory method.
3.  If the server does not return an OK response with a status code of 200, then throw an exception. (Even in the case of a "404 Not Found" server response, throw an exception. Do not return `null`. This is important when examining the data in `snapshot`, as shown below.)

content_copy

```
Future<Album> fetchAlbum() async {
  final response = await http
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  if (response.statusCode == 200) {
    // If the server did return a 200 OK response,
    // then parse the JSON.
    return Album.fromJson(jsonDecode(response.body));
  } else {
    // If the server did not return a 200 OK response,
    // then throw an exception.
    throw Exception('Failed to load album');
  }
}
```

Hooray! Now you've got a function that fetches an album from the internet.

[](https://docs.flutter.dev/cookbook/networking/fetch-data#4-fetch-the-data)4\. Fetch the data
----------------------------------------------------------------------------------------------

Call the `fetchAlbum()` method in either the [`initState()`](https://api.flutter.dev/flutter/widgets/State/initState.html) or [`didChangeDependencies()`](https://api.flutter.dev/flutter/widgets/State/didChangeDependencies.html) methods.

The `initState()` method is called exactly once and then never again. If you want to have the option of reloading the API in response to an [`InheritedWidget`](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html) changing, put the call into the `didChangeDependencies()` method. See [`State`](https://api.flutter.dev/flutter/widgets/State-class.html) for more details.

content_copy

```
class _MyAppState extends State<MyApp> {
  late Future<Album> futureAlbum;

  @override
  void initState() {
    super.initState();
    futureAlbum = fetchAlbum();
  }
  // ---
}
```

This Future is used in the next step.

[](https://docs.flutter.dev/cookbook/networking/fetch-data#5-display-the-data)5\. Display the data
--------------------------------------------------------------------------------------------------

To display the data on screen, use the [`FutureBuilder`](https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html) widget. The `FutureBuilder` widget comes with Flutter and makes it easy to work with asynchronous data sources.

You must provide two parameters:

1.  The `Future` you want to work with. In this case, the future returned from the `fetchAlbum()` function.
2.  A `builder` function that tells Flutter what to render, depending on the state of the `Future`: loading, success, or error.

Note that `snapshot.hasData` only returns `true` when the snapshot contains a non-null data value.

Because `fetchAlbum` can only return non-null values, the function should throw an exception even in the case of a "404 Not Found" server response. Throwing an exception sets the `snapshot.hasError` to `true` which can be used to display an error message.

Otherwise, the spinner will be displayed.

content_copy

```
FutureBuilder<Album>(
  future: futureAlbum,
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return Text(snapshot.data!.title);
    } else if (snapshot.hasError) {
      return Text('${snapshot.error}');
    }

    // By default, show a loading spinner.
    return const CircularProgressIndicator();
  },
)
```

[](https://docs.flutter.dev/cookbook/networking/fetch-data#why-is-fetchalbum-called-in-initstate)Why is fetchAlbum() called in initState()?
-------------------------------------------------------------------------------------------------------------------------------------------

Although it's convenient, it's not recommended to put an API call in a `build()` method.

Flutter calls the `build()` method every time it needs to change anything in the view, and this happens surprisingly often. The `fetchAlbum()` method, if placed inside `build()`, is repeatedly called on each rebuild causing the app to slow down.

Storing the `fetchAlbum()` result in a state variable ensures that the `Future` is executed only once and then cached for subsequent rebuilds.

[](https://docs.flutter.dev/cookbook/networking/fetch-data#testing)Testing
--------------------------------------------------------------------------

For information on how to test this functionality, see the following recipes:

-   [Introduction to unit testing](https://docs.flutter.dev/cookbook/testing/unit/introduction)
-   [Mock dependencies using Mockito](https://docs.flutter.dev/cookbook/testing/unit/mocking)

[](https://docs.flutter.dev/cookbook/networking/fetch-data#complete-example)Complete example
--------------------------------------------------------------------------------------------

content_copy

```
import 'dart:async';
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<Album> fetchAlbum() async {
  final response = await http
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  if (response.statusCode == 200) {
    // If the server did return a 200 OK response,
    // then parse the JSON.
    return Album.fromJson(jsonDecode(response.body));
  } else {
    // If the server did not return a 200 OK response,
    // then throw an exception.
    throw Exception('Failed to load album');
  }
}

class Album {
  final int userId;
  final int id;
  final String title;

  const Album({
    required this.userId,
    required this.id,
    required this.title,
  });

  factory Album.fromJson(Map<String, dynamic> json) {
    return Album(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
    );
  }
}

void main() => runApp(const MyApp());

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late Future<Album> futureAlbum;

  @override
  void initState() {
    super.initState();
    futureAlbum = fetchAlbum();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fetch Data Example',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Fetch Data Example'),
        ),
        body: Center(
          child: FutureBuilder<Album>(
            future: futureAlbum,
            builder: (context, snapshot) {
              if (snapshot.hasData) {
                return Text(snapshot.data!.title);
              } else if (snapshot.hasError) {
                return Text('${snapshot.error}');
              }

              // By default, show a loading spinner.
              return const CircularProgressIndicator();
            },
          ),
        ),
      ),
    );
  }
}
```
