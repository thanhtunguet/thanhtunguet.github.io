---
title: "Creating a Bill Scanner App on Android"
date: 2024-05-18 00:01:00 +0700
categories: ["Software Development", "Mobile"]
tags: ["Mobile Development", "Android"]
pin: false
---

In this technical article, we will walk through the process of creating a bill scanner app on Android using CameraX and ML Kit. This app will use the device's camera to capture images of bills and then read the text on those bills.

### Prerequisites

Before we start, make sure you have the following:
- Android Studio installed (version 4.1 or later)
- Basic knowledge of Java or Kotlin and Android development
- An Android device or emulator running API level 21 (Lollipop) or later

### Step 1: Setting Up the Project

1. Open Android Studio and create a new project.
2. Select "Empty Activity" and click "Next".
3. Name your project `BillScanner`, choose Java or Kotlin as the language, and click "Finish".

### Step 2: Adding Dependencies

To use CameraX and ML Kit, we need to add the necessary dependencies in your `build.gradle` file.

**app/build.gradle**:

```gradle
dependencies {
    // CameraX
    implementation "androidx.camera:camera-core:1.0.2"
    implementation "androidx.camera:camera-camera2:1.0.2"
    implementation "androidx.camera:camera-lifecycle:1.0.2"
    implementation "androidx.camera:camera-view:1.0.0-alpha28"
    
    // ML Kit Text Recognition
    implementation 'com.google.mlkit:text-recognition:16.0.0'
}
```

### Step 3: Creating the Camera and Text Recognition Activity

Create an activity that uses CameraX to preview the camera feed and ML Kit to detect text.

**MainActivity.java** (Java):

```java
package com.example.billscanner;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.util.Size;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.camera.core.CameraSelector;
import androidx.camera.core.ImageAnalysis;
import androidx.camera.core.ImageProxy;
import androidx.camera.core.Preview;
import androidx.camera.lifecycle.ProcessCameraProvider;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;
import com.google.common.util.concurrent.ListenableFuture;
import com.google.mlkit.vision.common.InputImage;
import com.google.mlkit.vision.text.Text;
import com.google.mlkit.vision.text.TextRecognition;
import com.google.mlkit.vision.text.TextRecognizer;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MainActivity extends AppCompatActivity {

    private final int REQUEST_CODE_PERMISSIONS = 1001;
    private final String[] REQUIRED_PERMISSIONS = new String[] {Manifest.permission.CAMERA};
    private ExecutorService cameraExecutor;
    private TextRecognizer textRecognizer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (allPermissionsGranted()) {
            startCamera();
        } else {
            ActivityCompat.requestPermissions(this, REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS);
        }

        cameraExecutor = Executors.newSingleThreadExecutor();
        textRecognizer = TextRecognition.getClient();

        Button captureButton = findViewById(R.id.captureButton);
        captureButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // Capture logic will be implemented here
            }
        });
    }

    private void startCamera() {
        ListenableFuture<ProcessCameraProvider> cameraProviderFuture = ProcessCameraProvider.getInstance(this);

        cameraProviderFuture.addListener(() -> {
            try {
                ProcessCameraProvider cameraProvider = cameraProviderFuture.get();
                bindPreview(cameraProvider);
            } catch (ExecutionException | InterruptedException e) {
                e.printStackTrace();
            }
        }, ContextCompat.getMainExecutor(this));
    }

    private void bindPreview(@NonNull ProcessCameraProvider cameraProvider) {
        Preview preview = new Preview.Builder()
                .build();

        CameraSelector cameraSelector = new CameraSelector.Builder()
                .requireLensFacing(CameraSelector.LENS_FACING_BACK)
                .build();

        ImageAnalysis imageAnalysis = new ImageAnalysis.Builder()
                .setTargetResolution(new Size(1280, 720))
                .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
                .build();

        imageAnalysis.setAnalyzer(cameraExecutor, new ImageAnalysis.Analyzer() {
            @Override
            public void analyze(@NonNull ImageProxy image) {
                if (image.getImage() != null) {
                    InputImage inputImage = InputImage.fromMediaImage(image.getImage(), image.getImageInfo().getRotationDegrees());
                    textRecognizer.process(inputImage)
                            .addOnSuccessListener(text -> processTextRecognitionResult(text))
                            .addOnFailureListener(Throwable::printStackTrace)
                            .addOnCompleteListener(task -> image.close());
                }
            }
        });

        cameraProvider.bindToLifecycle(this, cameraSelector, imageAnalysis, preview);

        preview.setSurfaceProvider(findViewById(R.id.viewFinder).getSurfaceProvider());
    }

    private void processTextRecognitionResult(Text texts) {
        StringBuilder detectedText = new StringBuilder();
        for (Text.TextBlock block : texts.getTextBlocks()) {
            detectedText.append(block.getText()).append("\n");
        }
        if (detectedText.length() > 0) {
            runOnUiThread(() -> {
                Toast.makeText(MainActivity.this, detectedText.toString(), Toast.LENGTH_LONG).show();
            });
        }
    }

    private boolean allPermissionsGranted() {
        for (String permission : REQUIRED_PERMISSIONS) {
            if (ContextCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
                return false;
            }
        }
        return true;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (requestCode == REQUEST_CODE_PERMISSIONS) {
            if (allPermissionsGranted()) {
                startCamera();
            } else {
                Toast.makeText(this, "Permissions not granted by the user.", Toast.LENGTH_SHORT).show();
                finish();
            }
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        cameraExecutor.shutdown();
    }
}
```

**MainActivity.kt** (Kotlin):

```kotlin
package com.example.billscanner

import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import android.util.Size
import android.widget.Button
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.camera.core.*
import androidx.camera.lifecycle.ProcessCameraProvider
import androidx.core.content.ContextCompat
import com.google.mlkit.vision.common.InputImage
import com.google.mlkit.vision.text.Text
import com.google.mlkit.vision.text.TextRecognition
import com.google.mlkit.vision.text.TextRecognizer
import java.util.concurrent.ExecutorService
import java.util.concurrent.Executors

class MainActivity : AppCompatActivity() {

    private lateinit var cameraExecutor: ExecutorService
    private lateinit var textRecognizer: TextRecognizer

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (allPermissionsGranted()) {
            startCamera()
        } else {
            requestPermissions()
        }

        cameraExecutor = Executors.newSingleThreadExecutor()
        textRecognizer = TextRecognition.getClient()

        val captureButton: Button = findViewById(R.id.captureButton)
        captureButton.setOnClickListener {
            // Capture logic will be implemented here
        }
    }

    private fun startCamera() {
        val cameraProviderFuture = ProcessCameraProvider.getInstance(this)

        cameraProviderFuture.addListener({
            val cameraProvider = cameraProviderFuture.get()
            bindPreview(cameraProvider)
        }, ContextCompat.getMainExecutor(this))
    }

    private fun bindPreview(cameraProvider: ProcessCameraProvider) {
        val preview = Preview.Builder().build()

        val cameraSelector = CameraSelector.Builder()
            .requireLensFacing(CameraSelector.LENS_FACING_BACK)
            .build()

        val imageAnalysis = ImageAnalysis.Builder()
            .setTargetResolution(Size(1280, 720))
            .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
            .build()

        imageAnalysis.setAnalyzer(cameraExecutor, { image ->
            if (image.image != null) {
                val inputImage = InputImage.fromMediaImage(image.image!!, image.imageInfo.rotationDegrees)
                textRecognizer.process(inputImage)
                    .addOnSuccessListener { text -> processTextRecognitionResult(text) }
                    .addOnFailureListener { it.printStackTrace() }
                    .addOnCompleteListener { image.close() }
            }
        })

        cameraProvider.bindToLifecycle(this, cameraSelector, imageAnalysis, preview)

        preview.setSurfaceProvider(findViewById<PreviewView>(R.id.viewFinder).surfaceProvider)
    }

    private fun processTextRecognitionResult(texts: Text) {
        val detectedText = StringBuilder()
        for (block in texts.textBlocks) {
            detectedText.append(block.text).append("\n")
        }
        if (detectedText.isNotEmpty()) {
            runOnUiThread {
                Toast.makeText(this, detectedText.toString(), Toast.LENGTH_LONG).show()
            }
        }
    }

    private fun allPermissionsGranted() = REQUIRED_PERMISSIONS.all {
        ContextCompat.checkSelfPermission(baseContext, it) == PackageManager.PERMISSION_GRANTED
    }

    private fun requestPermissions() {
        registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { permissions ->
            if (permissions.all { it.value }) {
                startCamera()
            } else {
                Toast.makeText(this, "Permissions not granted by the user.", Toast.LENGTH_SHORT).show()
                finish()
            }
        }.launch(REQUIRED_PERMISSIONS)
    }

    override fun onDestroy() {
        super.onDestroy()
        cameraExecutor.shutdown()
    }

    companion object {
        private val REQUIRED_PERMISSIONS = arrayOf(Manifest.permission.CAMERA)
    }
}
```

### Step 4: Creating the Layout

Create a layout file that includes a `PreviewView` for the camera preview and a "Capture" button.

**res/layout/activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.camera.view.PreviewView
        android:id="@+id/viewFinder"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@id/captureButton" />

    <Button
        android:id="@+id/captureButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_alignParentBottom="true"
        android:text="Capture" />

</RelativeLayout>
```

### Conclusion

In this tutorial, we have created a bill scanner app on Android using CameraX and ML Kit. The app uses the device's camera to capture images of bills and then reads the text on those bills, displaying the detected text to the user. With these steps, you can expand the app's functionality to suit your specific needs, such as saving scanned data or further processing the text. Happy coding!
