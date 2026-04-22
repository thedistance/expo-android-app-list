# expo-android-app-list

Expo native module for **Android** that reads metadata about **other installed apps** (package list, icons, permissions, JNI `.so` names, and selective files inside APK zips).

In this repo it lives as a **local fork** (`android-app-list/`) and is consumed by [`react-raptor`](../react-raptor) via `"expo-android-app-list": "file:../android-app-list"` so native and TypeScript changes stay in sync.

## Features

- **`getAll()`** — non-system packages the host app can see (with package visibility in mind).
- **`getPackageDetails(packageName)`** — name, version, size, install times, target SDK, system flag.
- **`getNativeLibraries(packageName)`** — distinct `.so` basenames from `nativeLibraryDir`, related paths, and **base + split APK** zip scans (no arbitrary cap on `.so` entries).
- **`getAppIcon(packageName, maxSize?)`** — base64 PNG.
- **`getPermissions(packageName)`** — `requestedPermissions` from `PackageInfo`.
- **`getFiles(packageName, paths[])`** — read small text files from APK zips by path (e.g. `assets/app.config`).
- **`hasZipEntries(packageName, paths[], exactMatch?)`** — booleans in path order; checks **base + split** APK zips.  
  - **`exactMatch === true`** — only a **full** zip entry path match (case-insensitive). Use this for Cordova/Capacitor/RN/Expo asset probes so nested paths like `node_modules/.../capacitor.config.json` do not match.  
  - **`exactMatch` omitted or `false`** — also matches entries whose path **ends with** `/<path>` (useful for .NET assembly paths under `assemblies/`).

## Installation

**Published package (external apps):**

```sh
npx expo install expo-android-app-list
```

**This monorepo (`react-raptor`):**

```json
"expo-android-app-list": "file:../android-app-list"
```

Run `npm install` from `react-raptor`; `postinstall` can build the module if `build/` is missing.

## Android configuration

### `QUERY_ALL_PACKAGES`

The module’s `AndroidManifest` declares **`android.permission.QUERY_ALL_PACKAGES`**. That broadens which installed packages are returned on recent Android versions.

It is **sensitive for Play Console**: you must declare a valid use and may need to justify it. For store builds that must avoid it, fork the module and remove the permission (expect **reduced** visibility of other apps).

### Host `queries` (recommended)

The consuming app should still declare `<queries>` intents (e.g. `MAIN`, `https` `VIEW`) so **normal** package visibility works together with your use case.

## API examples

### `getAll`

```typescript
import { ExpoAndroidAppList } from "expo-android-app-list";

const apps = await ExpoAndroidAppList.getAll();
// AndroidAppListPackage[]
```

Each item includes: `packageName`, `appName`, `versionName`, `size`, `isSystemApp`, `firstInstallTime`, `lastUpdateTime`, `targetSdkVersion`.

### `getPackageDetails`

```typescript
const details = await ExpoAndroidAppList.getPackageDetails("com.example.app");
// AndroidAppListPackage | null
```

### `getNativeLibraries`

```typescript
const libs = await ExpoAndroidAppList.getNativeLibraries("com.example.app");
// string[] — .so basenames, e.g. "libreactnative.so"
```

### `hasZipEntries`

```typescript
// Exact: only "assets/app.config" at zip root (case-insensitive)
const [hasConfig] = await ExpoAndroidAppList.hasZipEntries(
  "com.example.app",
  ["assets/app.config"],
  true,
);

// Relaxed: entry may end with "/assemblies/Microsoft.Maui.Controls.dll"
const hits = await ExpoAndroidAppList.hasZipEntries("com.example.app", probePaths, false);
```

### `getFiles`

```typescript
const files = await ExpoAndroidAppList.getFiles("com.example.app", ["assets/app.config"]);
const raw = files[0]?.content;
```

### `getAppIcon`

```typescript
import { Image } from "expo-image";

const icon = await ExpoAndroidAppList.getAppIcon("com.example.app", 256);
<Image
  source={{ uri: `data:image/png;base64,${icon}` }}
  style={{ width: 64, height: 64 }}
/>;
```

### `getPermissions`

```typescript
const permissions = await ExpoAndroidAppList.getPermissions("com.example.app");
```

## Used by React Raptor

[React Raptor](../react-raptor) uses this module to scan the device and **classify** apps (React Native / Expo, Flutter, WebAPK PWAs, Cordova, etc.). Classification logic lives in the app; this module provides **I/O** only.

## License

MIT

## Contributing

Contributions are welcome; open a PR against the upstream project or this monorepo fork as appropriate.
