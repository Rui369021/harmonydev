---
name: harmonyos-dev
description: HarmonyOS NEXT (鸿蒙) native application development guide. Covers ArkTS language, ArkUI declarative UI, Stage model, state management, data persistence, distributed capabilities, performance optimization, and build/debug. Read this before HarmonyOS application development.
description_zh: 鸿蒙 HarmonyOS NEXT 原生应用开发指南。涵盖 ArkTS 语言、ArkUI 声明式 UI、Stage 模型、状态管理、数据持久化、分布式能力、性能优化与构建调试。
description_en: HarmonyOS NEXT native application development guide. Covers ArkTS, ArkUI, Stage model, state management, data persistence, distributed capabilities, performance, and build/debug.
version: 1.0.0
metadata:
  version: 1.0.0
  category: mobile
  sources:
  - HarmonyOS Official Documentation (developer.huawei.com)
  - ArkTS Language Specification
  - ArkUI Component Reference
  - HarmonyOS Design Guidelines
  - DevEco Studio Documentation
license: MIT
---

## Trigger

Read this skill when the user asks to develop, debug, build, or design HarmonyOS (鸿蒙) applications. Trigger keywords include: HarmonyOS, 鸿蒙, ArkTS, ArkUI, DevEco Studio, Stage 模型, .ets 文件, oh-package.json5, module.json5, OHOS API, HarmonyOS NEXT, 纯血鸿蒙.

## 1. Technology Stack Overview (2026)

HarmonyOS NEXT (纯血鸿蒙) has completely removed the AOSP compatibility layer. It is a pure native OS based on a microkernel architecture.

| Layer | Technology | Description | Priority |
|-------|-----------|-------------|----------|
| **Language** | ArkTS | TypeScript superset with declarative decorators; compiled to Ark bytecode via ArkCompiler | ★★★★★ |
| **UI Framework** | ArkUI | Declarative + reactive + cross-device (phone/tablet/watch/PC) | ★★★★★ |
| **App Model** | Stage Model | UIAbility + ExtensionAbility; API 9+ recommended model | ★★★★★ |
| **State Mgmt** | @State / @Prop / @Link / @Provide / @Consume / AppStorage | Local -> parent-child -> bidirectional -> global | ★★★★★ |
| **Persistence** | Preferences / RelationalStore / CloudDB | KV store / SQLite / distributed cloud DB | ★★★★☆ |
| **Distributed** | Distributed Soft Bus / continueAbility | Cross-device migration, collaboration, flow | ★★★★☆ |
| **Performance** | LazyForEach / animateTo / render optimization | Large lists / complex animations / low power | ★★★★☆ |
| **AI** | HiAI / Foundation Models | On-device LLM (text generation, image understanding) | ★★★☆☆ |
| **Cross-platform** | ArkUI-X | One codebase for HarmonyOS + Android/iOS (experimental) | ★★★☆☆ |
| **IDE** | DevEco Studio (latest: 26.0 Beta1 / API 26) | Based on IntelliJ IDEA; built-in SDK manager, emulator, Previewer | ★★★★★ |
| **Pkg Manager** | OHPM | HarmonyOS official package manager (similar to npm) | ★★★★☆ |
| **Cloud** | AGC (AppGallery Connect) | Cloud DB, cloud functions, auth, push | ★★★☆☆ |
| **Build Tool** | Hvigor | Build system integrated with DevEco Studio | ★★★★☆ |

**Current versions**: ArkTS 3.0+, DevEco Studio 26.0 Beta1 (HarmonyOS 7.0 / API 26). For new projects, always use the latest stable DevEco Studio.

---

## 2. Project Structure (Stage Model)

Standard project layout created by DevEco Studio:

```
MyApplication/
├── AppScope/                          # App-level global config
│   ├── app.json5                      # bundleName, versionCode, versionName, icon, label
│   └── resources/                     # Global shared resources (icons, etc.)
│       └── base/element/
│           └── string.json            # Global strings (app_name defined here ONLY)
├── entry/                             # Main entry module (HAP)
│   ├── src/main/
│   │   ├── ets/
│   │   │   ├── entryability/
│   │   │   │   └── EntryAbility.ets   # Ability lifecycle entry
│   │   │   ├── entrybackupability/    # Backup/restore extension (optional)
│   │   │   │   └── EntryBackupAbility.ets
│   │   │   └── pages/
│   │   │       ├── Index.ets          # Main page (@Entry component)
│   │   │       └── Profile.ets        # Other pages
│   │   ├── resources/
│   │   │   ├── base/
│   │   │   │   ├── element/           # Strings, colors, floats
│   │   │   │   ├── media/             # Images, icons
│   │   │   │   └── profile/
│   │   │   │       └── main_pages.json  # Page routing config
│   │   │   ├── dark/                  # Dark mode resources
│   │   │   └── rawfile/               # Raw files (not compiled)
│   │   └── module.json5               # Module config (abilities, permissions, etc.)
│   ├── build-profile.json5            # Module build config
│   ├── hvigorfile.ts                  # Module build script
│   ├── obfuscation-rules.txt          # Obfuscation rules (Release mode)
│   └── oh-package.json5               # Module dependencies
├── build-profile.json5                # Project-level build config (signing, products)
├── hvigorfile.ts                      # Project build script
├── oh-package.json5                   # Project-level dependencies
└── oh_modules/                        # Installed dependencies (like node_modules)
```

### 2.1 Key Config Files

**AppScope/app.json5** (app-level):
```json5
{
  "app": {
    "bundleName": "com.example.myapp",
    "vendor": "ExampleCorp",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "icon": "$media:app_icon",
    "label": "$string:app_name"
  }
}
```

> **IMPORTANT**: `app_name` must be defined in `AppScope/resources/base/element/string.json` ONLY. Do NOT redefine it in `entry` module — it causes a compile conflict.

**entry/src/main/module.json5** (module-level):
```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": ["phone", "tablet", "2in1"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntrance": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "icon": "$media:app_icon",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:app_icon",
        "startWindowBackground": "$color:start_window_background",
        "exported": true,
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ]
      }
    ],
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET",
        "reason": "$string:reason_internet",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

**resources/base/profile/main_pages.json** (page routing):
```json
{
  "src": [
    "pages/Index",
    "pages/Profile",
    "pages/Settings"
  ]
}
```

**oh-package.json5** (dependencies):
```json5
{
  "name": "my-application",
  "version": "1.0.0",
  "description": "My HarmonyOS Application",
  "main": "",
  "author": "",
  "license": "Apache-2.0",
  "dependencies": {
    "@ohos/hypium": "1.0.21"
  }
}
```

See [Project Structure & Config](references/project-structure.md) for full details.

---

## 3. ArkTS Language Essentials

ArkTS is a superset of TypeScript with declarative UI extensions. It compiles to Ark bytecode via ArkCompiler.

### 3.1 Component Declaration

```typescript
// Basic component
@Component
struct MyComponent {
  @State message: string = 'Hello';

  build() {
    Text(this.message)
      .fontSize(20)
  }
}

// Entry component (page root)
@Entry
@Component
struct Index {
  build() {
    Column() {
      MyComponent()
    }
  }
}
```

### 3.2 Key Decorators

| Decorator | Scope | Purpose |
|-----------|-------|---------|
| `@Entry` | Page-level struct | Marks the root component of a page |
| `@Component` | struct | Declares a reusable UI component |
| `@State` | Local state | Triggers UI refresh when changed |
| `@Prop` | Parent -> child | One-way data binding (parent to child) |
| `@Link` | Parent <-> child | Two-way data binding (bidirectional) |
| `@Provide` | Ancestor | Provide data to descendants |
| `@Consume` | Descendant | Consume data from ancestor |
| `@Builder` | Function | Reusable UI builder function |
| `@BuilderParam` | Component param | Pass builder functions as component params |
| `@Extend` | Built-in component | Extend built-in component styles |
| `@Styles` | Function | Reusable style definitions |
| `@Observed` | Class | Makes a class observable for deep observation |
| `@Track` | Class field | Field-level change tracking with @Observed |
| `@StorageLink` | AppStorage | Bidirectional binding to AppStorage |
| `@StorageProp` | AppStorage | One-way binding from AppStorage |
| `@LocalStorageLink` | LocalStorage | Bidirectional binding to LocalStorage |

### 3.3 Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Struct/Component | PascalCase | `UserCard`, `IndexPage` |
| Function/Variable | camelCase | `getUserData()`, `isLoading` |
| Constant | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |
| Builder function | camelCase | `@Builder buildHeader()` |
| Resource reference | `$type:name` | `$r('app.string.title')`, `$media:icon` |

### 3.4 Resource Access

```typescript
// String resource
Text($r('app.string.hello'))
  .fontSize($r('app.float.title_size'))

// Media resource
Image($r('app.media.background'))

// Color resource
.backgroundColor($r('app.color.primary'))

// System resource (built-in)
Text($r('sys.string.ok'))
  .fontColor($r('sys.color.ohos_id_color_foreground'))

// Raw file
Image($rawfile('images/logo.png'))
```

### 3.5 Common ArkTS Pitfalls

```typescript
// WRONG: Direct array mutation does NOT trigger @State refresh
@State items: string[] = ['a', 'b'];
addBad() {
  this.items.push('c');  // UI will NOT update!
}

// CORRECT: Reassign to trigger refresh
addGood() {
  this.items = [...this.items, 'c'];  // UI updates
}

// WRONG: Object property mutation does NOT trigger @State
@State user: User = { name: 'Alice', age: 20 };
updateBad() {
  this.user.name = 'Bob';  // UI may NOT update!
}

// CORRECT: Reassign the object
updateGood() {
  this.user = { ...this.user, name: 'Bob' };  // UI updates
}

// WRONG: Complex computation inside build()
build() {
  Text(this.expensiveCompute(this.data))  // Runs every re-render!
}

// CORRECT: Pre-compute in lifecycle or @State
aboutToAppear() {
  this.computedResult = this.expensiveCompute(this.data);
}
build() {
  Text(this.computedResult)
}
```

See [ArkTS Language Guide](references/arkts-guide.md) for full syntax reference.

---

## 4. ArkUI Declarative UI

### 4.1 Layout Components

```typescript
// Column (vertical stack)
Column({ space: 16 }) {
  Text('Title').fontSize(24)
  Text('Subtitle').fontSize(16)
}
.width('100%')
.alignItems(HorizontalAlign.Center)

// Row (horizontal stack)
Row({ space: 12 }) {
  Image($r('app.media.avatar')).width(48).height(48)
  Text('User Name').fontSize(18)
}

// Flex (flexible layout)
Flex({ direction: FlexDirection.Row, justifyContent: FlexAlign.SpaceBetween }) {
  Text('Left')
  Text('Right')
}

// Stack (overlay)
Stack({ alignContent: Alignment.Center }) {
  Image($r('app.media.bg')).width('100%').height('100%')
  Text('Overlay Text').fontSize(20)
}

// Grid
Grid() {
  GridItem() { Text('1') }
  GridItem() { Text('2') }
  GridItem() { Text('3') }
}
.columnsTemplate('1fr 1fr 1fr')
.rowsTemplate('1fr 1fr')
```

### 4.2 Common UI Components

```typescript
// Button
Button('Submit', { type: ButtonType.Capsule })
  .width('100%')
  .height(48)
  .backgroundColor($r('app.color.primary'))
  .onClick(() => { /* handle */ })

// TextInput
TextInput({ placeholder: 'Enter your name' })
  .width('100%')
  .height(48)
  .onChange((value: string) => { this.name = value; })

// Image
Image($r('app.media.photo'))
  .width(200)
  .height(200)
  .objectFit(ImageFit.Cover)
  .borderRadius(12)

// List with LazyForEach (performance critical for large lists)
List({ space: 8 }) {
  LazyForEach(this.dataSource, (item: Item) => {
    ListItem() {
      Row() {
        Text(item.title).fontSize(16)
        Text(item.subtitle).fontSize(14)
      }
    }
  }, (item: Item) => item.id.toString())
}
```

### 4.3 Navigation

```typescript
// Router navigation (page-based)
import router from '@ohos.router';

// Navigate to page
router.pushUrl({ url: 'pages/Detail' });

// Navigate with params
router.pushUrl({
  url: 'pages/Detail',
  params: { id: 42, name: 'test' }
});

// Navigate and replace (no back stack)
router.replaceUrl({ url: 'pages/Home' });

// Go back
router.back();

// Get params on target page
let params = router.getParams() as Record<string, Object>;
let id = params['id'] as number;

// Navigation with transitions
router.pushUrl({ url: 'pages/Detail' })
  .then(() => { console.info('Navigation success'); })
  .catch((err: Error) => { console.error(err.message); });
```

### 4.4 Builder Functions

```typescript
// Define reusable UI fragments
@Builder
function HeaderItem(title: string, subtitle: string) {
  Row() {
    Text(title).fontSize(20).fontWeight(FontWeight.Bold)
    Text(subtitle).fontSize(14).fontColor('#999')
  }
  .width('100%')
  .padding(16)
}

// Use in build()
build() {
  Column() {
    HeaderItem('Settings', 'Manage your preferences')
    HeaderItem('About', 'App version and info')
  }
}
```

See [ArkUI Components Guide](references/arkui-components.md) for full component reference.

---

## 5. State Management

### 5.1 State Management Hierarchy

```
@State (local)     -> @Prop (one-way parent->child) -> @Link (two-way parent<->child)
                                                            |
@Provide (ancestor) -> @Consume (descendant)                 |
                                                            v
AppStorage (global) <- @StorageLink/@StorageProp     LocalStorage (page-level)
```

### 5.2 Practical Patterns

```typescript
// Parent-child one-way (@Prop)
@Component
struct ChildComponent {
  @Prop title: string;  // Receives from parent, local copy

  build() {
    Text(this.title).fontSize(20)
  }
}

// Parent-child two-way (@Link)
@Component
struct Counter {
  @Link count: number;  // Bidirectional binding

  build() {
    Button(`Count: ${this.count}`)
      .onClick(() => { this.count++; })  // Updates parent too!
  }
}

@Entry
@Component
struct Parent {
  @State count: number = 0;
  @State title: string = 'My Title';

  build() {
    Column() {
      ChildComponent({ title: this.title })     // @Prop
      Counter({ count: $count })                // @Link (note the $ prefix!)
    }
  }
}

// Global state with AppStorage
@Entry
@Component
struct App {
  @StorageLink('currentUser') user: User = { name: '', age: 0 };

  aboutToAppear() {
    // Initialize global state
    AppStorage.setOrCreate('currentUser', { name: 'Alice', age: 25 });
  }

  build() {
    Column() {
      Text(`Hello, ${this.user.name}`)
      Button('Change Name')
        .onClick(() => {
          this.user = { name: 'Bob', age: 30 };  // Updates AppStorage globally
        })
    }
  }
}

// Ancestor-descendant with @Provide/@Consume
@Entry
@Component
struct GrandParent {
  @Provide('theme') theme: string = 'light';

  build() {
    Column() {
      ParentComponent()
    }
  }
}

@Component
struct DeepChild {
  @Consume('theme') theme: string;  // Gets value from any ancestor

  build() {
    Text(this.theme)  // 'light'
  }
}
```

> **IMPORTANT**: When passing `@Link` from parent to child, use the `$` prefix: `Child({ count: $count })`. Forgetting the `$` is a very common bug.

See [State Management](references/state-management.md) for advanced patterns.

---

## 6. Lifecycle

### 6.1 UIAbility Lifecycle

```
onCreate -> onWindowStageCreate -> onForeground -> [running] -> onBackground -> onWindowStageDestroy -> onDestroy
                                         |                          |
                                    (windowStage)              (may return to foreground)
```

```typescript
import UIAbility from '@ohos.app.ability.UIAbility';
import window from '@ohos.window';

export default class EntryAbility extends UIAbility {
  onCreate(want, launchParam) {
    console.info('Ability created');
    // Initialize data, parse launch parameters
  }

  onDestroy() {
    console.info('Ability destroyed');
    // Clean up resources
  }

  onWindowStageCreate(windowStage: window.WindowStage) {
    console.info('Window stage created');
    // Load the entry page
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        console.error(`Failed to load content: ${err.message}`);
        return;
      }
      console.info('Content loaded successfully');
    });
  }

  onWindowStageDestroy() {
    console.info('Window stage destroyed');
    // Release UI resources
  }

  onForeground() {
    console.info('Ability foregrounded');
    // Resume tasks, refresh data
  }

  onBackground() {
    console.info('Ability backgrounded');
    // Pause tasks, save state
  }
}
```

### 6.2 Component Lifecycle

```typescript
@Component
struct MyComponent {
  // Before the component is about to appear (before build)
  aboutToAppear() {
    // Fetch data, init state
  }

  // After the component appears (after first render)
  aboutToDisappear() {
    // Clean up timers, listeners
  }

  // When page is shown (only for @Entry components)
  onPageShow() {
    // Refresh data when returning to page
  }

  // When page is hidden
  onPageHide() {
    // Pause animations, save draft
  }

  build() { /* ... */ }
}
```

---

## 7. Data Persistence

### 7.1 Preferences (Key-Value Store)

```typescript
import preferences from '@ohos.data.preferences';

// Write
async function saveData(context: Context, key: string, value: string) {
  const pref = await preferences.getPreferences(context, { name: 'myAppPrefs' });
  await pref.put(key, value);
  await pref.flush();  // Must call flush to persist!
}

// Read
async function readData(context: Context, key: string): Promise<string> {
  const pref = await preferences.getPreferences(context, { name: 'myAppPrefs' });
  return await pref.get(key, '') as string;
}

// Delete
async function deleteData(context: Context, key: string) {
  const pref = await preferences.getPreferences(context, { name: 'myAppPrefs' });
  await pref.delete(key);
  await pref.flush();
}
```

### 7.2 RelationalStore (SQLite)

```typescript
import relationalStore from '@ohos.data.relationalStore';

async function initDB(context: Context) {
  const config: relationalStore.StoreConfig = {
    name: 'MyDatabase.db',
    securityLevel: relationalStore.SecurityLevel.S1,
  };

  const store = await relationalStore.getRdbStore(context, config);

  // Create table
  await store.executeSql(
    `CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      age INTEGER,
      created_at INTEGER
    )`
  );

  // Insert
  const values = {
    name: 'Alice',
    age: 25,
    created_at: Date.now(),
  };
  await store.insert('users', values);

  // Query
  const resultSet = await store.query(
    'SELECT * FROM users WHERE age > ?',
    [20]
  );
  while (resultSet.goToNextRow()) {
    const id = resultSet.getDouble(resultSet.getColumnIndex('id'));
    const name = resultSet.getString(resultSet.getColumnIndex('name'));
    console.info(`User: ${id}, ${name}`);
  }
  resultSet.close();
}
```

---

## 8. Network Requests

```typescript
import http from '@ohos.net.http';

// GET request
async function fetchUserData(url: string): Promise<string> {
  const httpRequest = http.createHttp();
  try {
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.GET,
      header: { 'Content-Type': 'application/json' },
      expectDataType: http.HttpDataType.STRING,
      connectTimeout: 60000,
      readTimeout: 60000,
    });
    if (response.responseCode === 200) {
      return response.result as string;
    }
    throw new Error(`HTTP ${response.responseCode}`);
  } finally {
    httpRequest.destroy();  // Must destroy to release resources!
  }
}

// POST request
async function postUserData(url: string, data: object): Promise<string> {
  const httpRequest = http.createHttp();
  try {
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.POST,
      header: { 'Content-Type': 'application/json' },
      extraData: JSON.stringify(data),
      expectDataType: http.HttpDataType.STRING,
    });
    return response.result as string;
  } finally {
    httpRequest.destroy();
  }
}
```

> **IMPORTANT**: Always call `httpRequest.destroy()` in a `finally` block. Forgetting this causes resource leaks and eventual crashes.

> Network permission must be declared in `module.json5` under `requestPermissions` with `ohos.permission.INTERNET`.

---

## 9. Build & Debug Commands

### 9.1 Command-Line Build (Hvigor)

```bash
# Full build (assemble HAP)
hvigorw assembleApp

# Build with specific product
hvigorw assembleApp -p product=default

# Clean build
hvigorw clean

# Sync dependencies
hvigorw --sync -p product=default --parallel --incremental

# Build with debug configuration
hvigorw assembleHap --mode module -p product=default -p buildMode=debug

# Build with release configuration
hvigorw assembleHap --mode module -p product=default -p buildMode=release
```

### 9.2 OHPM Commands

```bash
# Install dependencies
ohpm install

# Install specific package
ohpm install @ohos/xxx

# Install dev dependency
ohpm install --dev @ohos/hypium

# Update dependencies
ohpm update

# List installed packages
ohpm list
```

### 9.3 Debugging

```typescript
// Use hilog for logging (NOT console.log in production)
import hilog from '@ohos.hilog';

const DOMAIN = 0x0001;
const TAG = 'MyApp';

hilog.info(DOMAIN, TAG, 'App started, version: %{public}s', '1.0.0');
hilog.warn(DOMAIN, TAG, 'Cache miss, falling back to network');
hilog.error(DOMAIN, TAG, 'Request failed: %{public}s', error.message);
```

| Log Level | Function | Use Case |
|-----------|----------|----------|
| Debug | `hilog.debug()` | Development debugging only |
| Info | `hilog.info()` | Key checkpoints, normal flow |
| Warn | `hilog.warn()` | Abnormal but recoverable |
| Error | `hilog.error()` | Failures, caught exceptions |
| Fatal | `hilog.fatal()` | Unrecoverable errors |

---

## 10. Common Pitfalls & Solutions

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| @State not updating | UI doesn't refresh after change | Reassign value: `this.xxx = newValue` (not `.push()`) |
| @Link missing `$` prefix | Compile error or no two-way binding | Use `Child({ prop: $stateVar })` not `Child({ prop: stateVar })` |
| LazyForEach flickering | List stutters/jitters during scroll | Add key generator: `LazyForEach(data, item => {...}, item => item.id)` |
| Network resource leak | App crashes after many requests | Call `httpRequest.destroy()` in `finally` block |
| Preferences not saving | Data lost after restart | Call `await pref.flush()` after `pref.put()` |
| Distributed migration fails | Cannot flow to other device | Set `distributed: true` and `continueOn: true` in module.json5 |
| Resource not found | Build error: resource not found | Check resource path: `$r('app.type.name')` matches `resources/base/type/name.json` |
| Duplicate app_name | Compile conflict | Define `app_name` in AppScope ONLY, not in entry module |
| Large list OOM | Memory crash on scrolling | Use `LazyForEach` instead of `ForEach` for lists > 100 items |
| build() re-renders slow | UI lag during interaction | Move expensive computation out of `build()` to `aboutToAppear()` |
| Obfuscation breaks code | Release build crashes | Add keep rules in `obfuscation-rules.txt` for reflection-based code |

---

## 11. Performance Optimization Top 5

1. **Large lists**: Always use `LazyForEach` with a key generator (never `ForEach` for > 100 items)
2. **Animations**: Use `transition` + `.animateTo()` instead of manual frame updates
3. **build() optimization**: Move complex logic to `aboutToAppear()` or computed `@State` — never compute in `build()`
4. **Images**: Use `.objectFit(ImageFit.Contain)` + enable image caching; preload critical images
5. **ArkCompiler**: Enable AOT optimization in build-profile.json5 (default in DevEco)

```typescript
// Animation example
@State scale: number = 1;

build() {
  Image($r('app.media.photo'))
    .scale({ x: this.scale, y: this.scale })
    .animation({ duration: 300, curve: Curve.EaseInOut })
    .onClick(() => {
      this.scale = this.scale === 1 ? 1.2 : 1;
    })
}
```

---

## 12. HarmonyOS UI Design Guidelines

### 12.1 Design Principles

| Principle | Description |
|-----------|-------------|
| **Meta-design** | Consistent visual language across all HarmonyOS devices |
| **Adaptive** | Responsive layouts for phone/tablet/watch/PC/2-in-1 |
| **Immersive** | Full-screen experiences with minimal chrome |
| **Natural** | Physics-based animations and gesture interactions |

### 12.2 Key Specifications

| Spec | Value |
|------|-------|
| Grid unit | 8vp |
| Touch target min | 48vp x 48vp |
| Primary touch target | 56vp |
| Corner radius (cards) | 12vp - 16vp |
| Corner radius (buttons) | 8vp (default) / full (Capsule) |
| Page padding | 16vp - 24vp |
| Color contrast (text) | 4.5:1 minimum |

### 12.3 Responsive Layout

```typescript
// Use percentage + Flex for responsive
Column() {
  Text('Header').fontSize(24)
}
.width('100%')
.height('100%')
.padding(16)

// Grid breakpoints
GridRow({ breakpoints: { value: ['600vp', '840vp'], reference: BreakpointsReference.WindowSize } }) {
  GridCol({ span: { sm: 12, md: 6, lg: 4 } }) {
    CardComponent()
  }
  GridCol({ span: { sm: 12, md: 6, lg: 4 } }) {
    CardComponent()
  }
}
```

---

## 13. Distributed Capabilities

```typescript
// Enable in module.json5
"abilities": [{
  "name": "EntryAbility",
  "distributed": true,
  "continueOn": true
}]

// Trigger cross-device migration
Button('Migrate to other device')
  .onClick(() => {
    this.context.continueAbility({
      abilityName: "EntryAbility",
      bundleName: this.context.bundleName
    });
  })

// Handle migration data on target device
async onContinueAbility(want: Want): Promise<void> {
  const data = want.parameters?.['migrationData'];
  // Restore state from migration data
}
```

---

## 14. Quick Start Template

Minimal working page template:

```typescript
// pages/Index.ets
import { promptAction } from '@kit.ArkUI';
import { hilog } from '@kit.PerformanceAnalysisKit';

const DOMAIN = 0x0001;
const TAG = 'IndexPage';

@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS';
  @State count: number = 0;

  aboutToAppear() {
    hilog.info(DOMAIN, TAG, 'Index page created');
  }

  build() {
    Column({ space: 20 }) {
      Text(this.message)
        .fontSize(28)
        .fontWeight(FontWeight.Bold)
        .textAlign(TextAlign.Center)

      Text(`Count: ${this.count}`)
        .fontSize(20)

      Button('Increment', { type: ButtonType.Capsule })
        .width('80%')
        .height(48)
        .onClick(() => {
          this.count++;
          promptAction.showToast({ message: `Count: ${this.count}` });
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

---

## References

| Topic | File |
|-------|------|
| Project structure & all config files | [project-structure.md](references/project-structure.md) |
| ArkTS full syntax & type system | [arkts-guide.md](references/arkts-guide.md) |
| ArkUI component library & custom components | [arkui-components.md](references/arkui-components.md) |
| State management patterns & MVVM | [state-management.md](references/state-management.md) |
| Performance, pitfalls & anti-patterns | [performance-pitfalls.md](references/performance-pitfalls.md) |
