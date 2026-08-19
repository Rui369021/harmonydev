# Performance, Pitfalls & Anti-Patterns

## Top Performance Optimizations

### 1. LazyForEach for Large Lists

```typescript
// WRONG: ForEach renders ALL items eagerly (memory + render time)
ForEach(this.items, (item: Data) => {
  ListItem() { /* ... */ }
})
// 1000 items = 1000 components rendered = OOM risk

// CORRECT: LazyForEach only renders visible items
LazyForEach(this.dataSource, (item: Data) => {
  ListItem() { /* ... */ }
}, (item: Data) => item.id.toString())
// 1000 items = ~10 visible components rendered
```

**Requirements for LazyForEach**:
- Must implement `IDataSource` interface
- Must provide a key generator function (3rd parameter)
- Key must be unique and stable per item

### 2. Avoid Computation in build()

```typescript
// WRONG: Expensive computation runs on EVERY re-render
@State data: number[] = [1, 2, 3, /* ... 10000 items */];

build() {
  Text(`Total: ${this.data.reduce((a, b) => a + b, 0)}`)  // Runs every time!
}

// CORRECT: Pre-compute in lifecycle hook
@State data: number[] = [1, 2, 3, /* ... */];
@State total: number = 0;

aboutToAppear() {
  this.total = this.data.reduce((a, b) => a + b, 0);
}

build() {
  Text(`Total: ${this.total}`)
}
```

### 3. Animation Performance

```typescript
// WRONG: Manual frame updates (causes jank)
setInterval(() => {
  this.x += 1;
}, 16);

// CORRECT: Use animation() for property changes
@State x: number = 0;

build() {
  Text('Slide')
    .translate({ x: this.x })
    .animation({ duration: 300, curve: Curve.EaseOut })
    .onClick(() => {
      this.x = 200;  // Smooth animation, GPU-accelerated
    })
}

// CORRECT: Use animateTo for explicit control
build() {
  Text('Slide')
    .translate({ x: this.x })
    .onClick(() => {
      animateTo({ duration: 300, curve: Curve.EaseOut }, () => {
        this.x = 200;
      });
    })
}
```

### 4. Image Optimization

```typescript
// WRONG: Large image without optimization
Image($r('app.media.large_photo'))
  .width(100)
  .height(100)

// CORRECT: Add objectFit, alt, interpolation
Image($r('app.media.large_photo'))
  .width(100)
  .height(100)
  .objectFit(ImageFit.Cover)              // Crop to fill, no distortion
  .interpolation(ImageInterpolation.High)  // Smooth rendering
  .alt($r('app.media.placeholder'))         // Show placeholder while loading
  .draggable(false)                        // Prevent unintended drag
```

### 5. Conditional Rendering (Reduce Component Count)

```typescript
// WRONG: All branches always rendered (just hidden)
@State showA: boolean = true;

build() {
  Stack() {
    ComponentA().visibility(this.showA ? Visibility.Visible : Visibility.Hidden)
    ComponentB().visibility(!this.showA ? Visibility.Visible : Visibility.Hidden)
  }
  // Both components are in the tree, consuming memory

// CORRECT: Only render the needed branch
build() {
  if (this.showA) {
    ComponentA()
  } else {
    ComponentB()
  }
  // Only one component in the tree
}
```

## Common Pitfalls

### Pitfall 1: @State Not Updating UI

**Symptom**: You change a variable but the UI doesn't refresh.

```typescript
// Problem: Array.push doesn't trigger @State
@State items: string[] = ['a', 'b'];

addBad() {
  this.items.push('c');  // Array reference unchanged -> NO refresh
}

// Solution: Reassign the array
addGood() {
  this.items = [...this.items, 'c'];  // New reference -> refresh!
}

// Problem: Object property change doesn't trigger @State
@State user: User = { name: 'Alice', age: 25 };

updateBad() {
  this.user.name = 'Bob';  // Object reference unchanged -> NO refresh
}

// Solution: Reassign the object
updateGood() {
  this.user = { ...this.user, name: 'Bob' };  // New reference -> refresh!
}

// OR: Use @Observed + @ObjectLink for property-level tracking
@Observed
class User {
  name: string = '';
  age: number = 0;
}

@Component
struct UserCard {
  @ObjectLink user: User;  // Property changes ARE tracked
}
```

### Pitfall 2: Missing $ Prefix for @Link

```typescript
// WRONG: Forgetting $ causes one-way binding instead of two-way
@Component
struct Child {
  @Link value: number;
}

@Entry
@Component
struct Parent {
  @State count: number = 0;

  build() {
    Child({ value: this.count })  // WRONG: No $ = one-way @Prop behavior
  }
}

// CORRECT: Use $ prefix for two-way binding
build() {
  Child({ value: $count })  // CORRECT: $ = two-way @Link
}
```

### Pitfall 3: Network Resource Leak

```typescript
// WRONG: HttpRequest not destroyed = resource leak + eventual crash
async function fetchData(url: string) {
  const http = http.createHttp();
  const response = await http.request(url);
  return response.result;  // http NEVER destroyed!
}

// CORRECT: Destroy in finally block
async function fetchData(url: string) {
  const http = http.createHttp();
  try {
    const response = await http.request(url);
    return response.result;
  } finally {
    http.destroy();  // ALWAYS destroy!
  }
}
```

### Pitfall 4: Preferences Not Persisting

```typescript
// WRONG: Missing flush() call
async function saveData(context: Context) {
  const pref = await preferences.getPreferences(context, { name: 'myApp' });
  await pref.put('key', 'value');
  // Data NOT persisted! Lost on restart.
}

// CORRECT: Call flush() after put()
async function saveData(context: Context) {
  const pref = await preferences.getPreferences(context, { name: 'myApp' });
  await pref.put('key', 'value');
  await pref.flush();  // MUST call flush to persist to disk!
}
```

### Pitfall 5: Duplicate app_name Definition

```json5
// WRONG: app_name defined in BOTH AppScope and entry module
// AppScope/resources/base/element/string.json
{ "string": [{ "name": "app_name", "value": "MyApp" }] }

// entry/src/main/resources/base/element/string.json
{ "string": [{ "name": "app_name", "value": "MyApp" }] }
// COMPILE ERROR: Duplicate resource name!

// CORRECT: Define app_name in AppScope ONLY
// entry/src/main/resources/base/element/string.json
{ "string": [{ "name": "entry_desc", "value": "Entry module" }] }
```

### Pitfall 6: LazyForEach Missing Key Generator

```typescript
// WRONG: No key generator = rendering bugs, flickering, wrong items
LazyForEach(this.dataSource, (item: Data) => {
  ListItem() { Text(item.title) }
})
// No 3rd parameter = no key = diffing fails

// CORRECT: Always provide unique key generator
LazyForEach(this.dataSource, (item: Data) => {
  ListItem() { Text(item.title) }
}, (item: Data) => item.id.toString())
// Key = unique id per item = proper diffing
```

### Pitfall 7: Distributed Migration Not Working

```json5
// Problem: forgot to enable distributed in module.json5
"abilities": [{
  "name": "EntryAbility"
  // Missing: "distributed": true, "continueOn": true
}]

// CORRECT: Enable distributed flags
"abilities": [{
  "name": "EntryAbility",
  "distributed": true,
  "continueOn": true
}]
```

### Pitfall 8: build() Calling Non-Composable Functions

```typescript
// WRONG: Calling a function that returns UI from non-build context
function getGreeting(): string {
  // This is fine - returns a string
  return 'Hello';
}

// WRONG: Trying to call composable outside @Builder or build()
function renderHeader() {
  Text('Header')  // COMPILE ERROR: composable outside build context
}

// CORRECT: Wrap in @Builder
@Builder
function renderHeader() {
  Text('Header')  // OK: @Builder context
}

// Usage in build()
build() {
  Column() {
    renderHeader()  // Call the builder
  }
}
```

### Pitfall 9: Using console.log Instead of hilog

```typescript
// WRONG: console.log in production (no filtering, no level control)
console.log('Debug info');
console.error('Error occurred');

// CORRECT: Use hilog for structured logging
import { hilog } from '@kit.PerformanceAnalysisKit';

const DOMAIN = 0x0001;
const TAG = 'MyModule';

hilog.info(DOMAIN, TAG, 'User logged in: %{public}s', userId);
hilog.warn(DOMAIN, TAG, 'Cache miss for key: %{public}s', key);
hilog.error(DOMAIN, TAG, 'API failed: %{public}s', error.message);

// %{public}s = visible in logs (use for non-sensitive data)
// %{private}s = hidden in release logs (use for sensitive data)
```

### Pitfall 10: Not Handling Async Errors

```typescript
// WRONG: No error handling, unhandled promise rejection
async function loadData() {
  const response = await fetch(url);  // If fetch fails, app crashes
  const data = await response.json();
  return data;
}

// CORRECT: Wrap in try-catch
async function loadData(): Promise<Data | null> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    hilog.error(DOMAIN, TAG, 'Load failed: %{public}s', error.message);
    return null;  // Graceful fallback
  }
}
```

## Build Error Quick Reference

| Error | Cause | Fix |
|-------|-------|-----|
| `Resource not found` | Wrong resource path | Verify `$r('app.type.name')` matches file structure |
| `Duplicate resource` | Same resource name in multiple scopes | Define globally in AppScope, not in modules |
| `Type mismatch` | @Prop/@Link type mismatch | Ensure parent and child types match exactly |
| `Cannot find module` | Missing import or dependency | Check import path; run `ohpm install` |
| `@Link requires $ prefix` | Missing $ in binding | Use `Child({ prop: $stateVar })` |
| `Not a composable context` | Calling UI functions outside build() | Wrap in `@Builder` or call within `build()` |
| `ERR_PNPM_NO_MATCHING_VERSION` | Dependency version not found | Check `oh-package.json5` versions; run `ohpm install` |
| `hvigor build failed` | Build config error | Check `build-profile.json5`; run `hvigorw clean` then rebuild |
| `Ability not found` | Wrong ability name in config | Check `module.json5` `mainElement` matches ability name |
| `Permission denied` | Missing runtime permission | Add to `requestPermissions` and call `requestPermissionsFromUser` |

## Performance Checklist

- [ ] Large lists use `LazyForEach` with unique key generator
- [ ] No expensive computation inside `build()`
- [ ] Animations use `animation()` or `animateTo()`, not manual timers
- [ ] Images have `objectFit` set and use `alt` placeholder
- [ ] `@State` updates use reassignment, not mutation
- [ ] `@Observed`/`@ObjectLink` used for object property tracking
- [ ] Conditional rendering uses `if/else`, not `visibility`
- [ ] Network requests use `try/finally` with `destroy()`
- [ ] Preferences use `flush()` after `put()`
- [ ] Logging uses `hilog`, not `console.log`
- [ ] `build()` methods are short and focused
- [ ] Components are small and reusable
- [ ] `aboutToAppear()` for data initialization, not `build()`
- [ ] `aboutToDisappear()` for cleanup (timers, listeners)
