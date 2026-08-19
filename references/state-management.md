# State Management Guide

## State Management Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AppStorage (Global)                    │
│         @StorageLink / @StorageProp                       │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              LocalStorage (Page-level)             │   │
│  │         @LocalStorageLink / @LocalStorageProp      │   │
│  │                                                   │   │
│  │  ┌────────────────────────────────────────┐      │   │
│  │  │  @Entry Component                       │      │   │
│  │  │  ├── @State (local state)               │      │   │
│  │  │  ├── @Provide (provide to descendants)  │      │   │
│  │  │  └── children:                          │      │   │
│  │  │      ├── @Prop (one-way from parent)    │      │   │
│  │  │      ├── @Link (two-way with parent)    │      │   │
│  │  │      └── @Consume (from ancestor)       │      │   │
│  │  └────────────────────────────────────────┘      │   │
│  └───────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

## @State — Local Component State

Triggers UI refresh when the decorated variable changes.

```typescript
@Component
struct Counter {
  @State count: number = 0;
  @State isVisible: boolean = true;
  @State title: string = 'Hello';
  @State items: string[] = [];

  build() {
    Column() {
      Text(`${this.count}`)
      Button('Add')
        .onClick(() => {
          // CORRECT: Reassign triggers refresh
          this.count = this.count + 1;
        })
    }
  }
}
```

### Important Rules

1. **Reassignment triggers refresh**: `this.count = newValue` works
2. **Array mutation does NOT trigger refresh**: `this.items.push('x')` — must use `this.items = [...this.items, 'x']`
3. **Object property mutation does NOT trigger refresh**: `this.user.name = 'Bob'` — must use `this.user = { ...this.user, name: 'Bob' }`
4. **Only first-level properties are observed**: Nested object property changes need @Observed

### Supported Types

| Type | Observation Behavior |
|------|---------------------|
| `number`, `string`, `boolean` | Value change triggers refresh |
| `Object` (class instance) | Assignment triggers refresh; property change does NOT (use @Observed) |
| `Array` | Assignment triggers refresh; element mutation does NOT |
| `Map`, `Set` | Assignment triggers refresh; mutation does NOT |
| `undefined`, `null` | Assignment triggers refresh |

## @Prop — One-Way Parent to Child

Receives a value from parent. Creates a local copy. Changes in parent propagate down; changes in child do NOT propagate up.

```typescript
@Component
struct DisplayCard {
  @Prop title: string;        // Receives from parent
  @Prop count: number;        // Local copy, parent changes propagate

  build() {
    Column() {
      Text(this.title).fontSize(20)
      Text(`${this.count}`).fontSize(16)

      // This changes the LOCAL copy only — parent is NOT affected
      Button('Local increment')
        .onClick(() => { this.count++; })
    }
  }
}

@Entry
@Component
struct Parent {
  @State parentCount: number = 0;
  @State parentTitle: string = 'My Title';

  build() {
    Column() {
      DisplayCard({
        title: this.parentTitle,    // One-way binding
        count: this.parentCount     // Parent changes propagate to child
      })
      Button('Parent increment')
        .onClick(() => { this.parentCount++; })  // Child's @Prop updates too!
    }
  }
}
```

### @Prop Type Rules

- `@Prop` accepts: `string`, `number`, `boolean`, `Object`, `Array`
- Parent to child type must be compatible
- `@Prop` creates a **deep copy** of the value (changes are local)

## @Link — Two-Way Parent to Child

Bidirectional binding. Changes in child propagate to parent and vice versa.

```typescript
@Component
struct SliderControl {
  @Link value: number;    // Two-way binding with parent

  build() {
    Slider({ value: this.value, min: 0, max: 100 })
      .onChange((newValue: number) => {
        this.value = newValue;   // Updates parent too!
      })
  }
}

@Entry
@Component
struct Parent {
  @State volume: number = 50;

  build() {
    Column() {
      Text(`Volume: ${this.volume}`)
      // IMPORTANT: Use $ prefix for @Link binding!
      SliderControl({ value: $volume })
    }
  }
}
```

### @Link Binding Syntax

```typescript
// CORRECT: $ prefix creates two-way binding
Child({ prop: $stateVar })

// WRONG: Missing $ — this is actually @Prop behavior, not @Link
Child({ prop: stateVar })
```

### @Link Type Rules

- `@Link` accepts: `string`, `number`, `boolean`, `Object`, `Array`
- The parent's `@State` type and child's `@Link` type must be **exactly the same**
- `@Link` does NOT create a copy — it references the parent's state

## @Provide / @Consume — Ancestor to Descendant

Cross-level data sharing without prop drilling. An ancestor `@Provide`s data; any descendant can `@Consume` it.

```typescript
@Entry
@Component
struct App {
  @Provide('theme') theme: string = 'light';
  @Provide('user') user: User = { name: 'Alice', age: 25 };

  build() {
    Column() {
      HeaderComponent()
      ContentComponent()
      FooterComponent()
    }
  }
}

@Component
struct HeaderComponent {
  @Consume('theme') theme: string;   // Gets 'light' from App
  @Consume('user') user: User;       // Gets user from App

  build() {
    Row() {
      Text(`Hello, ${this.user.name}`)
        .fontColor(this.theme === 'dark' ? Color.White : Color.Black)
    }
  }
}

@Component
struct ContentComponent {
  @Consume('theme') theme: string;

  build() {
    Column() {
      Text('Content')
        .backgroundColor(this.theme === 'dark' ? '#333' : '#FFF')
    }
  }
}
```

### @Provide/@Consume Rules

- The string key must match between `@Provide('key')` and `@Consume('key')`
- Changes to `@Provide` value trigger refresh in ALL `@Consume` descendants
- Changes to `@Consume` value propagate back to `@Provide` ancestor (two-way)
- Works across any depth of component nesting

## AppStorage — Global State

Application-level key-value store. Persists across pages and components.

```typescript
// Set / create global state
AppStorage.setOrCreate('isLogin', false);
AppStorage.setOrCreate('currentUser', { name: '', age: 0 });

// Read global state (one-time read)
let isLogin = AppStorage.get<boolean>('isLogin');

// Set global state
AppStorage.set('isLogin', true);

// Delete global state
AppStorage.delete('currentUser');

// Check if key exists
let hasKey = AppStorage.has('isLogin');

// Clear all
AppStorage.clear();
```

### @StorageLink / @StorageProp — Bind to AppStorage

```typescript
@Entry
@Component
struct HomePage {
  // Two-way binding with AppStorage
  @StorageLink('isLogin') isLogin: boolean = false;
  @StorageLink('currentUser') user: User = { name: '', age: 0 };

  // One-way binding (read-only from AppStorage)
  @StorageProp('appVersion') appVersion: string = '1.0.0';

  build() {
    Column() {
      if (this.isLogin) {
        Text(`Welcome, ${this.user.name}`)
      } else {
        Button('Login')
          .onClick(() => {
            // Updates AppStorage globally — all @StorageLink observers refresh
            AppStorage.set('isLogin', true);
            AppStorage.set('currentUser', { name: 'Alice', age: 25 });
          })
      }

      Text(`Version: ${this.appVersion}`)
    }
  }
}
```

### @StorageLink vs @StorageProp

| Decorator | Direction | Local Change Updates AppStorage? | AppStorage Change Updates Local? |
|-----------|-----------|-----------------------------------|----------------------------------|
| `@StorageLink` | Two-way | YES | YES |
| `@StorageProp` | One-way | NO (local copy) | YES (initial read + subsequent updates) |

## LocalStorage — Page-Level State

Similar to AppStorage but scoped to a single page's component tree.

```typescript
// Create a LocalStorage with initial data
let localStorage = new LocalStorage({
  pageTitle: 'My Page',
  dataCount: 0
});

// Use with @Entry
@Entry(localStorage)
@Component
struct MyPage {
  // Two-way binding with LocalStorage
  @LocalStorageLink('pageTitle') pageTitle: string = '';
  @LocalStorageLink('dataCount') dataCount: number = 0;

  build() {
    Column() {
      Text(this.pageTitle)
      Text(`Count: ${this.dataCount}`)
      Button('Increment')
        .onClick(() => { this.dataCount++; })  // Updates LocalStorage
    }
  }
}
```

## @Observed / @ObjectLink — Deep Object Observation

For observing changes to nested object properties (class instances).

```typescript
// Step 1: Mark class as @Observed
@Observed
class UserModel {
  name: string;
  age: number;
  address: Address;

  constructor(name: string, age: number, address: Address) {
    this.name = name;
    this.age = age;
    this.address = address;
  }
}

@Observed
class Address {
  city: string;
  street: string;

  constructor(city: string, street: string) {
    this.city = city;
    this.street = street;
  }
}

// Step 2: Use @ObjectLink in child component
@Component
struct UserCard {
  @ObjectLink user: UserModel;    // Observes property changes on this object

  build() {
    Column() {
      Text(this.user.name)        // Updates when user.name changes!
      Text(`Age: ${this.user.age}`)
      Text(this.user.address.city) // Updates when address.city changes!
      Button('Change Name')
        .onClick(() => {
          // Direct property mutation IS observed because of @Observed
          this.user.name = 'Bob';
        })
    }
  }
}

// Step 3: Use @State in parent to hold the observed object
@Entry
@Component
struct UserList {
  @State users: UserModel[] = [
    new UserModel('Alice', 25, new Address('Beijing', 'Main St'))
  ];

  build() {
    Column() {
      ForEach(this.users, (user: UserModel) => {
        UserCard({ user: user })
      }, (user: UserModel) => user.name)
    }
  }
}
```

### @Observed Rules

1. `@Observed` must be applied to the **class definition**
2. `@ObjectLink` must be used in the **child component** that receives the object
3. The parent must use `@State` to hold the object
4. Only first-level properties of `@Observed` class are tracked; nested `@Observed` classes enable deeper tracking
5. `@ObjectLink` does NOT accept `undefined` or `null` — the object must be initialized

## @Track — Field-Level Tracking

Fine-grained control over which fields trigger re-render.

```typescript
@Observed
class Product {
  @Track name: string;        // Only 'name' changes trigger re-render
  @Track price: number;       // Only 'price' changes trigger re-render
  description: string;        // Changes do NOT trigger re-render (no @Track)

  constructor(name: string, price: number, description: string) {
    this.name = name;
    this.price = price;
    this.description = description;
  }
}

@Component
struct ProductCard {
  @ObjectLink product: Product;

  build() {
    Column() {
      Text(this.product.name)     // Re-renders when 'name' changes
      Text(`$${this.product.price}`)  // Re-renders when 'price' changes
      Text(this.product.description)  // Does NOT re-render on description change
    }
  }
}
```

## MVVM Pattern

```typescript
// Model
@Observed
class TodoItem {
  id: number;
  title: string;
  done: boolean;

  constructor(id: number, title: string) {
    this.id = id;
    this.title = title;
    this.done = false;
  }
}

// ViewModel
class TodoViewModel {
  @State items: TodoItem[] = [];

  addItem(title: string) {
    const item = new TodoItem(Date.now(), title);
    this.items = [...this.items, item];
  }

  toggleItem(id: number) {
    this.items = this.items.map(item => {
      if (item.id === id) {
        item.done = !item.done;  // @Observed tracks this
      }
      return item;
    });
  }

  deleteItem(id: number) {
    this.items = this.items.filter(item => item.id !== id);
  }
}

// View
@Entry
@Component
struct TodoApp {
  @State viewModel: TodoViewModel = new TodoViewModel();
  @State newTodoText: string = '';

  build() {
    Column({ space: 12 }) {
      // Input
      Row() {
        TextInput({ placeholder: 'Add todo...' })
          .layoutWeight(1)
          .onChange((value: string) => { this.newTodoText = value; })
        Button('Add')
          .onClick(() => {
            if (this.newTodoText.trim()) {
              this.viewModel.addItem(this.newTodoText);
              this.newTodoText = '';
            }
          })
      }

      // List
      List() {
        ForEach(this.viewModel.items, (item: TodoItem) => {
          ListItem() {
            Row() {
              Text(item.title)
                .decoration(item.done ? { type: TextDecorationType.LineThrough } : {})
              Button(item.done ? 'Undo' : 'Done')
                .onClick(() => { this.viewModel.toggleItem(item.id); })
            }
          }
        }, (item: TodoItem) => item.id.toString())
      }
    }
    .padding(16)
  }
}
```

## State Management Decision Matrix

| Scenario | Recommended Decorator | Why |
|----------|----------------------|-----|
| Simple local counter | `@State` | Local, self-contained |
| Parent passes config to child | `@Prop` | One-way, child gets a copy |
| Child modifies parent's value | `@Link` | Two-way binding |
| Theme/language across many components | `@Provide` + `@Consume` | No prop drilling |
| Global user session, login state | `AppStorage` + `@StorageLink` | App-wide, persistent |
| Page-scoped shared data | `LocalStorage` + `@LocalStorageLink` | Isolated to page tree |
| Object property changes need tracking | `@Observed` + `@ObjectLink` | Deep property observation |
| Fine-grained field tracking | `@Track` | Performance optimization |
