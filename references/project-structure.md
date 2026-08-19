# Project Structure & Configuration

## Full Directory Tree

```
MyApplication/
├── AppScope/                              # App-level global config (ONE per app)
│   ├── app.json5                          # bundleName, version, icon, label
│   ├── resources/
│   │   └── base/
│   │       └── element/
│   │           ├── string.json            # Global strings (app_name HERE ONLY)
│   │           ├── color.json             # Global colors
│   │           └── float.json             # Global float dimensions
│   └── resources/
│       ├── base/media/                    # Global media (app icon, etc.)
│       └── dark/                          # Dark mode overrides (optional)
│
├── entry/                                 # Main entry module (HAP package)
│   ├── src/
│   │   ├── main/
│   │   │   ├── ets/
│   │   │   │   ├── entryability/
│   │   │   │   │   └── EntryAbility.ets   # Main ability (lifecycle)
│   │   │   │   ├── entrybackupability/    # Backup extension (auto-generated)
│   │   │   │   │   └── EntryBackupAbility.ets
│   │   │   │   └── pages/
│   │   │   │       ├── Index.ets          # Home page
│   │   │   │       ├── Detail.ets         # Detail page
│   │   │   │       └── Settings.ets       # Settings page
│   │   │   ├── resources/
│   │   │   │   ├── base/
│   │   │   │   │   ├── element/
│   │   │   │   │   │   ├── string.json    # Module strings
│   │   │   │   │   │   ├── color.json     # Module colors
│   │   │   │   │   │   └── float.json     # Module float dimensions
│   │   │   │   │   ├── media/             # Module images/icons
│   │   │   │   │   └── profile/
│   │   │   │   │       └── main_pages.json  # Page routing registry
│   │   │   │   ├── dark/                  # Dark mode resources
│   │   │   │   │   └── element/
│   │   │   │   │       └── color.json     # Dark mode color overrides
│   │   │   │   ├── en_US/                 # English locale resources
│   │   │   │   │   └── element/
│   │   │   │   │       └── string.json
│   │   │   │   ├── zh_CN/                 # Chinese locale resources
│   │   │   │   │   └── element/
│   │   │   │   │       └── string.json
│   │   │   │   └── rawfile/               # Raw files (not compiled, copied as-is)
│   │   │   └── module.json5               # Module config
│   │   ├── ohosTest/                      # Instrumentation tests (optional)
│   │   │   └── ets/
│   │   │       └── test/
│   │   │           └── Ability.test.ets
│   │   └── test/                          # Unit tests (optional)
│   │       └── Ability.test.ets
│   ├── build-profile.json5                # Module build config
│   ├── hvigorfile.ts                      # Module build script
│   ├── obfuscation-rules.txt              # Code obfuscation rules (Release)
│   └── oh-package.json5                   # Module dependencies
│
├── library/                               # Shared library module (HAR/HSP, optional)
│   ├── src/main/
│   │   ├── ets/
│   │   │   └── components/                # Shared components
│   │   ├── resources/
│   │   └── module.json5
│   ├── build-profile.json5
│   └── oh-package.json5
│
├── build-profile.json5                    # Project-level build config
├── hvigorfile.ts                          # Project build script
├── oh-package.json5                       # Project-level dependencies
├── code-linter.json5                      # Linter config (optional)
└── oh_modules/                            # Installed dependencies (like node_modules)
```

## Module Types

| Type | Extension | Description |
|------|-----------|-------------|
| **HAP** (Harmony Ability Package) | .hap | Installable application package; each HAP is an independent module |
| **HSP** (Harmony Shared Package) | .hsp | Shared dynamic package; loaded at runtime, reduces app size |
| **HAR** (Harmony Archive) | .har | Static shared library; compiled into each consuming module |
| **APP** | .app | Application package containing one or more HAPs for AppGallery |

## app.json5 (AppScope)

```json5
{
  "app": {
    "bundleName": "com.example.myapp",    // Unique app ID, reverse domain format
    "vendor": "ExampleCorp",              // Developer name
    "versionCode": 1000000,               // Integer version code (for update comparison)
    "versionName": "1.0.0",               // Display version string
    "icon": "$media:app_icon",            // App icon reference
    "label": "$string:app_name",          // App name reference
    "minAPIVersion": 12,                  // Minimum API level (HarmonyOS NEXT = API 12+)
    "targetAPIVersion": 26,               // Target API level
    "apiReleaseType": "Release"           // API release type: Release | Canary | Beta
  }
}
```

### Rules
- `bundleName` must be globally unique and in reverse-domain format
- `app_name` string MUST be defined in AppScope resources ONLY
- `versionCode` must be an integer that increases with each release
- Changing `bundleName` creates a new app identity (cannot update existing installs)

## module.json5 (entry)

Full configuration reference:

```json5
{
  "module": {
    "name": "entry",                        // Module name
    "type": "entry",                        // Module type: entry | feature | shared
    "description": "$string:module_desc",   // Module description
    "mainElement": "EntryAbility",          // Main entry ability
    "deviceTypes": [                        // Target devices
      "phone",
      "tablet",
      "2in1",
      "car",
      "wearable",
      "smartScreen"
    ],
    "deliveryWithInstall": true,            // Install with app (must be true for entry)
    "installationFree": false,              // Atomic service (元服务) support
    "pages": "$profile:main_pages",         // Page routing config
    "abilities": [                          // UIAbility declarations
      {
        "name": "EntryAbility",
        "srcEntrance": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "icon": "$media:app_icon",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:app_icon",        // Splash screen icon
        "startWindowBackground": "$color:start_window_background",  // Splash bg color
        "exported": true,                             // Can be called by other apps
        "skills": [                                   // Intent filters
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]         // Launcher entry
          }
        ],
        "distributed": true,                          // Enable cross-device distribution
        "continueOn": true                            // Support ability migration
      }
    ],
    "extensionAbilities": [                          // Extension abilities (background services)
      {
        "name": "FormExtensionAbility",
        "srcEntrance": "./ets/formability/FormExtensionAbility.ets",
        "type": "form",                               // form | service | dataShare
        "metadata": [
          {
            "name": "ohos.extension.form",
            "resource": "$profile:form_config"
          }
        ]
      }
    ],
    "requestPermissions": [                          // System permissions
      {
        "name": "ohos.permission.INTERNET",
        "reason": "$string:reason_internet",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"                             // inuse | always
        }
      },
      {
        "name": "ohos.permission.WRITE_MEDIA",
        "reason": "$string:reason_write_media",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ],
    "metadata": [                                    // Custom metadata
      {
        "name": "custom_data",
        "value": "custom_value"
      }
    ]
  }
}
```

### Common Permissions

| Permission | Description | Grant Type |
|------------|-------------|------------|
| `ohos.permission.INTERNET` | Network access | Normal (auto-grant) |
| `ohos.permission.WRITE_MEDIA` | Write to media library | Normal |
| `ohos.permission.READ_MEDIA` | Read media library | Normal |
| `ohos.permission.CAMERA` | Camera access | Dynamic (user prompt) |
| `ohos.permission.MICROPHONE` | Microphone access | Dynamic (user prompt) |
| `ohos.permission.LOCATION` | GPS location | Dynamic (user prompt) |
| `ohos.permission.READ_CALENDAR` | Read calendar | Dynamic (user prompt) |
| `ohos.permission.WRITE_CALENDAR` | Write calendar | Dynamic (user prompt) |
| `ohos.permission.READ_CONTACTS` | Read contacts | Dynamic (user prompt) |
| `ohos.permission.WRITE_CONTACTS` | Write contacts | Dynamic (user prompt) |

### Dynamic Permission Request

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import { common } from '@ohos.abilityKit';

async function requestCameraPermission(context: common.UIAbilityContext): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  const permissions: string[] = ['ohos.permission.CAMERA'];

  try {
    const result = await atManager.requestPermissionsFromUser(context, permissions);
    return result.authResults[0] === 0;  // 0 = granted
  } catch (err) {
    console.error(`Permission request failed: ${err.message}`);
    return false;
  }
}
```

## main_pages.json

```json
{
  "src": [
    "pages/Index",
    "pages/Detail",
    "pages/Settings",
    "pages/Profile/EditProfile"
  ]
}
```

> Every page referenced here MUST have a corresponding `.ets` file at `src/main/ets/{path}.ets`.

## build-profile.json5 (Project-level)

```json5
{
  "app": {
    "signingConfigs": [
      {
        "name": "default",
        "type": "HarmonyOS",
        "material": {
          "certpath": "",
          "storePassword": "",
          "keyAlias": "",
          "keyPassword": "",
          "profile": "",
          "signAlg": "SHA256withECDSA",
          "storeFile": ""
        }
      }
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compatibleSdkVersion": "5.0.0(12)",
        "runtimeOS": "HarmonyOS",
        "buildOption": {
          "strictMode": {
            "useNormalizedOHOSUrl": true,
            "caseSensitiveCheck": true
          }
        }
      }
    ],
    "buildModeSet": [
      { "name": "debug", "externalOptions": {} },
      { "name": "release", "externalOptions": {} }
    ]
  },
  "modules": [
    { "name": "entry", "srcPath": "./entry", "targets": [{ "name": "default" }] }
  ]
}
```

## oh-package.json5

```json5
{
  "name": "my-application",
  "version": "1.0.0",
  "description": "My HarmonyOS Application",
  "main": "",
  "author": "Developer Name",
  "license": "Apache-2.0",
  "dependencies": {
    "@ohos/hypium": "1.0.21"
  },
  "devDependencies": {
    "@ohos/hvigor-ohos-plugin": "5.0.0"
  }
}
```

### Module-level oh-package.json5 (entry)

```json5
{
  "name": "entry",
  "version": "1.0.0",
  "description": "Entry module",
  "main": "",
  "author": "",
  "license": "Apache-2.0",
  "dependencies": {}
}
```

## Resource File Format

### string.json
```json
{
  "string": [
    { "name": "app_name", "value": "MyApp" },
    { "name": "login_title", "value": "Sign In" },
    { "name": "btn_submit", "value": "Submit" }
  ]
}
```

### color.json
```json
{
  "color": [
    { "name": "primary", "value": "#007DFF" },
    { "name": "background", "value": "#F1F3F5" },
    { "name": "start_window_background", "value": "#FFFFFF" }
  ]
}
```

### float.json
```json
{
  "float": [
    { "name": "title_size", "value": "24.0vp" },
    { "name": "body_size", "value": "16.0vp" },
    { "name": "card_radius", "value": "12.0vp" }
  ]
}
```

### Dark Mode Override (resources/dark/element/color.json)
```json
{
  "color": [
    { "name": "background", "value": "#1A1A1A" },
    { "name": "start_window_background", "value": "#1A1A1A" }
  ]
}
```

### Locale Override (resources/zh_CN/element/string.json)
```json
{
  "string": [
    { "name": "login_title", "value": "登录" },
    { "name": "btn_submit", "value": "提交" }
  ]
}
```
