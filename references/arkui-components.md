# ArkUI Components Guide

## Layout Components

### Column
Vertical layout. Children stacked top to bottom.

```typescript
Column({ space: 16 }) {           // space: gap between children
  Text('First').fontSize(16)
  Text('Second').fontSize(16)
}
.width('100%')
.height('100%')
.alignItems(HorizontalAlign.Center)    // Cross-axis alignment
.justifyContent(FlexAlign.Start)       // Main-axis alignment
.padding(16)
```

| HorizontalAlign | Description |
|-----------------|-------------|
| `Start` | Left aligned (default) |
| `Center` | Center aligned |
| `End` | Right aligned |

### Row
Horizontal layout. Children stacked left to right.

```typescript
Row({ space: 12 }) {
  Image($r('app.media.avatar')).width(40).height(40)
  Column() {
    Text('Alice').fontSize(16)
    Text('Developer').fontSize(12).fontColor('#999')
  }
}
.alignItems(VerticalAlign.Center)       // Cross-axis alignment
.justifyContent(FlexAlign.SpaceBetween) // Main-axis alignment
```

| VerticalAlign | Description |
|---------------|-------------|
| `Top` | Top aligned |
| `Center` | Center aligned (default) |
| `Bottom` | Bottom aligned |

### Flex
Flexible layout with wrap support.

```typescript
Flex({
  direction: FlexDirection.Row,          // Row | RowReverse | Column | ColumnReverse
  wrap: FlexWrap.Wrap,                   // NoWrap | Wrap | WrapReverse
  justifyContent: FlexAlign.SpaceAround,
  alignItems: ItemAlign.Stretch,
  space: { main: 8, cross: 8 }          // Gap (main/cross axis)
}) {
  Text('1').padding(8).backgroundColor(Color.Gray)
  Text('2').padding(8).backgroundColor(Color.Gray)
  Text('3').padding(8).backgroundColor(Color.Gray)
}
```

### Stack
Overlay layout. Children stacked on top of each other.

```typescript
Stack({ alignContent: Alignment.Center }) {
  Image($r('app.media.background'))
    .width('100%')
    .height(200)
  Text('Overlay')
    .fontSize(24)
    .fontColor(Color.White)
}
.width('100%')
.height(200)
```

### GridRow / GridCol (Responsive)

```typescript
GridRow({
  breakpoints: {
    value: ['600vp', '840vp'],
    reference: BreakpointsReference.WindowSize
  }
}) {
  GridCol({ span: { xs: 12, sm: 12, md: 6, lg: 4 } }) {
    Card('Item 1')
  }
  GridCol({ span: { xs: 12, sm: 12, md: 6, lg: 4 } }) {
    Card('Item 2')
  }
  GridCol({ span: { xs: 12, sm: 12, md: 12, lg: 4 } }) {
    Card('Item 3')
  }
}
```

| Breakpoint | Width Range |
|------------|-------------|
| `xs` | < 600vp |
| `sm` | 600vp - 840vp |
| `md` | 840vp - 1024vp (not default, customizable) |
| `lg` | > 1024vp (not default, customizable) |

## Basic Components

### Text

```typescript
Text('Hello World')
  .fontSize(16)
  .fontWeight(FontWeight.Bold)       // Normal | Bold | Bolder | Lighter | 100-900
  .fontColor('#333333')
  .textAlign(TextAlign.Center)       // Start | Center | End
  .lineHeight(24)
  .maxLines(2)
  .textOverflow({ overflow: TextOverflow.Ellipsis })  // Clip | Ellipsis | None
  .letterSpacing(0.5)
  .decoration({ type: TextDecorationType.Underline, color: '#333' })
  .copyOption(CopyOptions.InApp)     // None | InApp | LocalDevice
```

### Rich Text (Span)

```typescript
Text() {
  Span('Normal text ').fontSize(16)
  Span('bold text').fontSize(16).fontWeight(FontWeight.Bold)
  Span(' and ')
  Span('colored').fontColor(Color.Red).fontSize(16)
  Image($r('app.media.icon')).width(16).height(16)
}
```

### Image

```typescript
Image($r('app.media.photo'))
  .width(200)
  .height(200)
  .objectFit(ImageFit.Cover)          // Contain | Cover | Auto | Fill | ScaleDown | None
  .borderRadius(12)
  .alt($r('app.media.placeholder'))   // Fallback image
  .interpolation(ImageInterpolation.High)  // Image smoothing
  .draggable(false)
```

### Button

```typescript
// Text button
Button('Submit')
  .width('100%')
  .height(48)
  .fontSize(16)
  .fontWeight(FontWeight.Medium)
  .type(ButtonType.Capsule)           // Capsule | Circle | Normal
  .backgroundColor($r('app.color.primary'))
  .fontColor(Color.White)
  .onClick(() => {})

// Icon button
Button({ type: ButtonType.Circle }) {
    Image($r('app.media.add')).width(24).height(24)
  }
  .width(48)
  .height(48)

// Custom content button
Button({ type: ButtonType.Normal }) {
    Row({ space: 8 }) {
      Image($r('app.media.download')).width(20).height(20)
      Text('Download').fontSize(14)
    }
  }
  .padding({ left: 24, right: 24, top: 12, bottom: 12 })
  .borderRadius(24)
```

### TextInput

```typescript
TextInput({ placeholder: 'Enter email' })
  .width('100%')
  .height(48)
  .fontSize(16)
  .type(InputType.EmailAddress)        // Normal | Number | PhoneNumber | EmailAddress | Password
  .placeholderColor('#999999')
  .placeholderFont({ size: 16 })
  .caretColor($r('app.color.primary'))
  .maxLength(50)
  .enterKeyType(EnterKeyType.Done)     // Go | Search | Send | Next | Done
  .onChange((value: string) => { })
  .onSubmit((value: string) => { })
  .onFocus(() => { })
  .onBlur(() => { })
```

### TextArea (Multi-line)

```typescript
TextArea({ placeholder: 'Write your message...' })
  .width('100%')
  .height(120)
  .fontSize(16)
  .maxLength(500)
  .showCounter(true)                   // Show character counter
  .onChange((value: string) => { })
```

### Toggle

```typescript
Toggle({ type: ToggleType.Switch, isOn: this.isDarkMode })
  .onChange((isOn: boolean) => {
    this.isDarkMode = isOn;
  })

Toggle({ type: ToggleType.Checkbox, isOn: this.isChecked })
  .onChange((isOn: boolean) => { })

// Custom toggle group
Toggle({ type: ToggleType.Button }) {
    Text('Option').fontSize(14)
  }
  .selectedColor($r('app.color.primary'))
```

### Slider

```typescript
Slider({
  value: this.volume,
  min: 0,
  max: 100,
  step: 1,
  style: SliderStyle.OutSet           // OutSet | InSet
})
  .width('100%')
  .blockColor($r('app.color.primary'))
  .trackColor('#E5E5E5')
  .selectedColor($r('app.color.primary'))
  .onChange((value: number, mode: SliderChangeMode) => {
    this.volume = Math.floor(value);
  })
```

### Progress

```typescript
// Linear progress
Progress({ value: 75, total: 100, type: ProgressType.Linear })
  .width('100%')
  .color($r('app.color.primary'))
  .backgroundColor('#E5E5E5')

// Circular progress
Progress({ value: 75, total: 100, type: ProgressType.Circular })
  .width(60)
  .height(60)

// Loading indicator
LoadingProgress()
  .width(40)
  .height(40)
  .color($r('app.color.primary'))
```

## Container Components

### List

```typescript
List({ space: 8, initialIndex: 0 }) {
  LazyForEach(this.dataSource, (item: Item) => {
    ListItem() {
      Row() {
        Text(item.title).fontSize(16)
      }
      .padding(16)
      .backgroundColor(Color.White)
      .borderRadius(12)
    }
  }, (item: Item) => item.id.toString())
}
.width('100%')
.height('100%')
.listDirection(Axis.Vertical)         // Vertical | Horizontal
.divider({ strokeWidth: 1, color: '#E5E5E5', startMargin: 16, endMargin: 16 })
.scrollBar(BarState.Auto)
.onScroll((scrollOffset: number, scrollState: ScrollState) => { })
.onReachEnd(() => { /* load more */ })
```

### Grid

```typescript
Grid() {
  ForEach(this.items, (item: string) => {
    GridItem() {
      Text(item).fontSize(14).padding(12)
    }
  }, (item: string) => item)
}
.columnsTemplate('1fr 1fr 1fr')       // 3 columns
.rowsTemplate('1fr 1fr')              // 2 rows (optional)
.columnsGap(8)
.rowsGap(8)
.width('100%')
.height('100%')
```

### Tabs

```typescript
Tabs({ barPosition: BarPosition.Start, index: this.currentIndex }) {
  TabContent() {
    Text('Content 1')
  }
  .tabBar('Tab 1')

  TabContent() {
    Text('Content 2')
  }
  .tabBar('Tab 2')

  TabContent() {
    Text('Content 3')
  }
  .tabBar('Tab 3')
}
.onChange((index: number) => {
  this.currentIndex = index;
})
.scrollable(true)
```

### ScrollView

```typescript
Scroll() {
  Column() {
    Text('Content block 1').padding(20)
    Text('Content block 2').padding(20)
    Text('Content block 3').padding(20)
  }
}
.width('100%')
.height('100%')
.scrollBar(BarState.Auto)
.onScrollEdge((side: Edge) => { })
```

## Navigation Components

### Router (Page-based navigation)

```typescript
import router from '@ohos.router';

// Push page onto stack
router.pushUrl({ url: 'pages/Detail' });

// Push with params
router.pushUrl({
  url: 'pages/Detail',
  params: { id: 42, title: 'Product' }
});

// Replace current page (no back)
router.replaceUrl({ url: 'pages/Home' });

// Replace with params
router.replaceUrl({
  url: 'pages/Login',
  params: { reason: 'expired' }
});

// Go back
router.back();
router.back({ url: 'pages/Home' });  // Back to specific page

// Get params on target page
let params = router.getParams() as Record<string, Object>;
let id = params['id'] as number;
let title = params['title'] as string;

// Check stack length
let stackSize = router.getLength();

// Clear stack
router.clear();
```

### Navigation Component (Advanced)

```typescript
@Component
struct NavigationExample {
  @Provide('navPathStack') navPathStack: NavPathStack = new NavPathStack();

  @Builder
  pageMap(name: string) {
    if (name === 'detail') {
      DetailPage()
    } else if (name === 'settings') {
      SettingsPage()
    }
  }

  build() {
    Navigation(this.navPathStack) {
      // Home page content
      Column() {
        Text('Home')
        Button('Go to Detail')
          .onClick(() => {
            this.navPathStack.pushPath({ name: 'detail' });
          })
      }
    }
    .navDestination(this.pageMap)
    .mode(NavigationMode.Stack)
    .title('My App')
    .titleMode(NavigationTitleMode.Mini)
  }
}

@Component
struct DetailPage {
  @Consume('navPathStack') navPathStack: NavPathStack;

  build() {
    NavDestination() {
      Column() {
        Text('Detail Page')
        Button('Back')
          .onClick(() => { this.navPathStack.pop(); })
      }
    }
    .title('Detail')
  }
}
```

## Dialog Components

### AlertDialog

```typescript
AlertDialog.show({
  title: 'Delete Item',
  message: 'Are you sure you want to delete this item?',
  autoCancel: true,
  alignment: DialogAlignment.Center,
  offset: { dx: 0, dy: -20 },
  primaryButton: {
    value: 'Cancel',
    action: () => { console.info('Cancelled'); }
  },
  secondaryButton: {
    value: 'Delete',
    fontColor: Color.Red,
    action: () => {
      // Delete logic
      console.info('Deleted');
    }
  },
  cancel: () => { console.info('Dialog dismissed'); }
});
```

### ActionSheet

```typescript
ActionSheet.show({
  title: 'Choose Action',
  message: 'Select an option',
  autoCancel: true,
  sheets: [
    {
      title: 'Camera',
      action: () => { /* open camera */ }
    },
    {
      title: 'Gallery',
      action: () => { /* open gallery */ }
    },
    {
      title: 'Cancel',
      action: () => { }
    }
  ]
});
```

### Toast

```typescript
import { promptAction } from '@kit.ArkUI';

// Simple toast
promptAction.showToast({
  message: 'Saved successfully',
  duration: 2000,
  bottom: 100
});

// Toast with options
promptAction.showToast({
  message: 'Hello',
  duration: 3000,
  showMode: ToastShowMode.TOP,    // DEFAULT | TOP
  alignment: Alignment.Center
});
```

## Custom Components

### Full Custom Component Example

```typescript
interface CardProps {
  title: string;
  subtitle?: string;
  onCardClick?: () => void;
}

@Component
struct InfoCard {
  @Prop title: string = '';
  @Prop subtitle: string = '';
  onCardClick: () => void = () => {};

  build() {
    Column() {
      Text(this.title)
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .fontColor('#333333')

      if (this.subtitle) {
        Text(this.subtitle)
          .fontSize(14)
          .fontColor('#999999')
          .margin({ top: 4 })
      }
    }
    .padding(16)
    .borderRadius(12)
    .backgroundColor(Color.White)
    .shadow({
      radius: 8,
      color: 'rgba(0, 0, 0, 0.08)',
      offsetX: 0,
      offsetY: 2
    })
    .onClick(() => { this.onCardClick(); })
  }
}

// Usage
@Component
struct HomePage {
  build() {
    Column({ space: 12 }) {
      InfoCard({
        title: 'Total Sales',
        subtitle: '$12,345 this month',
        onCardClick: () => { router.pushUrl({ url: 'pages/Detail' }); }
      })
      InfoCard({
        title: 'Active Users',
        subtitle: '1,234 online now'
      })
    }
    .padding(16)
  }
}
```

## Animation

### Property Animation

```typescript
@State scaleValue: number = 1;
@State opacityValue: number = 1;

build() {
  Image($r('app.media.photo'))
    .width(100)
    .height(100)
    .scale({ x: this.scaleValue, y: this.scaleValue })
    .opacity(this.opacityValue)
    .animation({
      duration: 300,
      curve: Curve.EaseInOut,        // Linear | Ease | EaseIn | EaseOut | EaseInOut | FastOutSlowIn
      delay: 0,
      iterations: 1,
      playMode: PlayMode.Normal      // Normal | Reverse | Alternate | AlternateReverse
    })
    .onClick(() => {
      this.scaleValue = this.scaleValue === 1 ? 1.5 : 1;
      this.opacityValue = this.opacityValue === 1 ? 0.5 : 1;
    })
}
```

### animateTo (Explicit Animation)

```typescript
@State translateX: number = 0;

build() {
  Row() {
    Text('Slide me')
  }
  .translate({ x: this.translateX })
  .onClick(() => {
    animateTo({
      duration: 500,
      curve: Curve.EaseOut,
      onFinish: () => { console.info('Animation done'); }
    }, () => {
      this.translateX = this.translateX === 0 ? 200 : 0;
    });
  })
}
```

### Transition Animations

```typescript
@State isVisible: boolean = false;

build() {
  Column() {
    Button('Toggle')
      .onClick(() => { this.isVisible = !this.isVisible; })

    if (this.isVisible) {
      Text('Animated content')
        .padding(20)
        .backgroundColor(Color.White)
        .transition({
          type: TransitionType.All,
          opacity: 0,
          translate: { y: 50 }
        })
    }
  }
}
```
