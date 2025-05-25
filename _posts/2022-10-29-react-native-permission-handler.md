---
title: Handling Permissions in React Native
date: 2022-10-29 19:10:00 +0700
categories: ["Mobile Development", "React Native"]
tags: ["Mobile Development", "React Native", "Libraries"]
pin: false
---

`react-native-permissions` is a very popular library for managing permissions in the React Native world.
It provides a specific guide on the permission handling flow in the application as follows:

### iOS flow

```
   ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
   ┃ check(PERMISSIONS.IOS.CAMERA) ┃
   ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                   │
       Is the feature available
           on this device ?
                   │           ╔════╗
                   ├───────────║ NO ║──────────────┐
                   │           ╚════╝              │
                ╔═════╗                            ▼
                ║ YES ║                 ┌─────────────────────┐
                ╚═════╝                 │ RESULTS.UNAVAILABLE │
                   │                    └─────────────────────┘
           Is the permission
             requestable ?
                   │           ╔════╗
                   ├───────────║ NO ║──────────────┐
                   │           ╚════╝              │
                ╔═════╗                            ▼
                ║ YES ║                  ┌───────────────────┐
                ╚═════╝                  │ RESULTS.BLOCKED / │
                   │                     │ RESULTS.LIMITED / │
                   │                     │  RESULTS.GRANTED  │
                   ▼                     └───────────────────┘
          ┌────────────────┐
          │ RESULTS.DENIED │
          └────────────────┘
                   │
                   ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃ request(PERMISSIONS.IOS.CAMERA) ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                   │
         Does the user accept
            the request ?
                   │           ╔════╗
                   ├───────────║ NO ║──────────────┐
                   │           ╚════╝              │
                ╔═════╗                            ▼
                ║ YES ║                   ┌─────────────────┐
                ╚═════╝                   │ RESULTS.BLOCKED │
                   │                      └─────────────────┘
                   ▼
          ┌─────────────────┐
          │ RESULTS.GRANTED │
          └─────────────────┘
```

### Android flow

```
 ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
 ┃ check(PERMISSIONS.ANDROID.CAMERA) ┃
 ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                   │
       Is the feature available
           on this device ?
                   │           ╔════╗
                   ├───────────║ NO ║──────────────┐
                   │           ╚════╝              │
                ╔═════╗                            ▼
                ║ YES ║                 ┌─────────────────────┐
                ╚═════╝                 │ RESULTS.UNAVAILABLE │
                   │                    └─────────────────────┘
           Is the permission
           already granted ?
                   │           ╔═════╗
                   ├───────────║ YES ║─────────────┐
                   │           ╚═════╝             │
                ╔════╗                             ▼
                ║ NO ║                   ┌───────────────────┐
                ╚════╝                   │  RESULTS.GRANTED  │
                   │                     └───────────────────┘
                   ▼
          ┌────────────────┐
          │ RESULTS.DENIED │◀──────────────────────┐
          └────────────────┘                       │
                   │                               │
                   ▼                               │
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓         ╔═════╗
┃ request(PERMISSIONS.ANDROID.CAMERA) ┃         ║ YES ║
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛         ╚═════╝
                   │                               │
         Does the user accept                      │
            the request ?                          │
                   │           ╔════╗      Is the permission
                   ├───────────║ NO ║──── still requestable ?
                   │           ╚════╝              │
                ╔═════╗                         ╔════╗
                ║ YES ║                         ║ NO ║
                ╚═════╝                         ╚════╝
                   │                               │
                   ▼                               ▼
          ┌─────────────────┐             ┌─────────────────┐
          │ RESULTS.GRANTED │             │ RESULTS.BLOCKED │
          └─────────────────┘             └─────────────────┘
```

### Windows flow

```
   ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
   ┃ check(PERMISSIONS.WINDOWS.WEBCAM) ┃
   ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                     │
         Is the feature available
              on this device ?
                     │           ╔════╗
                     ├───────────║ NO ║──────────────┐
                     │           ╚════╝              │
                  ╔═════╗                            ▼
                  ║ YES ║                 ┌─────────────────────┐
                  ╚═════╝                 │ RESULTS.UNAVAILABLE │
                     │                    └─────────────────────┘
             Is the permission
               requestable ?
                     │           ╔════╗
                     ├───────────║ NO ║──────────────┐
                     │           ╚════╝              │
                  ╔═════╗                            ▼
                  ║ YES ║                  ┌───────────────────┐
                  ╚═════╝                  │ RESULTS.BLOCKED / │
                     │                     │  RESULTS.GRANTED  │
                     ▼                     └───────────────────┘
            ┌────────────────┐
            │ RESULTS.DENIED │
            └────────────────┘
                     │
                     ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃ request(PERMISSIONS.WINDOWS.WEBCAM) ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                     │
           Does the user accept
              the request ?
                     │           ╔════╗
                     ├───────────║ NO ║──────────────┐
                     │           ╚════╝              │
                  ╔═════╗                            ▼
                  ║ YES ║                   ┌─────────────────┐
                  ╚═════╝                   │ RESULTS.BLOCKED │
                     │                      └─────────────────┘
                     ▼
            ┌─────────────────┐
            │ RESULTS.GRANTED │
            └─────────────────┘
```

So the question is: how do we implement it in code?

Basically, we will need to handle the following cases:

- Request permission in a component
- Request permission when an event occurs (users touched the button, ...)

For each type of permission, we will need 3 methods as follows:

- `usePermission`: hook used in a component or custom hook
- `checkPermission`: check permission
- `requestPermission`: request permission

We write it as an abstract class as follows:

```typescript
export function usePermission(this: AbstractPermissionService) {
  React.useEffect(() => {
    this.checkPermission();
  }, []);
}

export abstract class AbstractPermissionService extends Service {
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

For each permission, create a separate class to handle the permission, inheriting from `AbstractPermissionService` and implementing the corresponding methods and getter.

## Example:

### Location permission

```typescript
export class GeolocationService extends AbstractGeolocationService {
  public get permission(): Permission {
    return Platform.select({
      android: PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION,
      ios: PERMISSIONS.IOS.LOCATION_WHEN_IN_USE,
    })!;
  }
}
```

### Camera permission

```typescript
export class CameraService extends AbstractGeolocationService {
  public get permission(): Permission {
    return Platform.select({
      android: PERMISSIONS.ANDROID.CAMERA,
      ios: PERMISSIONS.IOS.CAMERA,
    })!;
  }
}
```

## Usage in a component

```typescript
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

  // ...
}
```
