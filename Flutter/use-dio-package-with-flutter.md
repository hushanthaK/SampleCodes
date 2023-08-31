# Packages
  <li>dio</li>
  <li>shared_preferences</li>
  <li>get_it</li>

# File Structure
<pre>
  |--lib
  | |
  | |--dioHttpClient
  | | |--dio
  | | | |
  | | | |-dio_client.dart
  | | | |-dio_logging.dart
  | | |
  | | |--exception
  | | | |
  | | | |-dio_exception.dart
  | | | 
  | | |--response
  | | | |
  | | | |-api_response.dart
  | | | |-error_response.dart
  | | |
  | | |--model
  | | | |
  | | | |-response_model.dart
  | | | 
  | | 
  | |-di_container.dart
  | |-main.dart
  |
</pre>

### main.dart
```dart
import 'package:flutter/material.dart';

import 'di_container.dart' as di;

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await di.init();
  runApp(
    MultiProvider(
      providers: [
        // add providers
      ],
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  static final navigatorKey = GlobalKey<NavigatorState>();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        title: 'Flutter Demo',
        home: const SplashScreen());
  }
}
```

### di-container.dart
```dart
import 'package:get_it/get_it.dart';
import 'package:shared_preferences/shared_preferences.dart';

final sl = GetIt.instance;

Future<void> init() async {
  sl.registerLazySingleton(() => DioClient(AppConstants.BASE_URL, sl(),
      loggingInterceptor: sl(), sharedPreferences: sl()));

  // Repository

  // Provider

  // External
  final sharedPreferences = await SharedPreferences.getInstance();
  sl.registerLazySingleton(() => sharedPreferences);
  sl.registerLazySingleton(() => Dio());
  sl.registerLazySingleton(() => LoggingInterceptor());
}

```






