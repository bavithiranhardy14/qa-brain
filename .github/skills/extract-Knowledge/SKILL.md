---
name: extract-knowledge
description: 'Extract and build QA knowledge across API, Web UI, Android, and iOS projects. Use when: scanning project paths, analysing monoliths or monorepos, identifying product context and business rules, or generating merged QA knowledge files.'
---

## Quick-Start for Developers

### Step 1 — Identify what you have

Run this in your terminal to list the top-level contents of your project:

```bash
# Windows (PowerShell)
Get-ChildItem "D:\your-project" -Depth 1 | Select-Object Name, PSIsContainer

# Mac / Linux
ls -la /path/to/your-project
find /path/to/your-project -maxdepth 2 -name "package.json" -o -name "build.gradle" -o -name "Podfile" -o -name "pom.xml" -o -name "*.csproj" | sort
```

Look for these indicator files to identify which platforms are present:

| File you find | Platform present |
|---|---|
| `package.json` + `src/routes/` or `src/controllers/` | **API** (Node.js) |
| `package.json` + `src/pages/` or `src/views/` or `App.tsx` | **Web UI** |
| `package.json` (Next.js `app/` or `pages/` dir) | **Web UI** (Next.js) |
| `pom.xml` or `build.gradle` without `android { }` block | **API** (Java/Kotlin Spring) |
| `requirements.txt` or `pyproject.toml` | **API** (Python) |
| `*.csproj` or `*.sln` | **API** (.NET) |
| `go.mod` | **API** (Go) |
| `Gemfile` | **API** (Ruby on Rails) |
| `build.gradle` or `build.gradle.kts` **with** `android { }` block | **Android** |
| `AndroidManifest.xml` | **Android** |
| `*.xcodeproj` or `*.xcworkspace` or `Podfile` or `Package.swift` | **iOS** |
| `Info.plist` | **iOS** |

> **Example — what a full-stack monorepo looks like after running the terminal command:**
>
> ```
> ShopEase/
> ├── api/          package.json + src/controllers/    → API (Node.js / Express)
> ├── web/          package.json + src/pages/           → Web UI (React + Vite)
> ├── android/      build.gradle with android { }       → Android (Kotlin)
> ├── ios/          ShopEase.xcworkspace + Podfile       → iOS (Swift)
> └── README.md
> ```
> Four platforms detected — use the "Full stack (all 4)" invocation below.

---

### Step 2 — Choose your invocation

| Your project structure | What to type |
|---|---|
| **Single folder — all in one** | `"Extract knowledge from this project at <path>"` |
| **Monorepo — subfolders per platform** | `"Extract knowledge from this monorepo at <path>"` — skill auto-detects subfolders |
| **Separate repos — you know the paths** | `"Extract knowledge — API at <p1>, Web at <p2>, Android at <p3>, iOS at <p4>"` |
| **Only API** | `"Extract knowledge from the API at <path>"` |
| **Only Web UI** | `"Extract knowledge from the Web UI at <path>"` |
| **Only Android** | `"Extract knowledge from the Android project at <path>"` |
| **Only iOS** | `"Extract knowledge from the iOS project at <path>"` |
| **API + Web** | `"Extract knowledge — API at <path>, Web UI at <path>"` |
| **API + Android** | `"Extract knowledge — API at <path>, Android at <path>"` |
| **API + iOS** | `"Extract knowledge — API at <path>, iOS at <path>"` |
| **API + Android + iOS** | `"Extract knowledge — API at <p1>, Android at <p2>, iOS at <p3>"` |
| **Full stack (all 4)** | `"Extract knowledge — API at <p1>, Web at <p2>, Android at <p3>, iOS at <p4>"` |

You can omit any platform that does not exist. The skill merges only what is present.

> **Example — full-stack project, type this exactly:**
> ```
> Extract knowledge — API at D:\ShopEase\api, Web at D:\ShopEase\web, Android at D:\ShopEase\android, iOS at D:\ShopEase\ios
> ```
> **Example — monorepo, let the skill find everything:**
> ```
> Extract knowledge from this monorepo at D:\ShopEase
> ```
> **Example — backend only (no mobile yet):**
> ```
> Extract knowledge from the API at D:\ShopEase\api
> ```

---

## Monolith and Monorepo Auto-Detection

When the developer says `"Extract knowledge from this monorepo at <path>"` or points at a single root folder, follow this detection procedure **before** scanning.

### Detection Procedure

**1. List the root folder contents.**

**2. Check for known monorepo tool configuration files:**

| File found at root | Monorepo tool | What to expect |
|---|---|---|
| `nx.json` | Nx | Apps in `apps/` folder; each subfolder is a project |
| `turbo.json` | Turborepo | Apps in `apps/` and packages in `packages/` |
| `lerna.json` | Lerna | Packages listed in `packages/` |
| `pnpm-workspace.yaml` | pnpm workspaces | Packages listed in workspace globs |
| `rush.json` | Rush | Projects listed inside the JSON |
| `settings.gradle` with `include(":app", ":feature-*")` | Android multi-module | Each included module is a sub-project |
| Multiple `*.xcodeproj` at root | iOS multi-target | Each `.xcodeproj` = one target |

**3. If no monorepo tool found, scan root for platform indicator files directly (true monolith):**

Common true-monolith patterns and how to handle them:

| What you find | What it is | How to scan |
|---|---|---|
| `package.json` with both `"react"` AND express/fastify/NestJS in dependencies | Node.js full-stack monolith (e.g. Create React App inside Express, or Next.js with API routes) | Scan as **Web + API combined** — `src/pages/api/` or `src/server/` = API; `src/pages/` or `src/app/` = Web UI |
| `pom.xml` or `build.gradle` with both Spring MVC controllers AND Thymeleaf / JSP templates | Java full-stack | Scan controllers as **API modules**; HTML templates as **Web UI workflows** |
| `*.csproj` with both Controllers and Razor Views | .NET MVC monolith | Controllers = **API**; Razor Views / Blazor components = **Web UI workflows** |
| Django / Rails app with both `views.py` / controllers AND HTML templates | Python/Ruby full-stack | URL routes + views = **API modules**; HTML templates = **Web UI workflows** |
| Single repo with `android/` and `ios/` subfolders (React Native / Flutter) | Cross-platform mobile | `android/` = **Android**; `ios/` = **iOS**; JS/Dart root = **Web UI layer** |
| React Native without `android/` or `ios/` split | RN monorepo | Treat as **Web UI** + note it compiles to both Android and iOS |
| Flutter project (`pubspec.yaml`) | Flutter | `lib/` = shared UI; `android/` = Android config; `ios/` = iOS config |

**4. Build the project map before scanning:**

After detection, write a summary like this before proceeding:

```
Detected platforms:
  API      → path/to/api        (Node.js / NestJS)
  Web UI   → path/to/web        (React + Vite)
  Android  → path/to/android    (Kotlin, minSdk 24)
  iOS      → path/to/ios        (Swift, iOS 15+)

Scanning each platform now...
```

If a platform is ambiguous (e.g. a `package.json` could be API or Web), read the `main` field and `dependencies` to decide, then state your decision before scanning.

> **Example — filled-in project map for ShopEase:**
> ```
> Detected platforms:
>   API      → D:\ShopEase\api       (Node.js 18 / Express 4)
>   Web UI   → D:\ShopEase\web       (React 18 + Vite)
>   Android  → D:\ShopEase\android   (Kotlin, minSdk 26, targetSdk 34)
>   iOS      → D:\ShopEase\ios       (Swift 5.9, iOS 15+, CocoaPods)
>
> Scanning each platform now...
> ```

---

## What Each Platform Contributes — Master Table

| Knowledge Area | API | Web UI | Android | iOS |
|---|---|---|---|---|
| App name + type | ✅ package.json / pom.xml | ✅ package.json | ✅ build.gradle / AndroidManifest | ✅ Info.plist / Package.swift |
| **Modules** | ✅ Controllers / route groups | ✅ Pages / route config | ✅ Activities / Fragments / nav graph | ✅ ViewControllers / SwiftUI Views / nav stack |
| **User roles** | ✅ Auth middleware, role enums, guards, DB seeders | ✅ Route guards, role-based rendering | ✅ Role checks in ViewModel / Repository | ✅ Role checks in ViewModel / Interactor |
| **User workflows** | ⬜ (API steps only) | ✅ Page → form → API call → result | ✅ Screen → user action → API call → state update | ✅ View → user action → API call → state update |
| **Tech stack** | ✅ Backend framework, DB, auth, integrations | ✅ Frontend framework, build tool, UI libs | ✅ Android SDK, libraries, Gradle config | ✅ iOS SDK, CocoaPods / SPM, build config |
| **Business rules** | ✅ Services, use-cases, domain models (authoritative) | ✅ Client-side guards (secondary) | ✅ ViewModel / Repository logic (secondary) | ✅ ViewModel / Interactor logic (secondary) |
| **Validation rules** | ✅ DTOs, Zod/Joi/Yup on server | ✅ Zod/Yup/RHF form schemas | ✅ ViewModel input validation, Room constraints | ✅ ViewModel validation, CoreData / SwiftData constraints |
| **System constraints** | ✅ Constants, env vars, config files | ✅ Frontend env vars, input length caps | ✅ Manifest permissions, build constants | ✅ Info.plist permissions, build settings |
| **Edge cases** | ✅ try/catch, TODO/FIXME, rollback logic | ✅ Error boundaries, empty-state handling | ✅ try/catch in coroutines, error sealed classes | ✅ do/catch, Result types, error enums |

> **Rule:** When the same rule appears in multiple platforms, write it once and mark `Source = Both` or list all platforms (e.g., `API + Android`).

---

## Platform Scan Guides

### Platform 1 — API / Backend

Supports: Node.js (Express / NestJS / Fastify), Python (Django / FastAPI / Flask), Java / Kotlin (Spring), .NET (ASP.NET Core), Ruby on Rails, Go.

**Step 1 — Identify the project type**

| File found | Project type |
|---|---|
| `package.json` | Node.js |
| `requirements.txt` / `pyproject.toml` | Python |
| `pom.xml` / `build.gradle` (no `android` block) | Java / Kotlin Spring |
| `*.csproj` / `*.sln` | .NET |
| `Gemfile` | Ruby on Rails |
| `go.mod` | Go |

**Step 2 — Files to scan and what to extract**

| File / Folder | Extract |
|---|---|
| `package.json` / `requirements.txt` / `pom.xml` / `*.csproj` | App name, backend framework, DB drivers, auth libs, third-party integrations |
| `.env.example` / `config/` / `constants/` / `appsettings.json` | System constraints — every key = one constraint (rate limits, timeouts, max values, feature flags) |
| `src/routes/` / `src/controllers/` / `Controllers/` / `app/Http/Controllers/` | One module per router file or controller class |
| `src/services/` / `src/use-cases/` / `src/domain/` / `Services/` | Business rules — scan every `if`, `throw`, calculation, eligibility check, approval flow |
| `src/validators/` / `src/dtos/` / `src/schemas/` / `Requests/` | Server-side validation — field name + condition + error message |
| `src/middleware/` / `src/guards/` / `Middleware/` / `Filters/` | User roles, JWT claims checked, permission enforcement |
| `src/models/` / `src/entities/` / `Models/` / ORM files | Data constraints: `@IsNotNull`, `unique`, `maxLength`, foreign key rules |
| `src/exceptions/` / `src/errors/` | Named error types = edge cases |
| All `*.test.ts` / `*.spec.py` / `*Test.java` / `*Tests.cs` | Test names and assertions often state business rules explicitly |
| `TODO`, `FIXME`, `HACK` comments anywhere | Known edge cases, deferred logic, known bugs |
| Transaction / try-catch blocks | Rollback conditions = data integrity edge cases |

> **Examples — API code → what gets extracted:**
>
> **Business rules** from `src/services/OrderService.ts`:
> ```typescript
> if (cart.items.length === 0)
>   throw new BadRequestException('Cart cannot be empty');
> if (order.totalAmount > 50000)
>   throw new ForbiddenException('Single order limit is ₹50,000');
> if (user.kycStatus !== 'verified' && order.totalAmount > 10000)
>   throw new ForbiddenException('KYC required for orders above ₹10,000');
> ```
> → Extracted as:
> | BR-01 | Cart must contain at least one item before checkout | Order | API | High |
> | BR-02 | Single order total cannot exceed ₹50,000 | Order | API | High |
> | BR-03 | KYC verification required for orders above ₹10,000 | Order | API | High |
>
> **Validation rules** from `src/dtos/CreateOrderDto.ts`:
> ```typescript
> @IsNotEmpty({ message: 'Delivery address is required' })
> @MaxLength(250, { message: 'Address too long' })
> deliveryAddress: string;
>
> @IsIn(['card', 'upi', 'cod'], { message: 'Invalid payment method' })
> paymentMethod: string;
> ```
> → Extracted as:
> | `deliveryAddress` | Required, max 250 chars | "Delivery address is required" / "Address too long" | API |
> | `paymentMethod` | Must be one of: card, upi, cod | "Invalid payment method" | API |
>
> **System constraints** from `.env.example`:
> ```
> RATE_LIMIT_REQUESTS_PER_MINUTE=100
> MAX_CART_ITEMS=50
> OTP_EXPIRY_SECONDS=300
> SESSION_EXPIRY_HOURS=24
> ```
> → Extracted as:
> - API rate limit: 100 requests per minute — Source: API `RATE_LIMIT_REQUESTS_PER_MINUTE`
> - Max items in cart: 50 — Source: API `MAX_CART_ITEMS`
> - OTP valid for 300 seconds — Source: API `OTP_EXPIRY_SECONDS`

---

### Platform 2 — Web UI

Supports: React, Next.js, Angular, Vue / Nuxt, Svelte.

**Step 1 — Identify the framework**

| File found | Framework |
|---|---|
| `package.json` → `"react"` | React / Next.js |
| `package.json` → `"@angular/core"` | Angular |
| `package.json` → `"vue"` | Vue / Nuxt |
| `package.json` → `"svelte"` | Svelte |

**Step 2 — Files to scan and what to extract**

| File / Folder | Extract |
|---|---|
| `package.json` | Frontend framework, build tool (Vite/Webpack), UI component libraries, state management, form libraries |
| `src/router/` / `src/routes/` / `App.tsx` / `pages/` (Next.js) / `app/` (Next.js App Router) | Module list — one module per top-level route group or page folder |
| `src/pages/` / `src/views/` / `src/screens/` / `app/**/page.tsx` | User workflows — for each page trace: page load → user action → API call (`fetch`/`axios`/`useQuery`) → result rendered |
| Form schema files: `*.schema.ts`, `*.validation.ts`, Zod schemas (`z.object`), Yup schemas (`yup.object`) | Client-side validation rules — field + condition + error message |
| `useForm`, `react-hook-form`, `Formik` usage inside components | Additional validation constraints not in a schema file |
| `src/hooks/useAuth*` / `AuthContext` / `ProtectedRoute` / Angular `CanActivate` guards | User roles, route-level access control |
| `src/store/` / `src/context/` / NgRx / Pinia / Vuex | State-dependent business logic visible in UI (e.g., only show X if user has role Y) |
| `.env.example` / `src/config/` / `environment.ts` | Frontend-specific constraints: `MAX_FILE_SIZE`, input limits, feature flags |
| `src/services/` / `src/api/` / `src/lib/api*` | API endpoint URLs — map each call to the corresponding workflow step |

> **Examples — Web UI code → what gets extracted:**
>
> **Validation rules** from `src/schemas/checkoutSchema.ts`:
> ```typescript
> export const checkoutSchema = z.object({
>   pincode: z.string()
>     .length(6, 'Pincode must be exactly 6 digits')
>     .regex(/^\d+$/, 'Pincode must be numeric'),
>   phone: z.string()
>     .regex(/^[6-9]\d{9}$/, 'Enter a valid 10-digit mobile number'),
>   couponCode: z.string().max(20, 'Coupon code too long').optional(),
> });
> ```
> → Extracted as:
> | `pincode` | Exactly 6 numeric digits | "Pincode must be exactly 6 digits" | Web |
> | `phone` | 10 digits, starts with 6–9 | "Enter a valid 10-digit mobile number" | Web |
>
> **User workflow** traced from `src/pages/CheckoutPage.tsx`:
> ```tsx
> function CheckoutPage() {
>   const { data: cart } = useQuery(['cart'], fetchCart);        // step 1
>   const { mutate: placeOrder } = useMutation(createOrder, {   // API call
>     onSuccess: () => navigate('/order-confirmed'),             // step 5
>   });
>   // form: deliveryAddress, pincode, phone, paymentMethod
> }
> ```
> → Workflow extracted:
> **Workflow: Place an Order** — Entry: `/checkout` — Triggered by: "Proceed to Checkout" click
> 1. User reviews cart items on Checkout page; cart loaded via GET /api/cart
> 2. User fills address, pincode, phone, payment method; Zod schema validates client-side
> 3. POST /api/orders with cart + delivery + payment
> 4. API validates rules (BR-01, BR-02, BR-03), creates order, deducts stock
> 5. Navigate to /order-confirmed with order ID

---

### Platform 3 — Android

Supports: Kotlin (preferred) or Java, Gradle build system, MVVM / MVI / MVP architecture.

**Step 1 — Identify the project**

| File found | Meaning |
|---|---|
| `build.gradle` or `build.gradle.kts` with `android { }` block | Android project root |
| `AndroidManifest.xml` | Entry point for permissions, activities, deep links |
| `settings.gradle` / `settings.gradle.kts` | Module list (app, feature modules, shared libs) |

**Step 2 — Files to scan and what to extract**

| File / Folder | Extract |
|---|---|
| `app/build.gradle` / `build.gradle.kts` | App name (`applicationId`), SDK versions (`minSdk`, `targetSdk`), build types, dependencies (Retrofit, Room, Hilt/Dagger, Coroutines, Compose, etc.) |
| `AndroidManifest.xml` | `<uses-permission>` = system constraints; `<activity>` names = module list; `<intent-filter>` deep links = entry points; `android:exported` = security constraints |
| `res/navigation/` / `nav_graph.xml` / Jetpack Navigation Kotlin DSL | All screens = module list; navigation edges = user workflow steps |
| `*ViewModel.kt` / `*ViewModel.java` | Business rules enforced on client — validation logic, state machines, eligibility checks in `viewModelScope` |
| `*Repository.kt` / `*Repository.java` | API calls (Retrofit), local DB calls (Room), caching rules = system constraints |
| `*UseCase.kt` / `*Interactor.kt` | Business rules — same as API use-cases but client-side |
| `*Dao.kt` / `*Entity.kt` (Room) | Data constraints: `@PrimaryKey`, `@NotNull`, `UNIQUE`, `FOREIGN KEY` |
| `Retrofit` interface files (`*Api.kt` / `*Service.kt`) | API endpoint URLs → maps to workflow steps |
| `res/values/strings.xml` | Error messages, validation text shown to user |
| `*Fragment.kt` / `*Activity.kt` / Composable functions | User workflow steps — trace user interaction (`onClick`, `onSubmit`) through ViewModel to Repository to API |
| `build.gradle` `buildConfigField` entries | System constraints from build config |
| All `*Test.kt` / `*Test.java` files in `test/` and `androidTest/` | Business rules and edge cases stated explicitly in test assertions |
| `TODO`, `FIXME` comments | Known edge cases |

> **Examples — Android code → what gets extracted:**
>
> **Business rules** from `OrderViewModel.kt`:
> ```kotlin
> fun submitOrder(cart: Cart) {
>     viewModelScope.launch {
>         if (cart.items.isEmpty()) {
>             _uiState.value = UiState.Error("Add at least one item to cart")
>             return@launch
>         }
>         if (cart.totalAmount > 50_000) {
>             _uiState.value = UiState.Error("Order total cannot exceed ₹50,000")
>             return@launch
>         }
>         val result = orderRepository.placeOrder(cart)
>         _uiState.value = when (result) {
>             is Result.Success -> UiState.OrderPlaced(result.data)
>             is Result.Error   -> UiState.Error(result.message)
>         }
>     }
> }
> ```
> → BR-01 and BR-02 also enforced Android-side → mark `Source = API + Android`
>
> **Modules + system constraints** from `AndroidManifest.xml`:
> ```xml
> <uses-permission android:name="android.permission.CAMERA" />
> <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
> <activity android:name=".checkout.CheckoutActivity" />
> <activity android:name=".orders.OrderHistoryActivity" />
> ```
> → Modules: Checkout, Order History
> → System constraints: Camera permission required; Location (fine) permission required

---

### Platform 4 — iOS

Supports: Swift (preferred) or Objective-C, CocoaPods / Swift Package Manager, MVVM / VIPER / Clean Architecture.

**Step 1 — Identify the project**

| File found | Meaning |
|---|---|
| `*.xcodeproj` / `*.xcworkspace` | Xcode project root |
| `Podfile` | CocoaPods dependency manager |
| `Package.swift` | Swift Package Manager |
| `Info.plist` | App metadata, permissions, URL schemes |

**Step 2 — Files to scan and what to extract**

| File / Folder | Extract |
|---|---|
| `Podfile` / `Package.swift` | Frontend framework (UIKit / SwiftUI), networking library (Alamofire / URLSession), persistence (CoreData / SwiftData / Realm), auth (Auth0 / Firebase Auth / Keychain), analytics |
| `Info.plist` | `NSCameraUsageDescription` etc. = system constraints (permissions); `CFBundleURLTypes` = deep link entry points; background modes = offline capability |
| `*AppDelegate.swift` / `*SceneDelegate.swift` / `@main App` struct | App-level configuration, initial navigation setup |
| Navigation: `UINavigationController`, `UITabBarController`, `SwiftUI NavigationStack`, `Coordinator*.swift` | Module list (one module per tab / coordinator); workflow entry points |
| `*ViewController.swift` / SwiftUI `View` files | User workflow steps — trace `@IBAction` / button tap / form submit → ViewModel call → API call → UI state update |
| `*ViewModel.swift` / `*Presenter.swift` / `*Interactor.swift` | Business rules on client — validation, eligibility checks, state transitions |
| `*Repository.swift` / `*Service.swift` / `*APIClient.swift` | API calls (Alamofire / URLSession) — endpoint URLs map to workflow steps |
| `*CoreDataModel.xcdatamodeld` / SwiftData `@Model` classes | Data constraints: required attributes, relationships, unique constraints |
| `Localizable.strings` / `*.lproj/` | Error messages and validation text shown to user |
| `Result<_, Error>` types / custom `Error` enums | Named error cases = edge cases |
| All `*Tests.swift` / `*Spec.swift` files | Business rules and edge cases stated explicitly in test assertions |
| `TODO`, `FIXME`, `MARK:` comments | Known edge cases and deferred logic |
| Keychain usage (`SecItemAdd`, `KeychainAccess`) | Security constraints |

> **Examples — iOS code → what gets extracted:**
>
> **Business rules + validation** from `CheckoutViewModel.swift`:
> ```swift
> func validateOrder() -> String? {
>     guard !cart.items.isEmpty
>       else { return "Add at least one item to continue" }
>     guard cart.totalAmount <= 50_000
>       else { return "Order total cannot exceed ₹50,000" }
>     guard pincode.count == 6, pincode.allSatisfy({ $0.isNumber })
>       else { return "Enter a valid 6-digit pincode" }
>     return nil
> }
> ```
> → BR-01 and BR-02 also enforced iOS-side → mark `Source = API + Android + iOS`
> → Validation: `pincode` must be 6 numeric digits → mark `Enforced In = Multiple`
>
> **System constraints** from `Info.plist`:
> ```xml
> <key>NSCameraUsageDescription</key>
> <string>Used to scan QR codes for payments</string>
> <key>NSLocationWhenInUseUsageDescription</key>
> <string>Used to suggest nearby delivery addresses</string>
> ```
> → System constraints:
> - Camera permission required — scan QR codes for payments — Source: iOS `Info.plist`
> - Location (when in use) permission required — suggest delivery addresses — Source: iOS `Info.plist`

---

## Merge and Write Output Files

After scanning all provided project paths, combine all findings into two files.

### Output File 1 — `knowledge/product_context.md`

```markdown
# Product Context

## Application Overview
**Application Name:** [from API package.json / Android applicationId / iOS CFBundleDisplayName]
**Type:** [list every platform present — e.g. REST API (Node.js) + React Web App + Android App + iOS App]
**Primary Users:** [merged from API role enums + mobile role checks + UI route guards]

[2–3 sentence description of what the product does, derived from README files and app descriptions found across all projects]

---

## Modules

| Module | Description | Primary Roles | Found In |
|--------|-------------|---------------|----------|
| [Name] | [what it does] | [role] | API / Web / Android / iOS / All |

Rules:
- One row per distinct feature area.
- If the same feature exists across multiple platforms, merge into one row and list all platforms in Found In.
- Derive module names from: controller names (API), page/route names (Web), Activity/Fragment/screen names (Android), ViewController/View names (iOS), nav graph node names (Android), coordinator destinations (iOS).

---

## User Workflows

### Workflow 1: [Name]
**Platforms:** [Web / Android / iOS / All]
**Entry point:** [URL / screen name / deep link]
**Triggered by:** [user action]

1. [User-facing step — what the user sees and does]
2. [Client-side step — form validation, state change]
3. [API call — method + endpoint]
4. [API processing — service / use-case logic]
5. [Final outcome — DB write, response, screen navigation, push notification]

Rules:
- Derive step 1–2 from UI page components / Android Fragments / iOS ViewControllers.
- Derive step 3 from the API call in the service / repository layer.
- Derive step 4 from the API service/controller/use-case.
- Derive step 5 from the API response + client-side navigation after success.
- Write one workflow per major feature. Do not skip features.

> **Example — filled-in excerpt for ShopEase:**
> ```markdown
> ## Application Overview
> **Application Name:** ShopEase
> **Type:** REST API (Node.js / Express) + React Web App + Android App + iOS App
> **Primary Users:** Shoppers (registered), Guest Users, Store Admins
>
> ShopEase is a retail shopping platform letting customers browse products, manage a cart,
> place orders, and track deliveries. Store admins manage inventory and process refunds.
>
> ## Modules
> | Module | Description | Primary Roles | Found In |
> |--------|-------------|---------------|----------|
> | Authentication | Login, register, OTP verify, forgot password | Shopper, Admin | All |
> | Product Catalogue | Browse, search, filter products | Shopper, Guest | All |
> | Cart | Add/remove items, apply coupons | Shopper | All |
> | Checkout | Address, payment, order placement | Shopper | All |
> | Order Management | History, tracking, cancellation | Shopper, Admin | All |
> | Admin Dashboard | Inventory, analytics, refunds | Admin | API + Web |
>
> ## User Workflows
> ### Workflow 1: Place an Order
> **Platforms:** Web, Android, iOS
> **Entry point:** /checkout · CheckoutActivity · CheckoutView
> **Triggered by:** User taps "Proceed to Checkout"
>
> 1. User reviews cart items; cart loaded via GET /api/cart
> 2. User fills address, pincode (6 digits), phone (10 digits starting 6–9), payment method
> 3. Client validates form (Zod / ViewModel / ViewModel); POST /api/orders
> 4. API: validates BR-01 (non-empty cart), BR-02 (≤ ₹50K), BR-03 (KYC if > ₹10K); creates order
> 5. Order confirmed screen shown; push notification sent via Firebase
> ```

---

## Tech Stack

- **Web Frontend:** [framework + version, build tool, UI libs, state management, form libs — from Web package.json]
- **Backend:** [framework + runtime + version — from API package.json / pom.xml / *.csproj]
- **Android:** [language, min/target SDK, key libraries — from build.gradle]
- **iOS:** [language, min iOS version, key libraries — from Podfile / Package.swift]
- **Database:** [DB engine + ORM / ODM — from API drivers; Room for Android; CoreData/SwiftData for iOS]
- **Auth:** [auth mechanism — from API middleware + mobile Keychain/auth libs]
- **Integrations:** [all third-party services found across all platforms — payments, messaging, analytics, push, storage]
```

---

### Output File 2 — `knowledge/business_rules.md`

```markdown
# Business Rules

## Business Rules

| ID | Rule | Module | Source | Priority |
|----|------|--------|--------|----------|
| BR-01 | [plain-English rule] | [module name] | API / Web / Android / iOS / Multiple | High / Medium / Low |

Rules:
- Number sequentially BR-01, BR-02, ...
- API-enforced rules = High priority by default (server is authoritative).
- Client-only rules (Web / Android / iOS only) = Medium — note they can be bypassed by direct API calls.
- If the same rule appears in both API and mobile, mark Source = API + [platform] and Priority = High.

---

## Validation Rules

| Field | Rule | Error Message | Enforced In |
|-------|------|---------------|-------------|
| [field name] | [condition — e.g. "must be ≥ 8 characters"] | [exact error string or "Silent / no message"] | API / Web / Android / iOS / Multiple |

Rules:
- API: extract from DTO decorators, Zod/Joi/Yup schemas on server, model constraints.
- Web: extract from Zod/Yup/React Hook Form schemas, Formik `validate` functions.
- Android: extract from ViewModel `validate*` functions, Room `@NotNull`, Retrofit field annotations.
- iOS: extract from ViewModel validation functions, CoreData required attributes, `Localizable.strings` error messages.
- When client and server validate the same field, write one row and mark Enforced In = Multiple.

---

## System Constraints

- [constraint description] — Source: [platform] [file/env var name]

Examples of what to list:
- API rate limits (requests per second/minute)
- File upload size limits
- Session / token expiry durations
- Max pagination page size
- Android: `minSdk`, required permissions (`CAMERA`, `LOCATION`, etc.)
- iOS: required Info.plist permissions, background mode restrictions
- Timeouts (API, network, session)
- Feature flags and their default values

---

## Edge Cases

- [edge case description] — Source: [platform] [file or function name]

Examples of what to list:
- API: transaction rollback conditions, specific catch blocks that change behaviour
- Web: empty-state handling, offline detection, error boundary conditions
- Android: `onSaveInstanceState` restoring partial form, background process killed mid-flow
- iOS: `do/catch` with specific `Error` enum cases, app backgrounded mid-transaction
- Any TODO / FIXME comment that reveals deferred or incomplete logic
```

> **Example — filled-in excerpt for ShopEase:**
> ```markdown
> ## Business Rules
> | ID | Rule | Module | Source | Priority |
> |----|------|--------|--------|----------|
> | BR-01 | Cart must have at least one item before checkout | Cart | API + Android + iOS | High |
> | BR-02 | Single order total cannot exceed ₹50,000 | Order | API + Android + iOS | High |
> | BR-03 | KYC verification required for orders above ₹10,000 | Order | API | High |
> | BR-04 | COD not available for orders above ₹5,000 | Order | API | High |
> | BR-05 | A coupon can only be applied once per user account | Coupon | API | High |
> | BR-06 | Out-of-stock items cannot be added to cart | Cart | API + Web | High |
>
> ## Validation Rules
> | Field | Rule | Error Message | Enforced In |
> |-------|------|---------------|-------------|
> | `pincode` | Exactly 6 numeric digits | "Pincode must be exactly 6 digits" | API + Web + Android + iOS |
> | `phone` | 10 digits, starts with 6–9 | "Enter a valid 10-digit mobile number" | API + Web + Android + iOS |
> | `deliveryAddress` | Required, max 250 chars | "Delivery address is required" | API |
> | `paymentMethod` | One of: card, upi, cod | "Invalid payment method" | API |
> | `couponCode` | Max 20 chars, optional | "Coupon code too long" | Web |
>
> ## System Constraints
> - Rate limit: 100 requests/min — Source: API `.env` `RATE_LIMIT_REQUESTS_PER_MINUTE`
> - Max cart items: 50 — Source: API `.env` `MAX_CART_ITEMS`
> - OTP valid for 300 seconds — Source: API `.env` `OTP_EXPIRY_SECONDS`
> - Android minSdk: 26 (Android 8.0) — Source: Android `app/build.gradle`
> - Camera permission required (QR payments) — Source: iOS `Info.plist`
> - Location permission required (address suggestions) — Source: iOS `Info.plist`
>
> ## Edge Cases
> - Guest users can browse and add to cart but are redirected to login at checkout — Source: API `auth.middleware.ts`
> - If Razorpay is unreachable, order held as `payment_pending` for 30 min then auto-cancelled — Source: API `PaymentService.ts`
> - If stock drops to 0 between cart add and checkout, API returns out-of-stock item IDs — Source: API `OrderService.ts` + FIXME comment
> - Android: partial checkout form saved via `onSaveInstanceState` if app backgrounded — Source: Android `CheckoutFragment.kt`
> ```

---

## General Extraction Rules

1. **Do NOT invent rules** — only extract what is actually written in the code.
2. If a rule is implied by logic (e.g. `if (amount > 10000) throw LimitExceeded`) write it as plain English.
3. If a section is not found in any project, write `Not found in codebase` for that section.
4. **Always scan test files** (`*.test.*`, `*Test.*`, `*Spec.*`, `*Tests.swift`) — they often state business rules more clearly than production code.
5. When the same rule appears in multiple platforms, write it **once** and mark all platforms in the Source / Enforced In column.
6. Prefer **API** as the authoritative source for business rules. Mobile and Web rules are secondary (they can be bypassed).
7. Prefer **UI / Android / iOS** as the authoritative source for user workflow steps (how users interact).
8. After scanning all provided projects, write final merged files to:
   - `knowledge/product_context.md`
   - `knowledge/business_rules.md`
