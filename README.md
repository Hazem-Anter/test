
Below is a component-by-component breakdown (ordered by the routes file):

---

### Layouts

#### `MainLayout` (`./Layout/main-layout/main-layout.ts` + `.html`)
- **Selector:** `app-main-layout`
- **Purpose:** Shell for public/app pages (navbar + footer + `<router-outlet>`).
- **Key:** imports `Navbar` and `Footer`. Template places `<router-outlet>` in `.main-container`.
- **Notes:** Used as parent component for the majority of routes.

#### `AuthLayout` (`./Layout/auth-layout/auth-layout.ts` + `.html`)
- **Selector:** `app-auth-layout`
- **Purpose:** Layout used for auth pages (login/register). Contains `app-navbar` and a `<router-outlet>` for auth views.
- **Notes:** Lightweight wrapper.

---

### Public pages (inside `MainLayout`)

#### `Landing` (`features/landing/landing.ts` + `landing.html`)
- **Route:** `/` (empty path)
- **Purpose:** Home / hero / category carousel.
- **Key properties:** `categories` array, `isNavOpen`, `activeIndex`, auto-slide timer.
- **Key methods:** `nextSlide()`, `prevSlide()`, `openNav()`, `closeNav()`.
- **Template:** carousel + category cards linking to `category-list` with query param `category`.

#### `CategoryList` (`features/category-list/category-list.ts` + `category-list.html`)
- **Route:** `/category-list`
- **Purpose:** Show products filtered by category query param (`?category=...`).
- **Files:** `category-list.ts`, `category-list.html`, `category-list.css`
- **Key behavior:** subscribes to `route.queryParams`, calls `Data.getFilteredCategories(category)` to load `products`, lists them using `app-product-card`.

#### `About` (`features/about/about/about` — *not provided*)
- **Route:** `/about`
- **Purpose:** Static informational page about the store.
- **Notes:** You did not share its implementation; expect simple static markup.

#### `ProductList` (`features/products/product-list/product-list.ts` + `.html`)
- **Route:** `/products`
- **Purpose:** Display all products with a filter panel.
- **Key properties:** `products`, `filteredProducts`, `loading`.
- **Key methods:** `ngOnInit()` loads products via `Data.getProducts()`. `onProductsChanged(products)` receives filtered products from `FilterPanel`.
- **Template:** shows a left `app-filter-panel` and a grid of `<app-product-card>` components.

#### `ProductDetail` (`features/products/product-detail/product-detail.ts` + `.html`)
- **Route:** `/products/:id`
- **Purpose:** Product detail page with image gallery, quantity, add to cart, favourites, and reviews.
- **Key properties:** `productId`, `product`, `selectedImage`, `quantity`, `isInFavourites`, `newReview`, `stars`, `user`, `hasReviewed`.
- **Key methods:**
  - `ngOnInit()` reads `id` param and calls `loadProduct()`.
  - `loadProduct()` loads product via `Data.getProductById(id)`, merges local reviews stored in `localStorage` (key `reviews-${product.id}`), sets selected image.
  - `addToCart()` → calls `productService.addToCart(productId, quantity)` (throws messages as implemented).
  - `toggleFavourite()` → calls `productService.toggleFavourite(productId)`.
  - `submitReview()` → validates, appends review to `localStorage` and to `product.reviews`.
  - `incrementQuantity()`, `decrementQuantity()`, `setRating(star)`.
- **Notes:** Reviews are persisted locally per-product; favourites and cart require user to be logged in (Data service enforces that).

#### `ProductSearch` (`features/products/product-search/product-search.ts` + `.html`)
- **Route:** `/search/:query`
- **Purpose:** Search results page.
- **Key:** reads `query` from route params, calls `Data.searchProducts(query)`, shows results with `<app-product-card>`.

---

### Protected user routes (guarded by `authGuard`)

> `authGuard` checks normal `Auth` or `SocialAuth` current user and optionally enforces `route.data.role` (e.g., `admin`). If not logged in redirects to `/login`. If role mismatch redirects to `/`.

#### `Cart` (`features/cart/cart/cart.ts` + `.html`)
- **Route:** `/cart`
- **Purpose:** Show cart items, manage quantities, remove items, show subtotal, and proceed to checkout.
- **Key properties:** `allProducts`, `cartProductInfo`, `total`.
- **Key methods:** `ngOnInit()` loads cart items from `Data.getCartItems()` and fetches product details for each via `Data.getProductById()`. `addMore()`, `deduct()`, `removeItem()`, `recalculateTotal()`, `proceedToCheckout()`.

#### `Checkout` (`features/cart/checkout/checkout.ts` + `.html`)
- **Route:** `/checkout`
- **Purpose:** Collect shipping details, select payment method (credit card / PayPal / bank transfer), validate input, process orders.
- **Key properties:** Shipping fields (firstName, lastName, email, ...), `selectedPaymentMethod`, card/paypal/bank fields, `cartItems`, `subtotal`, `shipping`, `tax`, `total`, `submitted`.
- **Key methods:**
  - `loadOrderSummary()` fetches cart, populates `cartItems`, `calculateTotals()`.
  - `validateShippingForm()`, `processOrder()` (delegates to `processCreditCard()`, `processPayPal()`, `processBankTransfer()`).
  - Payment handlers create `order` object, call `Data.saveOrder(order)`, `Data.clearCart()` and navigate to `/order-confirmation`.

#### `OrderConfirmation` (`features/order-confirmation/order-confirmation.ts` + `.html`)
- **Route:** `/order-confirmation`
- **Purpose:** Displays order confirmation (order id, amount, shipping summary), shows countdown and redirects to landing/home.
- **Key:** uses `Data.GetLastOrder()` to load last saved order.

#### `FavouriteList` (`features/favourites/favourite-list/favourite-list.ts` + `.html`)
- **Route:** `/favourites`
- **Purpose:** Show user's favourites, allow removing or adding favourite items to cart.
- **Key properties:** `allFavoriteInfo`, `allFavorites`.
- **Key methods:** `ngOnInit()` loads favorites from `Data.getFavouritesItems()` and fetches product details. `RemoveFromWishlist(prodId)`, `AddToCart(prodId)`.

---

### Admin (under MainLayout > admin route)

#### `AdminDashboard` (`features/admin/admin-dashboard/admin-dashboard.ts` + `.html`)
- **Route:** `/admin`
- **Purpose:** Admin overview (summary cards: total users, total orders, pending orders) and tables of users and orders.
- **Key properties / methods:** `users`, `orders`, getters `totalUsers`, `totalOrders`, `pendingOrders`.
- **Access:** Protected by `authGuard` and `data: { role: 'admin' }`.

#### `ProductCrud` (`features/admin/product-crud/product-crud`)
- **Route:** `/admin/crud`
- **Purpose:** (Admin-only) UI to create/read/update/delete products.
- **Notes:** You referenced `ProductCrud` in the routes but did not send its implementation. Expect CRUD operations (likely calling an API or manipulating local state). Keep this component protected by the `admin` role.

---

### Auth pages (AuthLayout)

#### `Login` (`features/auth/login/login.ts` + `.html`)
- **Route:** `/login`
- **Purpose:** Normal login and social login via Firebase popup (Google & Facebook).
- **Key methods:**
  - `onSubmit()` → uses `Auth.login(formData)`.
  - `loginWithGoogle()` → uses Firebase `GoogleAuthProvider` and `signInWithPopup` to obtain user, maps to `User` model then `handleSocialUser()`.
  - `loginWithFacebook()` → similar to Google, with special handling for `auth/account-exists-with-different-credential` (attempts to link accounts).
  - `handleSocialUser(mappedUser)` saves or logs in user, updates `Data` and `Auth` state, and navigates to `/` or `/register`.
- **Notes:** After social login, `Login` may redirect new users to registration to complete profile.

#### `Register` (`features/auth/register/register.ts` + `.html`)
- **Route:** `/register`
- **Purpose:** Register a normal user (email/password) or complete social registration.
- **Key methods:**
  - `onSubmit()` calls `Auth.register(formData)` and displays success/error messages.
  - `registerWithGoogle()` / `registerWithFacebook()` call unified `handleSocialLogin(provider, source)` which signs in with popup, maps user, `saveSocialUser()` stores user and sets session.

---

### Guard

#### `authGuard` (`core/auth/auth-guard.ts`)
- **Type:** functional `CanActivateFn`
- **Purpose:** Protects routes by checking either `Auth.getCurrentUser()` or `SocialAuth.getCurrentUser()` and optionally enforces `route.data.role`.
- **Behavior:**
  - If not logged in → `router.createUrlTree(['/login'])`.
  - If `data.role` exists and user role ≠ required role → redirect to `/`.
  - Otherwise allow access.

---

### Shared Components (used by many routes)

#### `Navbar` / `Footer`
- Provide site navigation, show cart/favourites counts (likely reading `Data` signals), used inside both layouts.

#### `ProductCard` (`shared/components/product-card/product-card`)
- Displays product thumbnail, title, price, rating, add-to-cart and add-to-favourites actions. Used by `ProductList`, `CategoryList`, `ProductSearch`, etc.

#### `FilterPanel` (`shared/components/filter-panel/filter-panel`)
- Emits `productsChanged` with filtered products; used by `ProductList`.

---

## Where logic lives (quick map)
- **API calls:** `Data.getProducts()`, `getProductById()`, `searchProducts()`, `getFilteredCategories()`
- **Auth logic:** `Auth.register()`, `Auth.login()`, `Auth.logout()`, signal `auth.user`
- **Social login:** `SocialAuth.initGoogle()` / `loginWithFacebook()` / `loginSocial()`
- **Cart & Favourites:** `Data.addToCart()`, `Data.removeFromCart()`, `Data.addToFavourites()`, `Data.toggleFavourite()`
- **Orders:** `Data.saveOrder()` → read by `OrderConfirmation` via `Data.GetLastOrder()`
- **Route protection:** `authGuard` (checks `Auth` and `SocialAuth`)

---

## Notes & Recommendations
- **ProductCrud** implementation was not provided. Keep it admin-only and ensure it uses the same `Data` service or connects to the real API for create/update/delete.
- **About** and any minor pages not sent are assumed to be static; keep them inside `features/about` and reference from routes.
- **Consistency:** most components expect `Data` to return objects shaped like the DummyJSON product model — keep types aligned.
- **Security:** current auth is simulated via `localStorage`. Replace with a real backend + secure JWT flows for production.

---

### Quick reference: route -> component file paths (based on your structure)
- `''` (landing) → `features/landing/landing.ts` + `landing.html`
- `category-list` → `features/category-list/category-list.ts` + `category-list.html`
- `about` → `features/about/about/about.ts` + `.html` (not provided)
- `products` → `features/products/product-list/product-list.ts` + `.html`
- `products/:id` → `features/products/product-detail/product-detail.ts` + `.html`
- `search/:query` → `features/products/product-search/product-search.ts` + `.html`
- `cart` → `features/cart/cart/cart.ts` + `.html`
- `checkout` → `features/cart/checkout/checkout.ts` + `.html`
- `order-confirmation` → `features/order-confirmation/order-confirmation.ts` + `.html`
- `favourites` → `features/favourites/favourite-list/favourite-list.ts` + `.html`
- `admin` → `features/admin/admin-dashboard/admin-dashboard.ts` + `.html`
- `admin/crud` → `features/admin/product-crud/product-crud.ts` (not provided)
- `login` → `features/auth/login/login.ts` + `.html`
- `register` → `features/auth/register/register.ts` + `.html`

---

If you want, I can now:
- 1) Merge this Routes & Components section into the **full README** and regenerate the entire `README.md` so it replaces the existing README you already have; **or**
- 2) Generate a **developer reference file** (`DOCS.md`) that expands each component into an individual doc with code snippets and sample usage (perfect for handoff).

Which would you like me to do next?
