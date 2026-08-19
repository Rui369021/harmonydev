# ArkTS Language Guide

## Overview

ArkTS is a superset of TypeScript, extended with declarative UI syntax for HarmonyOS. It is compiled to Ark bytecode and runs on the Ark runtime (方舟运行时) via the ArkCompiler (方舟编译器).

## Key Differences from TypeScript

| Feature | TypeScript | ArkTS |
|---------|-----------|-------|
| Type checking | Runtime + compile (optional) | Strict compile-time only |
| UI syntax | JSX (React) | Declarative decorators (@Component, @Builder) |
| State management | Manual (useState, etc.) | Built-in decorators (@State, @Prop, @Link) |
| Null safety | Optional (strict mode) | Mandatory |
| Object layout | Dynamic | Static (AOT optimization) |
| Type inference | Widely used | Restricted (explicit types preferred) |

## Type System

### Basic Types

```typescript
// Primitives
let name: string = 'Alice';
let age: number = 25;
let isActive: boolean = true;

// Arrays
let items: string[] = ['a', 'b', 'c'];
let numbers: Array<number> = [1, 2, 3];

// Tuples (limited support)
let pair: [string, number] = ['Alice', 25];

// Enums
enum Color { Red, Green, Blue }
let c: Color = Color.Red;

// Union types
let id: string | number = 42;
```

### Interfaces & Classes

```typescript
// Interface (data shape)
interface User {
  id: number;
  name: string;
  email?: string;        // Optional
  readonly createdAt: number;  // Readonly
}

// Class with methods
class UserService {
  private users: User[] = [];

  add(user: User): void {
    this.users.push(user);
  }

  find(id: number): User | undefined {
    return this.users.find(u => u.id === id);
  }

  // Async method
  async fetchRemote(url: string): Promise<User[]> {
    // ... fetch logic
    return [];
  }
}

// Abstract class
abstract class Shape {
  abstract area(): number;
  describe(): string {
    return `Shape with area ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) { super(); }
  area(): number { return Math.PI * this.radius ** 2; }
}
```

### Generic Types

```typescript
// Generic function
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

// Generic class
class Stack<T> {
  private items: T[] = [];
  push(item: T): void { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}

// Generic interface
interface Repository<T> {
  findAll(): T[];
  findById(id: number): T | undefined;
}
```

## Decorator System

### @Entry — Page Root

Marks a component as a page entry point. Only ONE `@Entry` per page.

```typescript
@Entry
@Component
struct IndexPage {
  @State title: string = 'Home';

  build() {
    Column() {
      Text(this.title)
    }
  }
}
```

### @Component — UI Component

Declares a reusable UI component. Must have a `build()` method.

```typescript
@Component
struct UserAvatar {
  @Prop name: string;
  @Prop avatarUrl: string;

  build() {
    Row({ space: 8 }) {
      Image(this.avatarUrl).width(40).height(40).borderRadius(20)
      Text(this.name).fontSize(16)
    }
  }
}
```

### @Builder — Reusable UI Fragment

Extracts UI building logic into a reusable function. Does NOT create a component boundary.

```typescript
// Global builder
@Builder
function DividerLine(color: ResourceColor = '#E5E5E5') {
  Divider()
    .color(color)
    .strokeWidth(1)
    .margin({ top: 8, bottom: 8 })
}

// Component-scoped builder
@Component
struct MyPage {
  @Builder
  itemLayout(text: string, icon: Resource) {
    Row({ space: 12 }) {
      Image(icon).width(24).height(24)
      Text(text).fontSize(16)
    }
    .width('100%')
    .padding(16)
  }

  build() {
    Column() {
      this.itemLayout('Settings', $r('app.media.settings'))
      this.itemLayout('About', $r('app.media.about'))
    }
  }
}
```

### @BuilderParam — Builder as Parameter

Passes builder functions into a component (like React children / slots).

```typescript
@Component
struct Card {
  @BuilderParam content: () => void;

  build() {
    Column() {
      this.content()  // Render the passed builder
    }
    .padding(16)
    .borderRadius(12)
    .backgroundColor(Color.White)
  }
}

// Usage
@Component
struct HomePage {
  @Builder
  cardContent() {
    Text('Hello from card!').fontSize(18)
  }

  build() {
    Card({ content: this.cardContent })
  }
}
```

### @Extend — Extend Built-in Component Styles

```typescript
// Extend Text with custom styles
@Extend(Text)
functionHeaderText(text: ResourceStr) {
  .fontSize(24)
  .fontWeight(FontWeight.Bold)
  .fontColor('#333333')
  .margin({ bottom: 8 })
}

// Usage
Text('My Header').HeaderText()
```

### @Styles — Reusable Style Set

```typescript
@Styles
function cardStyle() {
  .padding(16)
  .borderRadius(12)
  .backgroundColor(Color.White)
  .shadow({ radius: 4, color: 'rgba(0,0,0,0.1)', offsetX: 0, offsetY: 2 })
}

// Usage
Column() {
  Text('Card content')
}
.cardStyle()
```

## Control Flow in build()

### if/else

```typescript
build() {
  Column() {
    if (this.isLoading) {
      LoadingProgress().width(40).height(40)
    } else if (this.data === null) {
      Text('No data').fontSize(16).fontColor('#999')
    } else {
      Text(this.data.title).fontSize(20)
    }
  }
}
```

### ForEach

```typescript
@State items: string[] = ['Apple', 'Banana', 'Cherry'];

build() {
  Column() {
    ForEach(this.items, (item: string, index: number) => {
      Text(`${index + 1}. ${item}`)
        .fontSize(16)
        .padding(8)
    }, (item: string) => item)  // Key generator (important for diffing)
  }
}
```

### LazyForEach (for large lists)

```typescript
import { LazyForEach } from '@ohos.arkui';

// Must use IDataSource implementation
class MyDataSource implements IDataSource {
  private list: string[] = [];
  private listeners: DataChangeListener[] = [];

  totalCount(): number { return this.list.length; }
  getData(index: number): string { return this.list[index]; }

  pushData(item: string): void {
    this.list.push(item);
    this.listeners.forEach(l => l.onDataAdd(this.list.length - 1));
  }

  registerDataChangeListener(listener: DataChangeListener): void {
    this.listeners.push(listener);
  }
  unregisterDataChangeListener(listener: DataChangeListener): void {
    this.listeners = this.listeners.filter(l => l !== listener);
  }
}

@Component
struct LongListPage {
  private dataSource: MyDataSource = new MyDataSource();

  aboutToAppear() {
    for (let i = 0; i < 1000; i++) {
      this.dataSource.pushData(`Item ${i + 1}`);
    }
  }

  build() {
    List() {
      LazyForEach(this.dataSource, (item: string) => {
        ListItem() {
          Text(item).fontSize(16).padding(16)
        }
      }, (item: string) => item)
    }
    .height('100%')
  }
}
```

## Event Handling

```typescript
@Component
struct EventDemo {
  @State count: number = 0;
  @State inputText: string = '';

  build() {
    Column({ space: 16 }) {
      // Click event
      Button('Click me')
        .onClick((event: ClickEvent) => {
          console.info(`Click at: ${event.x}, ${event.y}`);
          this.count++;
        })

      // Touch event (more detailed than click)
      Text('Touch me')
        .onTouch((event: TouchEvent) => {
          if (event.type === TouchType.Down) {
            console.info('Touch down');
          } else if (event.type === TouchType.Up) {
            console.info('Touch up');
          }
        })

      // Input change
      TextInput({ placeholder: 'Type here' })
        .onChange((value: string) => {
          this.inputText = value;
        })

      // Submit event
      TextInput({ placeholder: 'Press enter' })
        .onSubmit((value: string) => {
          console.info(`Submitted: ${value}`);
        })

      // Swipe gesture
      Text('Swipe me')
        .gesture(
          SwipeGesture({ fingers: 1, speed: 20 })
            .onAction((event: GestureEvent) => {
              console.info(`Swipe angle: ${event.angle}`);
            })
        )

      // Long press
      Text('Long press me')
        .gesture(
          LongPressGesture({ repeat: false, duration: 500 })
            .onAction(() => {
              console.info('Long pressed!');
            })
        )
    }
  }
}
```

## Async Patterns

```typescript
// Promise
function fetchData(url: string): Promise<string> {
  return new Promise((resolve, reject) => {
    // ... async operation
    if (success) {
      resolve(data);
    } else {
      reject(new Error('Failed'));
    }
  });
}

// async/await
async function loadData() {
  try {
    const result = await fetchData('https://api.example.com/data');
    return JSON.parse(result);
  } catch (error) {
    console.error(`Load failed: ${error.message}`);
    return null;
  }
}

// Parallel async
async function loadAll() {
  const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts()
  ]);
  return { users, posts };
}
```

## Import System

```typescript
// Import from system kit (HarmonyOS NEXT style)
import { promptAction } from '@kit.ArkUI';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { http } from '@kit.NetworkKit';

// Import from @ohos namespace (legacy, still works)
import http from '@ohos.net.http';
import router from '@ohos.router';
import preferences from '@ohos.data.preferences';

// Import local modules
import { UserCard } from '../components/UserCard';
import { UserService } from '../services/UserService';

// Type-only import
import type { User } from '../models/User';
```

## Null Safety

```typescript
// Strict null checking — types do NOT include null/undefined unless explicitly declared
let name: string = 'Alice';
// name = null;  // COMPILE ERROR

let optionalName: string | null = null;  // OK
let maybeName: string | undefined;       // OK

// Safe access
let length = optionalName?.length;       // Returns number | undefined

// Nullish coalescing
let displayName = optionalName ?? 'Unknown';

// Optional chaining on methods
let result = service?.findUser(1)?.name;
```
