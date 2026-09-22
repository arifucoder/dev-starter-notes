# React (Vite + TS) + Tailwind + shadcn Setup Guide

---

## ১. React Project তৈরি করা

```sh
npm create vite@latest my-react-app -- --template react-ts
```

---

## ২. Tailwind CSS Install করা

```sh
npm install tailwindcss @tailwindcss/vite
```

---

## ৩. `vite.config.ts` Update করা

Tailwind plugin আর path alias (`@`) যোগ করতে হবে:

```ts
import tailwindcss from "@tailwindcss/vite";
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

// https://vite.dev/config/
export default defineConfig({
	plugins: [react(), tailwindcss()],
	resolve: {
		alias: {
			"@": `${import.meta.dirname}/src`,
		},
	},
});
```

---

## ৪. `src/index.css`-এ Tailwind Import করা

```css
@import "tailwindcss";
```

---

## ৫. `tsconfig.app.json` Update করা

Path alias (`@/*`) কাজ করানোর জন্য এখানে `baseUrl` এবং `paths` যোগ করতে হবে:

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "target": "es2023",
    "lib": ["ES2023", "DOM"],
    "module": "esnext",
    "types": ["vite/client"],
    "allowArbitraryExtensions": true,
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",

    /* Path alias */
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },

    /* Linting */
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "erasableSyntaxOnly": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

---

## ৬. `tsconfig.json` Update করা

একই path alias root `tsconfig.json`-এও যোগ করতে হবে:

```json
{
  "files": [],
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

---

## ৭. `@types/node` Install করা

`vite.config.ts`-এ ব্যবহৃত `import.meta.dirname`-এর মতো Node-related type ঠিকভাবে চেনার জন্য:

```sh
npm install -D @types/node
```

---

## ৮. shadcn Init করা

সবশেষে shadcn install/init করা হবে:

```sh
npx shadcn@4.21.0 init
```