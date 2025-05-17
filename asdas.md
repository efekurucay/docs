Okay, here is a detailed project report for the "Fibiyo E-Commerce Platform" in English, including explanations for the diagrams and architecture.

---

**Fibiyo E-Commerce Platform - Detailed Project Report**

**1. Introduction and Project Summary**

Fibiyo is designed as a comprehensive e-commerce platform. It aims to provide a shopping experience for customers and enable sellers to list and sell their products. Additionally, an admin panel is included for platform management. The project is developed with a modern backend architecture (Spring Boot) and a reactive frontend (Angular). It incorporates innovative features such as Artificial Intelligence (AI) capabilities (image generation, review summarization), short video integration ("Feels"), and a subscription system.

**Purpose of the Report:** This report aims to provide a detailed explanation of the Fibiyo project's system architecture, backend and frontend structures, technologies used, core components, and data flows. It will also touch upon potential areas for future development.

---

**2. System Architecture**

The overall system architecture follows a typical three-tier (or n-tier) web application model and may involve various microservice-like interactions, especially with external services like AI and payment.

**Conceptual System Architecture Diagram:**

```mermaid
graph LR
    A[Users (Customer, Seller, Admin)] -->|Web Browser| B(Frontend - Angular)
    B <-->|API Requests/Responses| C{Backend (Spring Boot / Java)}
    C --> D[Database (MySQL)]
    C --> E[File Storage (Local/Cloud)]
    C --> F[Payment Service (Stripe)]
    C --> G[AI Service (OpenAI)]

    subgraph "User Interface Tier"
        B
    end

    subgraph "Application Tier"
        C
    end

    subgraph "Data Tier & External Services"
        D
        E
        F
        G
    end
```

**Diagram Explanation:**

*   **Users:** Customers, Sellers, and Admins access the platform via web browsers.
*   **Frontend (Angular):** Presents the user interface, manages user interactions, and sends requests to the backend API. Runs in the browser.
*   **Backend (Spring Boot):** Manages business logic, data processing, API endpoints, and integrations with external services.
*   **Database (MySQL):** Stores all persistent data (users, products, orders, etc.).
*   **File Storage (Local/Cloud):** Stores files such as product images and "Feel" videos (e.g., local `uploads/` directory or a cloud-based solution).
*   **Payment Service (Stripe):** Manages secure payment transactions.
*   **AI Service (OpenAI):** Used for image generation and potentially other text-based AI features.

*(Note: Optional components like Load Balancer and API Gateway from the Turkish report can be added here if scaling or microservice decomposition is planned, but for the current monolithic backend, this simplified view is more direct.)*

---

**3. Backend Architecture**

The backend is designed using the Spring Boot framework with a layered architecture. This structure aims for modularity, testability, and maintainability.

**3.1. Technologies and Libraries Used:**

*   **Framework:** Spring Boot 3.4.5
*   **Language:** Java 17
*   **Database:** MySQL (defined with `schema.sql`)
*   **ORM:** Spring Data JPA (with Hibernate implementation)
*   **Security:** Spring Security, JWT (JSON Web Tokens)
*   **API:** Spring Web (REST Controllers)
*   **Build Tool:** Apache Maven
*   **DTO Mapping:** MapStruct
*   **Payment Integration:** Stripe Java SDK
*   **AI Integration:** OpenAI Java SDK, WebClient (for HTTP requests)
*   **Email:** Spring Boot Starter Mail
*   **File Operations:** Thumbnailator (image compression), Apache Commons IO
*   **Others:** Lombok, Jakarta Validation

**3.2. Directory Structure and Layering (Conceptual Layered Architecture):**

The backend code is logically divided into layers based on the separation of concerns principle.

```mermaid
graph TD
    Z[Configuration (config)] --> Y
    Y[Presentation Layer (Web Layer - infrastructure/web/controller)] --> X
    X[Application Layer (Business Logic - application/*)] --> W
    W[Domain Layer (Core Entities - domain/*)] --> V
    V[Infrastructure Layer (External Interactions)]

    subgraph "Presentation (Web)"
        Y["Controllers (HTTP Request Handling, Response Preparation, Service Invocation)"]
    end

    subgraph "Application (Business Logic)"
        X1["Services (application/service & impl - Business rules, validation, orchestration)"]
        X2["DTOs (application/dto - Data Transfer Objects)"]
        X3["Mappers (application/mapper - Entity/DTO Conversion)"]
        X4["Exceptions (application/exception - Custom Errors, Global Handler)"]
        X5["Utilities (application/util - Helper classes)"]
    end
    X --- X1 & X2 & X3 & X4 & X5


    subgraph "Domain (Core Entities)"
        W1["Entities (domain/entity - Database Entities)"]
        W2["Enums (domain/enums - Constant Sets)"]
    end
    W --- W1 & W2

    subgraph "Infrastructure (External Interactions)"
      V1["Persistence (infrastructure/persistence - Repositories, Specifications)"]
      V2["Security (infrastructure/security - JWT, UserDetails, Filters)"]
      V3["Adapters (infrastructure/adapter - Email, AI, Storage, Payment)"]
    end
    V --- V1 & V2 & V3

    Z --> V2
    Z --> V3

    X1 --> X2
    X1 --> X3
    X1 --> W1
    X1 --> W2
    X1 --> V1
    X1 --> V2
    X1 --> V3


    V1 --> W1
    V1 --> W2

    V3 --> ExtSrvc((External Services e.g. Stripe, OpenAI, File Storage))
```

**Diagram Explanation:**

*   **Presentation Layer (`infrastructure/web/controller`):**
    *   Receives HTTP requests from the outside world (e.g., `/api/auth/login`, `/api/products`).
    *   Validates incoming requests (via DTO validation).
    *   Invokes appropriate service methods to trigger business logic.
    *   Prepares and returns HTTP responses (DTOs in JSON format) to the client.
    *   Uses Spring MVC annotations like `@RestController`, `@GetMapping`, `@PostMapping`.
    *   Utilizes `@PreAuthorize` for authorization.
*   **Application Layer (`application` package):**
    *   Contains the core business logic of the project.
    *   **Services (`application/service` and `impl`):** Implement business rules, coordinate between different domain entities and repositories. Called by controllers. E.g., `AuthServiceImpl` manages user registration and login, `OrderServiceImpl` handles order creation and updates.
    *   **DTOs (Data Transfer Objects - `application/dto`):** Used for data transfer between layers, especially in API requests and responses. Contain validation annotations (`@NotBlank`, `@Size`, etc.).
    *   **Mappers (`application/mapper`):** Convert between Entities and DTOs. MapStruct is used, as indicated by generated files like `CartItemMapperImpl.java`.
    *   **Exceptions (`application/exception`):** Custom error classes (e.g., `ResourceNotFoundException`) and a global error handler (`GlobalExceptionHandler`).
    *   **Utilities (`application/util`):** Helper classes used across the project (e.g., `SlugUtils`).
*   **Domain Layer (`domain` package):**
    *   Defines the core business entities and rules. Should be independent of other layers.
    *   **Entities (`domain/entity`):** JPA entities corresponding to database tables (e.g., `User`, `Product`, `Order`). Contain JPA annotations like `@Entity`, `@Table`.
    *   **Enums (`domain/enums`):** Define fixed sets of constants (e.g., `Role`, `OrderStatus`).
*   **Infrastructure Layer (`infrastructure` package):**
    *   Handles interactions with external systems and infrastructural concerns.
    *   **Persistence (`infrastructure/persistence`):**
        *   **Repositories (`repository`):** Spring Data JPA interfaces abstracting database operations (CRUD).
        *   **Specifications (`specification`):** Used with Spring Data JPA Specifications API to build dynamic and complex queries.
    *   **Security (`infrastructure/security`):** Classes related to authentication and authorization. Includes `JwtTokenProvider`, `JwtAuthenticationFilter`, `CustomUserDetailsService`, `AuthEntryPointJwt`, and custom security checkers like `ProductSecurity`, `OrderSecurity`.
    *   **Adapters (`infrastructure/adapter`):** Classes that integrate with external services (AI, Payment, Email, File Storage). They abstract the implementation details of external services from the business logic. E.g., `OpenAiServiceImpl` (also acts as an adapter), `LocalStorageServiceImpl`.
*   **Configuration Layer (`config` package):**
    *   Contains Spring Boot configurations. Classes like `SecurityConfig` (CORS, JWT filter, endpoint authorization), `StripeConfig`, `OpenAiConfig`, `WebConfig` (static file serving).

**3.3. Backend Component Diagram (Conceptual Packages and Interactions):**

This diagram illustrates how the main backend packages/modules interact.

```mermaid
graph LR
    subgraph "Presentation (Web)"
        A[Controllers]
    end

    subgraph "Application (Business Logic)"
        B[Services]
        C[DTOs]
        D[Mappers]
        E[Exceptions]
    end

    subgraph "Domain (Core Entities)"
        F[Entities]
        G[Enums]
    end

    subgraph "Infrastructure (External Interactions)"
        H[Repositories]
        I[Security Components]
        J[Adapters (AI, Payment, etc.)]
        K[Storage Service]
    end

    subgraph "Configuration"
        L[Configuration Classes]
    end

    A -->|uses| B
    A -->|uses| C
    A -->|handles| E

    B -->|uses| D
    B -->|uses| C
    B -->|uses| F
    B -->|uses| G
    B -->|uses| H
    B -->|uses| J
    B -->|uses| K
    B -->|uses| I

    D -->|maps to/from| C
    D -->|maps to/from| F

    H -->|manages| F
    H -->|uses| G

    I -->|secures| A
    I -->|uses| F

    J -->|interacts with| M{External Services (Stripe, OpenAI)}
    K -->|interacts with| N[File System/Cloud Storage]

    L -->|configures| I
    L -->|configures| J
    L -->|configures| K
```

**Explanation:**

*   **Controllers (A):** Receive HTTP requests, validate data using DTOs (C), call Services (B) for business logic, and may handle Exceptions (E).
*   **Services (B):** Contain main business logic. Use Mappers (D) for DTO (C) / Entity (F) conversion. Use Repositories (H) for database operations. May call Adapters (J) or Storage Service (K) for external interactions. Use project Enums (G) and Security context (I).
*   **Repositories (H):** Interact with the database, managing JPA Entities (F).
*   **Security Components (I):** Handle JWT, user authentication, and authorization, using User entity (F).
*   **Adapters (J) / Storage Service (K):** Interact with External Services (M) or the file system (N).
*   **Configuration Classes (L):** Configure systems like Spring Security (I), OpenAI (J), Stripe (J).

**3.4. Database Schema (`schema.sql` Summary):**

The `schema.sql` file provides a detailed DDL for MySQL 8.0. Key tables and their relationships include:

*   **`users`**: User information, roles, social login details, subscription info, AI quota.
*   **`categories`**: Hierarchical category structure (via `parent_category_id`).
*   **`products`**: Product details, price, stock, SKU, image URLs, seller and category relations, approval/active status, AI features. Includes fields for soft delete (`is_deleted`) and optimistic locking (`version`).
*   **`product_images`**: Multiple images per product.
*   **`coupons`**: Discount coupons, types, values, expiry dates.
*   **`orders`**: Customer orders, statuses, addresses, payment details, used coupon.
*   **`order_items`**: Individual items in an order, price at the time of purchase.
*   **`payments`**: Payment records, methods, transaction IDs, statuses.
*   **`reviews`**: Product reviews, ratings, approval status.
*   **`notifications`**: User notifications.
*   **`wishlist_items`**: Users' wishlisted products.
*   **`carts`**: Active shopping carts for users.
*   **`cart_items`**: Items within a shopping cart.
*   **`feels`**: Short video content ("Feels") uploaded by sellers, associated products, views, likes.
*   **`feel_likes`**: Tracks user likes for "Feels".

The schema correctly defines foreign key constraints, unique constraints, and indexes for performance. Strategies like `ON DELETE SET NULL` and `ON DELETE CASCADE` are used to maintain relational integrity. Calculated fields like `product.finalAmount` are now managed at the service layer.

---

**4. Frontend Architecture**

The frontend is developed using Angular and TypeScript, aiming for a modular structure, reactive programming (Signals), and best practices.

**4.1. Technologies and Libraries Used:**

*   **Framework:** Angular (v19.2.x)
*   **Language:** TypeScript (v5.7.x)
*   **Styling:** SCSS
*   **Build Tool:** Angular CLI
*   **HTTP Client:** Angular HttpClient (likely with `withFetch`)
*   **Routing:** Angular Router
*   **Forms:** Angular Reactive Forms
*   **State Management:** Angular Signals (preferred)
*   **UI Library:** Angular Material (partially used: MatCard, MatFormField, etc.)
*   **Others:** `ngx-infinite-scroll` (potentially for infinite scrolling in "Feels" list, etc.)

**4.2. Directory Structure and Modularity (Angular Standalone API Focused):**

The frontend aims for modularity using Angular's modern Standalone API.

```
frontend/
├── src/
│   ├── app/
│   │   ├── app.component.ts           # Root application component
│   │   ├── app.config.ts              # Main application configuration (providers)
│   │   ├── app.routes.ts              # Main routing (with lazy loading)
│   │   │
│   │   ├── core/                      # Core services, guards, interceptors
│   │   │   ├── constants/             # API endpoints, etc.
│   │   │   ├── guards/                # auth.guard.ts, role.guard.ts
│   │   │   ├── interceptors/          # auth.interceptor.ts, error.interceptor.ts
│   │   │   └── services/              # api.service.ts, auth.service.ts, storage.service.ts, notification.service.ts (core)
│   │   │
│   │   ├── shared/                    # Shared, reusable elements
│   │   │   ├── components/            # PaginatorComponent, LoadingSpinnerComponent, ConfirmationModalComponent, NotificationContainerComponent
│   │   │   ├── models/                # Interfaces corresponding to backend DTOs
│   │   │   ├── enums/                 # Enums corresponding to backend enums
│   │   │   └── pipes/                 # nl2br.pipe.ts, time-ago.pipe.ts
│   │   │
│   │   ├── layout/                    # Main layout components
│   │   │   ├── main-layout/           # Layout for general user interface
│   │   │   ├── admin-layout/          # Layout for admin panel
│   │   │   ├── header/
│   │   │   └── footer/
│   │   │
│   │   └── features/                  # Feature-based modules/directories
│   │       ├── home/
│   │       ├── auth/
│   │       ├── products/
│   │       ├── categories/
│   │       ├── cart/
│   │       ├── checkout/
│   │       ├── orders/
│   │       ├── profile/
│   │       ├── wishlist/
│   │       ├── notifications/
│   │       ├── feels/
│   │       ├── admin/
│   │       │   ├── dashboard/
│   │       │   ├── user-management/
│   │       │   ├── product-management/
│   │       │   ├── order-management/
│   │       │   └── components/ (admin-specific shared components)
│   │       ├── seller/
│   │       │   ├── dashboard/
│   │       │   ├── product-management/
│   │       │   ├── order-management/
│   │       │   └── components/ (seller-specific shared components)
│   │       ├── subscriptions/
│   │       └── reports/ (future)
│   │
│   ├── assets/
│   └── environments/
├── angular.json
├── package.json
└── tsconfig.json
```

*   **`core/`**: Contains foundational logic. `ApiService` handles backend communication. `AuthService` manages authentication and user state. Guards control route access. Interceptors modify HTTP requests/responses.
*   **`shared/`**: Reusable components (e.g., `PaginatorComponent`), data models (interfaces for DTOs), and enums.
*   **`layout/`**: Components that form the main page structure: `HeaderComponent`, `FooterComponent`, and layout wrappers like `MainLayoutComponent`, `AdminLayoutComponent`.
*   **`features/`**: Main functional areas of the application. Each feature resides in its own sub-directory, containing its components, services (if specific to the feature), and route definitions.

**4.3. Frontend Component Diagram (Example Flow - Product Listing & Detail):**

This diagram shows how components and services interact when a user views a product list and then a product detail page.

```mermaid
graph TD
    User[User] -->|Route Navigation| AppRouter[app.routes.ts]

    AppRouter -->|Lazy Load| ProductsRoutes[products.routes.ts]
    ProductsRoutes -->|Navigates to| PL[ProductListComponent]

    PL -->|Data Request| PS[ProductService]
    PS -->|HTTP GET (with filters)| AS[ApiService]
    AS -->|API Call| BackendAPI[Backend API (/api/products)]
    BackendAPI -->|Page<Product>| AS
    AS -->|Page<Product>| PS
    PS -->|Page<Product>| PL

    PL -->|Renders with *ngFor| PCC[ProductCardComponent]
    PCC -->|@Input() product| ProductData[Product Data]
    User -->|Clicks Add to Cart| PCC
    PCC -->|Add to Cart Request| CartService[CartService]
    User -->|Clicks Detail Link| AppRouter
    AppRouter -->|Lazy Load| ProductsRoutes
    ProductsRoutes -->|Navigates with :slug| PD[ProductDetailComponent]

    PD -->|Data Request (by slug)| PS
    PS -->|HTTP GET (by slug)| AS
    AS -->|API Call| BackendAPI_Detail[/api/products/slug/:slug]
    BackendAPI_Detail -->|Product| AS
    AS -->|Product| PS
    PS -->|Product| PD
    PD -->|Displays| ProductDetailsUI[Product Details UI]

    PD -->|Lists Reviews| ReviewsComp[ProductReviewsComponent]
    ReviewsComp -->|Data Request| ReviewService[ReviewService]
    ReviewService -->|HTTP GET| AS
    AS -->|API Call| BackendAPI_Reviews[/api/reviews/product/:id]
    BackendAPI_Reviews -->|Page<Review>| AS
    AS -->|Page<Review>| ReviewService
    ReviewService -->|Page<Review>| ReviewsComp

    subgraph "Product Feature"
        PL
        PCC
        PD
        PS
        ReviewsComp
        ReviewService
    end

    subgraph "Core"
        AppRouter
        AS
        CartService
    end
```

**Explanation:**

1.  User navigates to `/products` (or a filtered path).
2.  `app.routes.ts` lazy loads `products.routes.ts`.
3.  `ProductListComponent` becomes active.
4.  `ProductListComponent` requests products from `ProductService` (with filters and pagination).
5.  `ProductService` calls `/api/products` via `ApiService`.
6.  The `Page<Product>` data is returned to `ProductListComponent`.
7.  `ProductListComponent` renders `ProductCardComponent` for each product.
8.  User clicks a product detail link; `ProductDetailComponent` is routed via `:slug`.
9.  `ProductDetailComponent` fetches product details using `ProductService.getPublicProductBySlug()`.
10. `ProductReviewsComponent` (if part of the detail page) fetches reviews via `ReviewService`.
11. "Add to Cart" clicks from `ProductCardComponent` or `ProductDetailComponent` call `CartService.addItemToCart()`.

**4.4. State Management (Angular Signals):**

As per `.clinerules/rules.md`, Angular Signals are the preferred method for state management.

*   **Service State:** Global states like `currentUser` and `isAuthenticated` in `AuthService`, and cart state in `CartService`, are managed using `signal`.
*   **Component State:** Components use `signal` for local state and to read signals from services. `computed` signals are used for derived values (e.g., `pages` in `PaginatorComponent`). `effect` is used for side effects in response to signal changes (e.g., API calls, logging, as seen in `AuthService` and `CartService` constructors).

**4.5. Routing and Lazy Loading:**

*   `app.routes.ts` contains the main routing configuration.
*   **Lazy loading** is implemented for all feature modules/directories under `features/` using `loadChildren`, optimizing initial load time.
*   Routes are protected using `AuthGuard` and `RoleGuard`.

---

**5. Deployment Architecture (Conceptual)**

While specific deployment details are not provided, a typical web application deployment architecture might look like this:

```mermaid
graph TD
    A[User Browser] --> B(CDN - Optional for Frontend Statics)
    B --> C ((Load Balancer))
    C --> D (( Web Servers Nginx Apache serving Angular build))
    D --> C_LB(Load Balancer for Backend)
    C_LB --> E(Application Servers (Spring Boot JAR Cluster))
    E --> F(Database Server (MySQL))
    E --> G((External Services - Stripe, OpenAI))
```

*   **CDN (Content Delivery Network):** Can be used for frontend static files (JS, CSS, images) to speed up global access.
*   **Load Balancer:** Can be used for both frontend (web servers) and backend (application servers) to distribute incoming requests, improving performance and redundancy.
*   **Web Servers:** Servers like Nginx or Apache serve the static files from the Angular build and proxy API requests to the backend application servers.
*   **Application Servers:** Servers running the Spring Boot application, typically in a cluster.
*   **Database Server:** Server running the MySQL database. Replication or cluster setups can be used for redundancy and performance.
*   **External Services:** Services like Stripe and OpenAI run on their own infrastructure and are integrated via APIs.

---

**6. Core Features (Based on API and Frontend Structure)**

1.  **User Management & Authentication:** Registration, login, password recovery, profile view/update, role-based access.
2.  **Category Management:** Listing (active, all, sub, root), detail view, admin CRUD.
3.  **Product Management:** Public listing (paginated, filtered, sorted), detail view, seller CRUD, admin approval/rejection.
4.  **Shopping Cart:** View, add, update quantity, remove item, clear cart.
5.  **Wishlist:** View, add/remove product, check if product is wishlisted.
6.  **Order Management:** Customer order creation, history, details, cancellation. Admin/Seller order listing, details, status updates, tracking.
7.  **Review Management:** List product reviews, customer add/delete review, admin moderation.
8.  **Coupon Management:** Coupon validation, admin CRUD.
9.  **Notification System:** User notifications, unread count, mark as read, delete.
10. **Payment System (Stripe Integration):** Checkout session creation, webhook handling.
11. **Subscription System:** View status, create subscription checkout, periodic expiry checks.
12. **"Feels" (Short Video) Feature:** Seller creation/management, public listing/viewing, liking, admin moderation.
13. **AI Features (for Sellers):** Product image generation/editing (with quota), setting AI image as main.

---

**7. API Documentation Summary (`apidocs.md`)**

The `apidocs.md` file provides a detailed description of the backend API endpoints, request (Request) and response (Response) DTOs, and authorization rules. This document is the primary reference for frontend development. Its main sections include:

*   General Information (Base URL, Authentication, Error Responses)
*   Endpoint Groups by Controller:
    *   Auth, User, Category, Product (Public, Seller, Admin), SellerProduct, Review, Cart, Wishlist, Order, Coupon, Notification, Payment (Stripe), Subscription, AdminUser, Feel.
*   Additional DTO Definitions.

---

**8. Development Standards & Best Practices (Frontend Focused)**

The principles outlined in `.clinerules/angular.md` and `.clinerules/rules.md` aim to enhance the project's quality and maintainability:

*   **Modularity:** Preference for Standalone API, feature-based folder structure.
*   **Performance:** Lazy loading, Angular Signals, OnPush change detection, `NgOptimizedImage`, `@defer` blocks.
*   **Code Quality:** Strict TypeScript usage (no `any`), meaningful naming, file naming standards, SCSS usage, code cleanliness.
*   **Angular Best Practices:** `inject()` function, `async` pipe, `trackBy`, component composition.
*   **Accessibility:** Semantic HTML and ARIA attributes.
*   **Security:** XSS prevention, sanitization.
*   **Testing:** AAA pattern for unit tests.

---

**9. Potential Development Areas & Future Work**

*   **Comprehensive Testing:** Increase unit, integration, and E2E test coverage.
*   **CI/CD Pipeline:** Establish automated build, test, and deployment processes.
*   **Advanced State Management:** For larger projects, consider a more centralized state management solution (NgRx, Akita) in addition to or instead of Angular Signals for highly complex states.
*   **Real-time Notifications:** WebSocket integration for instant notifications.
*   **Detailed Reporting:** More comprehensive sales, product performance, etc., reports for Admin and Seller panels.
*   **Enhanced AI Features:** Review analysis, personalized product recommendations, chatbot integration.
*   **Microservice Architecture (If Needed):** If the project grows significantly, specific modules (e.g., payment, notification, AI) could be refactored into separate microservices.
*   **Mobile Application:** Develop native or cross-platform mobile applications for the platform.
*   **UI/UX Improvements:** Continuous enhancements and user testing to improve user experience.
*   **Docker & Containerization:** For easier deployment and scalability.
*   **Completion of `TODO.md` Items:** Address remaining tasks, especially for admin panel, seller panel, and reporting features.

---

**10. Conclusion**

The Fibiyo E-Commerce Platform is a feature-rich project built on a strong foundation on both the backend and frontend. The modern technologies used and the planned architecture have the potential to make the project scalable, maintainable, and performant.


---
