
# Otlaat

This project is powered by Flutter **1.22.6** and Dart **2.10.5**.

To select and install a specific version of flutter, we recommend using FVM: [fvm.app](https://fvm.app)

Bloc ([flutter_bloc](https://pub.dev/packages/flutter_bloc)) is used as an approach to state management, more precisely Cubit over Bloc from the same package. Bloc/Cubit handles `Events` and changes `States`. Screens and Widgtes subscribe to Bloc changes and send events (for Cubit, this is a direct method call).


The main packages used in the project:

```yaml
  flutter_bloc: "^6.1.1"
  injector: "^1.0.9"
  dio: "^3.0.10"
  google_maps_flutter: "^1.2.0"
  mad_pay: 1.0.0
  flutter_checkout_payment: "^0.1.0+2"
  google_sign_in: "^4.5.5"
  flutter_facebook_auth: ^3.3.3-no-nullsafety
  sign_in_with_apple: "^2.5.4"
  intl: "^0.16.1"
  flutter_localizations: ...
```


### Table of Contents
1. [How to build & run](#build&run)
2. [Configuration](#configuration)
	* [Dart](#configuration-dart)
	* [iOS](#configuration-ios)
	* [Android](#configuration-android)
	* [Firebase](#configuration-firebase)
3. [Localization](#localization)
4. [Project structure](#structure)
5. [Disabled features](#disabled-features)
	* [Google Pay](#disabled-features-googlepay) 
	* [FB login](#disabled-features-fb) 

## How to build & run <a name="build&run"></a>

The minimum required set of commands

```bash
- flutter pub get # to get dependencies
- flutter pub run intl_utils:generate # to generate l10n.dart file from .arb localization files 
- flutter pub run build_runner build --delete-conflicting-outputs # to generate JsonSerializable models
```

To build and run the project, you can use `Android Studio` or `Visual Studio Code` with Flutter/Dart plugins (`Flutter Intl`, `Dart`, `Flutter`), then the commands above will be executed automatically when building and running.


To build from console:

```bash
 flutter build ios --release --no-codesign --flavor dev
 # or
 flutter build android --release --flavor dev
```

You can assemble the flutter part separately and run the application from native projects

To run from console:

```bash
flutter run ios -t lib/main.dart --no-codesign --flavor dev
# or
flutter run android -t lib/main.dart --flavor dev
```

``lib/main.dart`` - this is the main launch point of the project

``--flavor dev `` - this is the environment specification for dart + `flavor` in Android and `scheme` in iOS native projects


More about configuration: [Configuration change](#configuration)


## Configuration <a name="configuration"></a>

The `flavor` flag is used to switch environments. Available in the project: dev, stage, live.

Due to the specifics of assembling flutter applications, the project configuration is split into 3 parts: [Dart](#configuration-dart) / [iOS](#configuration-ios) / [Android](#configuration-android).

### Dart <a name="configuration-dart"></a>

`lib/app_config.dart` - the main class responsible for the Flutter/Dart configuration of the application part. It lists the main variables used in the project and loads the variables depending on the selected `flavor` from the `.env `files.

* `--flavor dev` -> `.env`
* `--flavor stage` -> `.env.stage`
* `--flavor live` -> `.env.live`


**Important**: Variables needed in the native part, set in native ways

### iOS <a name="configuration-ios"></a>

iOS uses `*.xcconfig` files, they are located in the folder: `ios/Flutter`

`*.xcconfig` files are used depending on the `scheme` selected inside the iOS project.


<img src="docs/image/flavor_ios.png" alt="flavor_ios" style="zoom: 67%;" />

When building a project from flutter, the `flavor` flag matches the `schema` within the iOS project.

* `--flavor dev` -> `Config-Dev.xcconfig`
* `--flavor stage` -> `Config-Stage.xcconfig `
* `--flavor live` -> `Config-Live.xcconfig `


For iOS, the minimum set of parameters in `*.xcconfig` is:

```
PRODUCT_BUNDLE_IDENTIFIER   = ""
APPLE_PAY_MERCHANT_ID       = ""
FACEBOOK_APP_ID             = ""
FACEBOOK_URL_SCHEMES        = ""
FACEBOOK_DISPLAY_NAME       = ""
GMS_API_KEY                 = ""
GOOGLE_URL_SCHEMES          = ""
```


We should also mention the signature of iOS applications. Assembly signature parameters are specified on the native side, and it is possible for each individual `scheme` to specify its own signature option.

<img src="docs/image/signing_ios.png" alt="signing_ios" style="zoom:67%;" />


### Android <a name="configuration-android"></a>

Android uses `productFlavors`, they are in the folder: `android/app/build.gradle`

<img src="docs/image/flavor_android.png" alt="flavor_android" style="zoom:67%;" />


For Android, the minimum set of parameters:

```
resValue "string", "gms_api_key", ""
resValue "string", "facebook_app_id", ""
resValue "string", "fb_login_protocol_scheme", ""
```

### Firebase <a name="configuration-firebase"></a>

The project uses Firebase Console ([console.firebase.google.com](https://console.firebase.google.com)):

* Firebase Crashlytics - to collect error data
* Firebase Distribution - for assembly and testing in-house

Firebase is configured in the native part via `google-services.json` (`android/app/google-services.json`) in Android and `GoogleService-Info.plist` (`ios/GoogleService-Info.plist`) in iOS.

Important: Each `flavor` uses a different file because it uses a different `applicationID`/`bundleID`.

These files are generated in the settings of the firebase project.

We recommend not to add these files to the git project because they have keys and if they appear in the public domain, firebase will ask you to replace them.


## Localization <a name="localization"></a>

Localization in the application works through the Intl library ([pub.dev:intl](https://pub.dev/packages/intl))

After changing the arb files (`lib/l10n/intl_ar.arb` and `lib/l10n/intl_en.arb`), you need to regenerate the `l10n.dart` file using the `Flutter Intl` plugin or using this command:

```bash 
flutter pub run intl_utils:generate
```

After that, it becomes possible to use localization strings through class `S`:

```dart 
S.of(context).key 
# or 
S.current.key
```

But this approach has limitations on the name of the localization keys. For example, keys must start with a letter, cannot contain spaces, or contain special characters.

The project has utilities that generate valid `*.arb` files from `ar_original.json` and `en_origirinal.json` files to get around this limitation.

```bash 
# generates `*.arb` files from `*_original.json` files
go run localization_utils/sync_original_files.go 
```

```bash 
# additional script adds missing lines to ar_original.json from intl_ar.arb
go run localization_utils/update_ar_origin.go
```

For `*_original.json` there are no restrictions on the name of the keys, and the utility replaces special characters with valid ones. In this case, the same transformation takes place in the project in order to correlate the lines that come from the backend. In the code, `translation_service.dart` (`lib/services/translation_service.dart`) is responsible for this.

Therefore, it is better to add new lines to `*_original.json` files.


## Project structure <a name="structure"></a>

<img src="docs/image/folders.png" alt="folders" style="zoom:67%;" />


|   |   |
|---|---|
|*main.dart*|The entry point to the application. Here is the initialization of Crashlytics, registration of services in Inject (service locator) and registration of common Blocs that are used throughout the application.|
|*router.dart*|Registration of all routes in applications and processing of transitions.|
|*app.dart*|Class with the declaration of the main widget of the application, registration of the localization and the general theme of the application.|
|*app_config.dart*|Application variables.|
|---|---|
|*blocs/*|All `Bloc` used in the application.|
|*core/*|Application-wide components and base class extensions.|
|*exceptions/*|Custom exceptions used in application logic.|
|*l10n/*|Localization folder.|
|*models/*|Model classes used in the application.|
|*presentation/*|All UI elements used in the application.|
|---|---|
|*presentation/screens/*|Application screens and widgets necessary for them, divided into folders. Files with `*_screen.dart` - describe application screens. The rest describe the widgets they need.|
|*presentation/values/*|Constants for UI such as colors, fonts, images, etc.|
|*presentation/widgets/*|Common UI elements.|
|---|---|



## Disabled features <a name="disabled-features"></a>

### Google Pay <a name="disabled-features-googlepay"></a>

To enable the button for Google Pay - you need to uncomment the code in `payment_cubit.dart` (`blocs/payment/payment_cubit.dart`)

<img src="docs/image/google_pay.png" alt="google_pay" style="zoom:67%;" />


### Facebook login <a name="disabled-features-fb"></a>

The FB login button is commented out in the UI. 
To enable it, uncomment the code in `presentation/screens/auth/widgets/sso_block.dart`


<img src="docs/image/fb_login.png" alt="fb_login" style="zoom:67%;" />
README.md
جارٍ عرض README.md.
