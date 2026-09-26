# Phovia Tour Server — Project Structure

> **Modular MVC Pattern** অনুসরণ করে তৈরি একটি scalable **Express + TypeScript + Mongoose** backend।
> প্রতিটা feature নিজের folder-এ থাকে, তাই project বড় হলেও code গোছানো থাকে।

---

## Folder Structure

```
phovia-tour-server/
├── src/
│   ├── app/
│   │   ├── config/
│   │   │   └── env.ts                  # Load & validate environment variables
│   │   │
│   │   ├── modules/
│   │   │   ├── user/
│   │   │   │   ├── user.interface.ts   # TypeScript types & interfaces
│   │   │   │   ├── user.model.ts       # Mongoose schema & model
│   │   │   │   ├── user.validation.ts  # Zod validation schema
│   │   │   │   ├── user.service.ts     # Business logic & DB queries
│   │   │   │   ├── user.controller.ts  # Request / response handling
│   │   │   │   ├── user.route.ts       # Express router
│   │   │   │   └── user.constant.ts    # (Optional) constants & enums
│   │   │   │
│   │   │   ├── auth/
│   │   │   │   ├── auth.service.ts
│   │   │   │   ├── auth.controller.ts
│   │   │   │   └── auth.route.ts
│   │   │   │
│   │   │   ├── tour/
│   │   │   │   ├── tour.interface.ts
│   │   │   │   ├── tour.model.ts
│   │   │   │   ├── tour.validation.ts
│   │   │   │   ├── tour.service.ts
│   │   │   │   ├── tour.controller.ts
│   │   │   │   └── tour.route.ts
│   │   │   │
│   │   │   ├── booking/
│   │   │   ├── payment/
│   │   │   └── division/
│   │   │
│   │   ├── routes/
│   │   │   └── index.ts                # Register all module routes
│   │   │
│   │   ├── middlewares/
│   │   │   ├── globalErrorHandler.ts   # Handle all errors in one place
│   │   │   ├── notFound.ts             # 404 handler
│   │   │   ├── validateRequest.ts      # Zod validation middleware
│   │   │   └── checkAuth.ts            # JWT verify & role check
│   │   │
│   │   ├── errorHelpers/
│   │   │   ├── AppError.ts             # Custom error class
│   │   │   ├── handleZodError.ts
│   │   │   ├── handleCastError.ts
│   │   │   └── handleDuplicateError.ts
│   │   │
│   │   ├── utils/
│   │   │   ├── catchAsync.ts           # Async try-catch wrapper
│   │   │   ├── sendResponse.ts         # Consistent response format
│   │   │   ├── jwt.ts                  # Generate & verify tokens
│   │   │   └── QueryBuilder.ts         # Search, filter, sort, pagination
│   │   │
│   │   └── interfaces/
│   │       ├── error.types.ts          # Error response types
│   │       └── index.d.ts              # Extend Express Request (req.user)
│   │
│   ├── app.ts                          # Express app setup
│   └── server.ts                       # DB connection & server start
│
├── .env                                # Secret values (not pushed to git)
├── .env.example                        # Template for .env
├── .gitignore
├── package.json
└── tsconfig.json
```

---

## Top-level Folder-গুলোর কাজ

| Folder | কাজ |
|---|---|
| `config/` | `.env` থেকে value পড়ে এক জায়গায় export করে, যাতে সব জায়গায় `process.env` লিখতে না হয় |
| `modules/` | প্রতিটা feature-এর নিজস্ব folder — project-এর আসল কাজ এখানে |
| `routes/` | সব module-এর router এক জায়গায় জুড়ে দেয় |
| `middlewares/` | Error handler, auth, validation, 404-এর মতো reusable middleware |
| `errorHelpers/` | Custom `AppError` class আর বিভিন্ন ধরনের error (Zod, Cast, Duplicate) সুন্দর format-এ রূপান্তর |
| `utils/` | `catchAsync`, `sendResponse`, `jwt`, `QueryBuilder`-এর মতো helper |
| `interfaces/` | পুরো project-এ ব্যবহার হওয়া common types |

---

## একটা Module-এর ভেতরের File-গুলো

| File | দায়িত্ব |
|---|---|
| `*.interface.ts` | TypeScript `interface` / `type` define করা |
| `*.model.ts` | Mongoose `Schema` আর `Model` তৈরি |
| `*.validation.ts` | Zod দিয়ে request body-র validation schema |
| `*.service.ts` | Business logic + database query (আসল কাজ) |
| `*.controller.ts` | `req` থেকে data নেওয়া, service call করা, response পাঠানো |
| `*.route.ts` | Express `Router` — endpoint, middleware, controller জোড়া |
| `*.constant.ts` | (optional) enum, role, searchable fields-এর মতো constant |

> সব module-এ সব file লাগবে না। যেমন `auth`-এর নিজস্ব model নেই, সে `user` model ব্যবহার করে।

---

## Request Flow

```
Client Request
     │
     ▼
app.ts
     │
     ▼
routes/index.ts
     │
     ▼
module.route.ts
     │
     ▼
middlewares (checkAuth, validateRequest)
     │
     ▼
module.controller.ts
     │
     ▼
module.service.ts
     │
     ▼
module.model.ts  ──►  MongoDB
     │
     ▼
sendResponse  ──►  Client Response


On error:  errorHelpers  ──►  globalErrorHandler  ──►  Client
```

> **মূল নিয়ম:** Controller পাতলা রাখুন। DB query আর logic সবসময় **service**-এ লিখুন।

---

## Centralized Router

**`src/app/routes/index.ts`**

```ts
import { Router } from "express";
import { UserRoutes } from "../modules/user/user.route";
import { AuthRoutes } from "../modules/auth/auth.route";
import { TourRoutes } from "../modules/tour/tour.route";

export const router = Router();

const moduleRoutes = [
  { path: "/user", route: UserRoutes },
  { path: "/auth", route: AuthRoutes },
  { path: "/tour", route: TourRoutes },
];

moduleRoutes.forEach((route) => {
  router.use(route.path, route.route);
});
```

**`src/app.ts`**

```ts
app.use("/api/v1", router);
app.use(globalErrorHandler);
app.use(notFound);
```

ফলে endpoint হবে: `/api/v1/user/...`, `/api/v1/auth/...`, `/api/v1/tour/...`

---

## Naming Convention

- Module folder নাম **lowercase + singular**: `user`, `tour`, `booking`
- File নাম **`<module>.<type>.ts`** format-এ: `tour.service.ts`
- Variable ও function: **camelCase** → `createTour`, `getAllUsers`
- Class, Interface, Model: **PascalCase** → `AppError`, `ITour`, `Tour`
- Export object: **PascalCase** → `UserServices`, `TourControllers`, `UserRoutes`

---

## নতুন Module যোগ করার ধাপ

1. `src/app/modules/` এর ভেতরে নতুন folder বানান (যেমন `booking/`)
2. File তৈরি করুন: `interface` → `model` → `validation` → `service` → `controller` → `route`
3. `routes/index.ts`-এর `moduleRoutes` array-তে নতুন route যোগ করুন

ব্যস, নতুন feature ready!