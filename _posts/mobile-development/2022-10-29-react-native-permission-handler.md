---
title: Xử lý Permission trong React Native
date: 2022-10-29 19:10:00 +0700
categories: ["Mobile Development", "React Native"]
tags: ["Mobile Development", "React Native", "Libraries"]
pin: false
---

`react-native-permissions` là thư viện quản lý permission rất phổ biến trong thế giới React Native.
Nó cung cấp một hướng dẫn cụ thể về luồng xử lý permission trong ứng dụng như sau:

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

Vậy câu hỏi là: chúng ta implement nó trong code như thế nào?

Về cơ bản, chúng ta sẽ cần xử lý các trường hợp sau:

- Yêu cầu permission trong component
- Yêu cầu permission khi có sự kiện xảy ra (users touched the button, ...)

Với mỗi loại permission, ta sẽ cần 3 method như sau:

- `usePermission`: hook sử dụng trong component hoặc custom hook
- `checkPermission`: kiểm tra permission
- `requestPermission`: yêu cầu permission

Ta viết thành abstract class như sau:

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

Đối với mỗi permission, tạo một class xử lý permission riêng, kế thừa từ `AbstractPermissionService` và implement các method và getter tương ứng.

## Ví dụ:

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

## Sử dụng trong component

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
