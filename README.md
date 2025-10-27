
name: gesture_companion_pro
description: Combined touch + camera gesture control app.
publish_to: 'none'
version: 0.1.0

environment:
  sdk: '>=2.17.0 <3.0.0'

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  camera: ^0.11.0+2
  tflite_flutter: ^0.11.0
  flutter_tts: ^3.6.3
  provider: ^6.0.5

flutter:
  uses-material-design: true
  assets:
    - assets/models/hand_gesture.tflite
import 'package:flutter/material.dart';
import 'pages/home_page.dart';

void main() {
  runApp(const GestureCompanionPro());
}

class GestureCompanionPro extends StatelessWidget {
  const GestureCompanionPro({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Gesture Companion Pro',
      theme: ThemeData(primarySwatch: Colors.indigo),
      home: const HomePage(),
      debugShowCheckedModeBanner: false,
    );
  }
}
import 'package:flutter/material.dart';
import 'touch_gesture_page.dart';
import 'camera_gesture_page.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Gesture Companion Pro')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton.icon(
              icon: const Icon(Icons.touch_app),
              label: const Text('Touch Gestures'),
              onPressed: () {
                Navigator.push(context,
                    MaterialPageRoute(builder: (_) => const TouchGesturePage()));
              },
            ),
            const SizedBox(height: 20),
            ElevatedButton.icon(
              icon: const Icon(Icons.camera_alt),
              label: const Text('Camera Hand Gestures'),
              onPressed: () {
                Navigator.push(context,
                    MaterialPageRoute(builder: (_) => const CameraGesturePage()));
              },
            ),
          ],
        ),
      ),
    );
  }
}
import 'package:flutter/material.dart';
import '../services/tts_service.dart';

class TouchGesturePage extends StatefulWidget {
  const TouchGesturePage({super.key});

  @override
  State<TouchGesturePage> createState() => _TouchGesturePageState();
}

class _TouchGesturePageState extends State<TouchGesturePage> {
  String feedback = "Try tap, double tap, swipe, or pinch!";
  double scale = 1.0;
  Offset offset = Offset.zero;
  final tts = TTSService();

  void update(String msg) {
    setState(() => feedback = msg);
    tts.speak(msg);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("Touch Gestures")),
      body: Center(
        child: GestureDetector(
          onTap: () => update("Single tap detected"),
          onDoubleTap: () => update("Double tap detected"),
          onLongPress: () => update("Long press detected"),
          onPanUpdate: (details) =>
              setState(() => offset += details.delta),
          onScaleUpdate: (details) =>
              setState(() => scale = details.scale.clamp(0.5, 3.0)),
          child: Transform.translate(
            offset: offset,
            child: Transform.scale(
              scale: scale,
              child: Container(
                width: 200,
                height: 200,
                alignment: Alignment.center,
                decoration: BoxDecoration(
                  color: Colors.indigo,
                  borderRadius: BorderRadius.circular(20),
                ),
                child: Text(
                  feedback,
                  style: const TextStyle(color: Colors.white),
                  textAlign: TextAlign.center,
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
import 'package:flutter/material.dart';
import 'package:camera/camera.dart';
import '../services/tts_service.dart';

class CameraGesturePage extends StatefulWidget {
  const CameraGesturePage({super.key});

  @override
  State<CameraGesturePage> createState() => _CameraGesturePageState();
}

class _CameraGesturePageState extends State<CameraGesturePage> {
  CameraController? controller;
  bool initialized = false;
  String gesture = "Show your hand to the camera...";
  final tts = TTSService();

  @override
  void initState() {
    super.initState();
    initCamera();
  }

  Future<void> initCamera() async {
    final cameras = await availableCameras();
    controller = CameraController(cameras.first, ResolutionPreset.medium);
    await controller!.initialize();
    setState(() => initialized = true);
    // In a full version, connect this camera feed to TFLite model
  }

  @override
  void dispose() {
    controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("Camera Gestures")),
      body: initialized
          ? Stack(
              children: [
                CameraPreview(controller!),
                Align(
                  alignment: Alignment.bottomCenter,
                  child: Container(
                    color: Colors.black54,
                    height: 100,
                    alignment: Alignment.center,
                    child: Text(
                      gesture,
                      style: const TextStyle(
                        color: Colors.white,
                        fontSize: 18,
                      ),
                    ),
                  ),
                ),
              ],
            )
          : const Center(child: CircularProgressIndicator()),
    );
  }
}
import 'package:flutter_tts/flutter_tts.dart';

class TTSService {
  final FlutterTts _tts = FlutterTts();

  Future<void> speak(String text) async {
    await _tts.setLanguage("en-US");
    await _tts.setPitch(1.0);
    await _tts.speak(text);
  }
}
name: Flutter Android CI
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    - name: Setup Flutter
      uses: subosito/flutter-action@v2
      with:
        flutter-version: '3.22.0'
    - name: Install dependencies
      run: flutter pub get
    - name: Build APK
      run: flutter build apk --release
    - name: Upload APK
      uses: actions/upload-artifact@v3
      with:
        name: gesture_companion_pro_apk
        path: build/app/outputs/flutter-apk/app-release.apk
