---
title: "Creating a Bill Scanner App on iOS"
date: 2024-05-18 00:01:00 +0700
categories: [Mobile Development, "iOS"]
tags: ["Mobile Development", "iOS"]
pin: false
---

In this technical article, we will walk through the process of creating a bill scanner app on iOS using Swift and Vision framework. This app will use the iPhone camera to capture images of bills and then read the text on those bills.

### Prerequisites

Before we start, make sure you have the following:
- Xcode installed (version 12.0 or later)
- Basic knowledge of Swift and SwiftUI
- An iOS device or simulator running iOS 14.0 or later

### Step 1: Setting Up the Project

1. Open Xcode and create a new SwiftUI project.
2. Name your project `BillScanner` and ensure you select Swift as the language and SwiftUI for the user interface.

### Step 2: Adding Permissions

To use the camera, we need to add permissions in the `Info.plist` file.

1. Open `Info.plist`.
2. Add a new key `NSCameraUsageDescription` with a value explaining why your app needs camera access, e.g., "We need access to your camera to scan bills."

### Step 3: Creating the CameraViewController

We'll create a view controller to manage the camera session and text recognition using Vision.

**CameraViewController.swift**:

```swift
import UIKit
import AVFoundation
import Vision

class CameraViewController: UIViewController, AVCaptureVideoDataOutputSampleBufferDelegate {

    var captureSession: AVCaptureSession!
    var previewLayer: AVCaptureVideoPreviewLayer!
    var captureButton: UIButton!

    override func viewDidLoad() {
        super.viewDidLoad()

        // Set up the capture session
        captureSession = AVCaptureSession()
        captureSession.sessionPreset = .high

        guard let videoCaptureDevice = AVCaptureDevice.default(for: .video) else { return }
        let videoInput: AVCaptureDeviceInput

        do {
            videoInput = try AVCaptureDeviceInput(device: videoCaptureDevice)
        } catch {
            return
        }

        if (captureSession.canAddInput(videoInput)) {
            captureSession.addInput(videoInput)
        } else {
            return
        }

        let videoOutput = AVCaptureVideoDataOutput()
        videoOutput.setSampleBufferDelegate(self, queue: DispatchQueue(label: "videoQueue"))
        captureSession.addOutput(videoOutput)

        previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.frame = view.layer.bounds
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)

        captureSession.startRunning()

        // Add Capture Button
        captureButton = UIButton(type: .system)
        captureButton.setTitle("Capture", for: .normal)
        captureButton.frame = CGRect(x: (view.frame.width - 100) / 2, y: view.frame.height - 100, width: 100, height: 50)
        captureButton.addTarget(self, action: #selector(captureText), for: .touchUpInside)
        view.addSubview(captureButton)
    }

    @objc func captureText() {
        // Handle the capture logic here
    }

    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }

        let request = VNRecognizeTextRequest { (request, error) in
            guard let observations = request.results as? [VNRecognizedTextObservation] else { return }
            let recognizedTexts = observations.compactMap { $0.topCandidates(1).first?.string }
            if !recognizedTexts.isEmpty {
                DispatchQueue.main.async {
                    self.showDetectedTextAlert(detectedTexts: recognizedTexts)
                }
            }
        }

        let requestHandler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, options: [:])
        try? requestHandler.perform([request])
    }

    func showDetectedTextAlert(detectedTexts: [String]) {
        let message = detectedTexts.joined(separator: "\n")
        let alertController = UIAlertController(title: "Detected Text", message: message, preferredStyle: .alert)
        alertController.addAction(UIAlertAction(title: "OK", style: .default, handler: { _ in
            self.captureSession.startRunning()
        }))
        present(alertController, animated: true, completion: nil)
    }
}
```

### Step 4: Integrating CameraViewController with SwiftUI

We need to create a `UIViewControllerRepresentable` to use `CameraViewController` in SwiftUI.

**CameraView.swift**:

```swift
import SwiftUI
import UIKit

struct CameraView: UIViewControllerRepresentable {

    class Coordinator: NSObject {
        var parent: CameraView

        init(parent: CameraView) {
            self.parent = parent
        }
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(parent: self)
    }

    func makeUIViewController(context: Context) -> CameraViewController {
        let cameraViewController = CameraViewController()
        return cameraViewController
    }

    func updateUIViewController(_ uiViewController: CameraViewController, context: Context) {
        // No need to update UI
    }
}
```

### Step 5: Creating the Main Interface

In `ContentView`, we will create a button that presents the camera view when pressed.

**ContentView.swift**:

```swift
import SwiftUI

struct ContentView: View {
    @State private var isPresentingCamera = false

    var body: some View {
        VStack {
            Button(action: {
                isPresentingCamera = true
            }) {
                Text("Scan Bill")
                    .padding()
                    .background(Color.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
            .fullScreenCover(isPresented: $isPresentingCamera) {
                CameraView()
                    .edgesIgnoringSafeArea(.all)
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

### Step 6: Connecting the App Entry Point

Finally, we need to connect our `ContentView` to the app entry point.

**CCMApp.swift**:

```swift
import SwiftUI

@main
struct CCMApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Conclusion

In this tutorial, we have created a bill scanner app on iOS using Swift and Vision framework. The app uses the camera to capture images and read text on the bills, displaying the detected text to the user. With these steps, you can expand the app's functionality to suit your specific needs, such as saving scanned data or further processing the text. Happy coding!
