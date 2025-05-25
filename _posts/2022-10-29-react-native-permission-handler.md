---
title: "Mastering Permissions in React Native with react-native-permissions"
date: 2022-10-29 19:10:00 +0700
categories: ["Mobile Development", "React Native"]
tags: ["Mobile Development", "React Native", "Permissions", "Libraries"]
pin: false
---

# Mastering Permissions in React Native with react-native-permissions

Managing permissions is a critical aspect of mobile app development. The `react-native-permissions` library simplifies this process, providing a unified API for handling permissions across iOS and Android. This guide explores how to use the library effectively.

## Why Use react-native-permissions?

Permissions are essential for accessing device features like the camera, location, and microphone. However, handling permissions can be complex due to platform-specific differences. The `react-native-permissions` library offers:

- A consistent API for iOS and Android.
- Support for a wide range of permissions.
- Easy integration with your React Native app.

## Installation

Install the library using npm or yarn:

```bash
npm install react-native-permissions
```

Link the library (for React Native versions below 0.60):

```bash
react-native link react-native-permissions
```

For React Native 0.60 and above, autolinking is supported.

## Handling Permissions

### iOS Permissions Flow

The library provides a clear flow for handling permissions on iOS:

1. **Check Permission**: Use `check` to determine the current status of a permission.

   ```javascript
   import { check, PERMISSIONS, RESULTS } from 'react-native-permissions';

   check(PERMISSIONS.IOS.CAMERA)
     .then((result) => {
       switch (result) {
         case RESULTS.UNAVAILABLE:
           console.log('This feature is not available on this device.');
           break;
         case RESULTS.DENIED:
           console.log('The permission has not been requested or is denied.');
           break;
         case RESULTS.GRANTED:
           console.log('The permission is granted.');
           break;
         case RESULTS.BLOCKED:
           console.log('The permission is blocked.');
           break;
       }
     })
     .catch((error) => {
       console.error(error);
     });
   ```

2. **Request Permission**: If the permission is denied, request it using `request`.

   ```javascript
   import { request } from 'react-native-permissions';

   request(PERMISSIONS.IOS.CAMERA)
     .then((result) => {
       console.log(result);
     });
   ```

### Android Permissions Flow

The process is similar for Android, but you need to handle additional cases like `RESULTS.LIMITED`.

## Best Practices

- Always check the permission status before requesting it.
- Provide clear explanations to users about why a permission is needed.
- Handle edge cases, such as permissions being blocked.

## Conclusion

The `react-native-permissions` library is a powerful tool for managing permissions in React Native apps. By following the best practices outlined in this guide, you can ensure a smooth user experience while maintaining compliance with platform guidelines.

## Example: Implementing Permission Handling in Code

To implement permission handling in your code, you can create an abstract class for permission services and extend it for specific permissions. Here's how:

### Abstract Permission Service

```typescript
import { check, request, Permission, PermissionStatus } from 'react-native-permissions';

export function usePermission(this: AbstractPermissionService) {
  React.useEffect(() => {
    this.checkPermission();
  }, []);
}

export abstract class AbstractPermissionService {
  public readonly usePermission = usePermission;

  abstract get permission(): Permission;

  abstract handlePermissionUnavailable(): void | Promise<void>;

  abstract handlePermissionBlocked(): void | Promise<void>;

  abstract handlePermissionDenied(): void | Promise<void>;

  abstract handlePermissionLimited(): void | Promise<void>;

  abstract handlePermissionGranted(): void | Promise<void>;

  async requestPermission(): Promise<PermissionStatus> {
    const status = await request(this.permission);
    return this.handlePermissionStatus(status);
  }

  async checkPermission(): Promise<PermissionStatus> {
    const status = await check(this.permission);
    await this.handlePermissionStatus(status);
    return status;
  }

  private async handlePermissionStatus(status: PermissionStatus) {
    switch (status) {
      case "unavailable":
        await this.handlePermissionUnavailable();
        break;

      case "blocked":
        await this.handlePermissionBlocked();
        break;

      case "denied":
        await this.handlePermissionDenied();
        return this.requestPermission();

      case "limited":
        await this.handlePermissionLimited();
        break;

      case "granted":
      default:
        await this.handlePermissionGranted();
        break;
    }
    return status;
  }
}
```

### Location Permission Service Example

```typescript
import { Platform } from 'react-native';
import { PERMISSIONS } from 'react-native-permissions';
import { AbstractPermissionService } from './AbstractPermissionService';

export class GeolocationService extends AbstractPermissionService {
  public get permission(): Permission {
    return Platform.select({
      android: PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION,
      ios: PERMISSIONS.IOS.LOCATION_WHEN_IN_USE,
    })!;
  }

  // Implement the abstract methods for handling permission states
}
```

### Camera Permission Service Example

```typescript
import { Platform } from 'react-native';
import { PERMISSIONS } from 'react-native-permissions';
import { AbstractPermissionService } from './AbstractPermissionService';

export class CameraService extends AbstractPermissionService {
  public get permission(): Permission {
    return Platform.select({
      android: PERMISSIONS.ANDROID.CAMERA,
      ios: PERMISSIONS.IOS.CAMERA,
    })!;
  }

  // Implement the abstract methods for handling permission states
}
```

## Usage in a Component

To use the permission services in your components:

```typescript
import React from 'react';
import { View, Text, Button } from 'react-native';
import { cameraService } from './services/CameraService';
import { geolocationService } from './services/GeolocationService';

function ExampleComponent() {
  // call usePermission inside component
  cameraService.usePermission();

  const handleLocation = React.useCallback(() => {
    // check permission in effect or callback
    geolocationService.checkPermission().then((status) => {
      if (status === "granted") {
        // get position here
      }
    });
  }, []);

  return (
    <View>
      <Text>React Native Permissions Example</Text>
      <Button title="Check Location Permission" onPress={handleLocation} />
    </View>
  );
}
```

By structuring your permission handling this way, you create a scalable and maintainable codebase that adheres to best practices for React Native development.
