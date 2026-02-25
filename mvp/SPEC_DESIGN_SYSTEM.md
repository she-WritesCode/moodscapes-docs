# Moodscapes Design System

A comprehensive design system that brings the vibrant Moodscapes logo colors to life while maintaining Apple's native design aesthetic.

---

## 🎨 Color Palette

All colors extracted from the Moodscapes logo, converted to HSL for maximum flexibility.

### Primary Brand Colors

| Color | Variable | Hex | Usage |
|-------|----------|-----|-------|
| **Rose** | `--rose-500` | `#CC83A1` | Relationships, warmth, secondary actions |
| **Teal** | `--teal-500` | `#88C2C2` | Information, clarity, calm states |
| **Blue Start** | `--blue-start-500` | `#4B8FB9` | Trust, depth, gradient start |
| **Blue End** | `--blue-end-500` | `#60BDD7` | Progress, primary actions, gradient end |
| **Sage** | `--sage-500` | `#BECA85` | Success, growth, completion |
| **Yellow** | `--yellow-500` | `#EFDB9A` | Attention, warnings, highlights |
| **Peach** | `--peach-500` | `#EFB97C` | Warmth, collaboration, accents |
| **Terracotta** | `--terracotta-500` | `#E1A275` | Grounding, stability |
| **Coral** | `--coral-500` | `#DA8B6E` | Alerts, errors, urgency |
| **Mauve** | `--mauve-500` | `#D0818F` | Romance, softness, elegance |

### Color Variants

Each color has 9 variants (50-900) for different use cases:
- **50-100**: Very light backgrounds, subtle accents
- **200-300**: Light backgrounds, borders, hover states
- **400-500**: Base colors, primary usage
- **600-700**: Darker text, active states
- **800-900**: Very dark, high contrast

**Example usage:**
```css
background-color: hsl(var(--rose-100)); /* Light rose background */
color: hsl(var(--rose-600)); /* Darker rose text */
border-color: hsl(var(--rose-300)); /* Medium rose border */
```

---

## 🌈 Gradients

Pre-defined gradients for dynamic, eye-catching designs:

| Gradient | CSS Variable | Colors | Use Case |
|----------|--------------|--------|----------|
| **Blue** | `--gradient-blue` | Blue Start → Blue End | Progress bars, primary CTAs |
| **Warm** | `--gradient-warm` | Peach → Coral | Exciting moments, celebrations |
| **Cool** | `--gradient-cool` | Teal → Blue End | Calm sections, information |
| **Romantic** | `--gradient-romantic` | Rose → Mauve | Wedding themes, love stories |
| **Natural** | `--gradient-natural` | Sage → Yellow | Organic content, growth |
| **Rainbow** | `--gradient-rainbow` | All colors | Special occasions, onboarding |

**Usage:**
```html
<div class="bg-gradient-romantic p-6 rounded-xl">
  <!-- Romantic gradient background -->
</div>
```

---

## 💭 Mood-Based Theming

Color combinations that evoke specific emotions for wedding planning:

### Excited/Joyful
- **Primary**: Peach (`#EFB97C`)
- **Secondary**: Yellow (`#EFDB9A`)
- **Accent**: Coral Light
- **Use**: Celebration moments, milestone achievements

### Calm/Organized
- **Primary**: Teal (`#88C2C2`)
- **Secondary**: Blue End (`#60BDD7`)
- **Accent**: Blue Start Light
- **Use**: Planning tools, checklists, timelines

### Romantic/Dreamy
- **Primary**: Rose (`#CC83A1`)
- **Secondary**: Mauve (`#D0818F`)
- **Accent**: Rose Light
- **Use**: Love stories, couple profiles, inspiration

### Natural/Grounded
- **Primary**: Sage (`#BECA85`)
- **Secondary**: Terracotta (`#E1A275`)
- **Accent**: Sage Light
- **Use**: Vendor connections, budgeting, practical tools

**Usage:**
```html
<button class="mood-romantic btn-ios-filled">
  Save to Favorites
</button>
```

---

## 🎯 Semantic Color Tokens

Consistent color usage across the app:

| Token | Color | Usage |
|-------|-------|-------|
| `--primary` | Blue End | Primary actions, links, focus states |
| `--secondary` | Rose | Secondary actions, highlights |
| `--success` | Sage | Success messages, completed tasks |
| `--warning` | Yellow | Warnings, important notices |
| `--destructive` | Coral | Errors, delete actions, alerts |
| `--info` | Teal | Information, tips, guidance |

---

## 📝 Typography

Apple-native typography scale using SF Pro (system font):

| Class | Size | Line Height | Weight | Usage |
|-------|------|-------------|--------|-------|
| `.text-ios-large-title` | 34px | 41px | 700 | Page titles |
| `.text-ios-title2` | 22px | 28px | 700 | Section headers |
| `.text-ios-title3` | 20px | 25px | 400 | Subsection headers |
| `.text-ios-body` | 17px | 22px | 400 | Body text, buttons |
| `.text-ios-footnote` | 13px | 18px | 400 | Secondary text, captions |
| `.text-ios-caption` | 11px | 13px | 400 | Timestamps, labels |

**Font Family**: Comfortaa (brand font) for headings, system font for body

---

## 🧩 Component Patterns

### Buttons

**Filled Button (Primary)**
```html
<button class="btn-ios-filled bg-gradient-blue text-white">
  Continue
</button>
```

**Tinted Button (Secondary)**
```html
<button class="btn-ios-tinted">
  Skip for Now
</button>
```

**Color Variants**
```html
<button class="btn-ios-filled bg-rose text-white">Rose Button</button>
<button class="btn-ios-filled bg-sage text-foreground">Sage Button</button>
<button class="btn-ios-filled bg-coral text-white">Coral Button</button>
```

### Cards

**Basic iOS Card**
```html
<div class="card-ios p-4">
  <h3 class="text-ios-title3">Card Title</h3>
  <p class="text-ios-body">Card content</p>
</div>
```

**Colored Card Backgrounds**
```html
<div class="card-ios bg-peach-light p-4">
  <h3 class="text-peach">Warm Welcome</h3>
</div>
```

**Gradient Card**
```html
<div class="card-ios bg-gradient-romantic p-6 text-white">
  <h2 class="text-ios-title2">Your Love Story</h2>
</div>
```

### Inputs

**Standard Input**
```html
<input type="text" class="input-ios w-full" placeholder="Enter your name">
```

**Colored Focus State**
```html
<input type="email" class="input-ios w-full focus:border-rose" placeholder="Email">
```

### Badges

**Status Badges**
```html
<span class="badge-ios bg-sage text-white">Completed</span>
<span class="badge-ios bg-yellow text-foreground">Pending</span>
<span class="badge-ios bg-coral text-white">Urgent</span>
```

**Mood Badges**
```html
<span class="badge-ios mood-excited">Excited</span>
<span class="badge-ios mood-calm">Organized</span>
<span class="badge-ios mood-romantic">Dreamy</span>
```

### Lists

**iOS Grouped List**
```html
<div class="list-group-ios">
  <div class="p-4 bg-white">
    <p class="text-ios-body">List Item 1</p>
  </div>
  <div class="divider-ios"></div>
  <div class="p-4 bg-white">
    <p class="text-ios-body">List Item 2</p>
  </div>
</div>
```

### Glassmorphism

**Modern iOS Glass Effect**
```html
<div class="glass-ios rounded-2xl p-6">
  <h3 class="text-ios-title3">Floating Card</h3>
</div>
```

---

## ✨ Animations

### Spring Animation (Apple-style)
```html
<div class="animate-spring">
  <!-- Content appears with spring effect -->
</div>
```

### Smooth Transitions
```html
<button class="transition-ios hover:scale-105">
  Hover Me
</button>
```

---

## 📐 Spacing & Layout

Following iOS design guidelines:

- **Padding**: Use multiples of 4px (4, 8, 12, 16, 20, 24, 32, 40, 48)
- **Border Radius**: 
  - Small elements: `8px`
  - Cards/Buttons: `12px`
  - Large containers: `16px` - `20px`
- **Shadows**: Subtle, iOS-native shadows
  - Cards: `0 1px 3px rgba(0, 0, 0, 0.1)`
  - Elevated: `0 4px 12px rgba(0, 0, 0, 0.15)`

---

## 🎨 Usage Examples

### Feature Section Colors

```html
<!-- Timeline/Planning -->
<section class="bg-gradient-blue text-white">
  <h2>Your Wedding Timeline</h2>
</section>

<!-- Budget -->
<section class="bg-sage-light">
  <h2 class="text-sage">Budget Tracker</h2>
</section>

<!-- Guest List -->
<section class="bg-rose-light">
  <h2 class="text-rose">Guest Management</h2>
</section>

<!-- Vendors -->
<section class="bg-peach-light">
  <h2 class="text-peach">Find Vendors</h2>
</section>
```

### Progress Indicators

```html
<!-- Progress Bar -->
<div class="w-full bg-teal-100 rounded-full h-2">
  <div class="bg-gradient-blue h-2 rounded-full" style="width: 60%"></div>
</div>

<!-- Circular Progress -->
<div class="w-16 h-16 rounded-full border-4 border-teal-200 border-t-blue-end animate-spin"></div>
```

### Status Indicators

```html
<div class="flex items-center gap-2">
  <div class="w-2 h-2 rounded-full bg-sage"></div>
  <span class="text-sage text-ios-footnote">On Track</span>
</div>

<div class="flex items-center gap-2">
  <div class="w-2 h-2 rounded-full bg-yellow"></div>
  <span class="text-yellow text-ios-footnote">Needs Attention</span>
</div>

<div class="flex items-center gap-2">
  <div class="w-2 h-2 rounded-full bg-coral"></div>
  <span class="text-coral text-ios-footnote">Overdue</span>
</div>
```

---

## ♿ Accessibility

- **Color Contrast**: All text colors meet WCAG AA standards
- **Focus States**: Clear focus indicators using `--ring` color
- **Touch Targets**: Minimum 44x44px for all interactive elements
- **Semantic HTML**: Use proper heading hierarchy and ARIA labels

---

## 🚀 Quick Start

1. **Use semantic tokens** for consistent theming:
   ```html
   <button class="bg-primary text-primary-foreground">Primary Action</button>
   ```

2. **Apply mood-based colors** for emotional resonance:
   ```html
   <div class="mood-romantic p-6">Romantic content</div>
   ```

3. **Leverage gradients** for visual impact:
   ```html
   <header class="bg-gradient-rainbow">Rainbow header</header>
   ```

4. **Maintain Apple native feel** with iOS utilities:
   ```html
   <div class="card-ios transition-ios animate-spring">
     Native-feeling card
   </div>
   ```

---

## 💡 Best Practices

1. **Use light backgrounds** (`-50`, `-100`) for large sections
2. **Reserve gradients** for hero sections and CTAs
3. **Combine mood colors** thoughtfully to avoid overwhelming users
4. **Maintain consistency** by using semantic tokens
5. **Test on devices** to ensure colors look great on different screens
6. **Consider context**: Romantic colors for couple features, calm colors for planning tools
7. **Accessibility first**: Always check color contrast ratios

---

*Design system created for Moodscapes wedding planning app, combining vibrant brand colors with Apple's native design principles.*
