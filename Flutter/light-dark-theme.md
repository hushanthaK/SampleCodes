# Packages
  <li>provider</li>
  <li>shared_preferences</li>
  <li>get_it</li>
  
# File Structure
<pre>
  |--lib
  | |
  | |--provider
  | | |-theme_provider.dart
  | | |-splash_provider.dart
  | |
  | |--repository
  | | |-splash_repo.dart
  | |
  | |--screens
  | | |-splash_screen.dart
  | |
  | |--theme
  | | |-dark_theme.dart
  | | |-light_theme.dart
  | |
  | |-di_container.dart
  | |-main.dart
  |
</pre>

### main.dart
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'di_container.dart' as di;

import 'provider/splash_provider.dart';
import 'provider/theme_provider.dart';
import 'screens/splash/splash_screen.dart';
import 'theme/dark_theme.dart';
import 'theme/light_theme.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await di.init();
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider<SplashProvider>(
            create: (context) => di.sl<SplashProvider>()),
        ChangeNotifierProvider<ThemeProvider>(
            create: (context) => di.sl<ThemeProvider>()),
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
        theme: Provider.of<ThemeProvider>(context).darkTheme ? dark : light,
        home: const SplashScreen());
  }
}
```
### di-container.dart
```dart
import 'package:get_it/get_it.dart';
import 'package:shared_preferences/shared_preferences.dart';

import 'provider/splash_provider.dart';
import 'provider/theme_provider.dart';
import 'repository/splash_repo.dart';

final sl = GetIt.instance;

Future<void> init() async {
  // Repository
  sl.registerLazySingleton(() => SplashRepo(sharedPreferences: sl()));

  // Provider
  sl.registerFactory(() => ThemeProvider(sharedPreferences: sl()));
  sl.registerFactory(() => SplashProvider(splashRepo: sl()));

  // External
  final sharedPreferences = await SharedPreferences.getInstance();
  sl.registerLazySingleton(() => sharedPreferences);
}

```

### Theme Files
##### theme -> dark-theme.dart
```dart
import 'package:flutter/material.dart';

ThemeData dark = ThemeData(
  primaryColor: const Color(0xFFba4f41),
  brightness: Brightness.dark,
  scaffoldBackgroundColor: const Color(0xFF2C2C2C),
);
```

### Theme Files
##### theme -> light-theme.dart
```dart
import 'package:flutter/material.dart';

ThemeData light = ThemeData(
  primaryColor: const Color.fromARGB(255, 218, 53, 89),
  brightness: Brightness.light,
);
```

### Splash Screen
##### screens -> splash-screen.dart
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../../provider/splash_provider.dart';

class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  _SplashScreenState createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();

    init();
  }

  void init() {
    Provider.of<SplashProvider>(context, listen: false).initSharedData();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      body: Center(
      ),
    );
  }
}
```

### Repository
##### repository -> splash-repo.dart
```dart
import 'package:shared_preferences/shared_preferences.dart';

class SplashRepo {
  final SharedPreferences sharedPreferences;
  SplashRepo({required this.sharedPreferences});

  Future<bool> initSharedData() {
    if (!sharedPreferences.containsKey('THEME')) {
      return sharedPreferences.setBool('THEME', false);
    }
    return Future.value(true);
  }

  Future<bool> removeSharedData() {
    return sharedPreferences.clear();
  }
}
```

### Providers
##### repository -> splash-provider.dart
```dart
import 'package:flutter/foundation.dart';
import '../repository/splash_repo.dart';

class SplashProvider extends ChangeNotifier {
  final SplashRepo splashRepo;
  SplashProvider({required this.splashRepo});

  Future<bool> initSharedData() {
    return splashRepo.initSharedData();
  }

  Future<bool> removeSharedData() {
    return splashRepo.removeSharedData();
  }
}
```

### Providers
##### repository -> theme-provider.dart
```dart
import 'package:flutter/foundation.dart';
import 'package:shared_preferences/shared_preferences.dart';


class ThemeProvider with ChangeNotifier {
  final SharedPreferences sharedPreferences;
  ThemeProvider({required this.sharedPreferences}) {
    _loadCurrentTheme();
  }

  bool _darkTheme = true;
  bool get darkTheme => _darkTheme;

  void toggleTheme() async {
    _darkTheme = !_darkTheme;
    sharedPreferences.setBool('THEME', _darkTheme);
    notifyListeners();
  }

  void _loadCurrentTheme() async {
    _darkTheme = sharedPreferences.getBool('THEME') ?? false;
    notifyListeners();
  }
}
```
