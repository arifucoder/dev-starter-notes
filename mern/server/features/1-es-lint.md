# ESLint গাইড (TypeScript প্রজেক্টের জন্য)

## ESLint কেন দরকার?

আমরা প্রজেক্টটি TypeScript দিয়ে করছি। TypeScript আমাদের অনেক সুবিধা দেয়, যেমন type check করে এবং ভুল করলে error দেখিয়ে মনে করিয়ে দেয়। কিন্তু কোড পরিষ্কার (clean) আছে কিনা বা প্রজেক্টের guideline মানা হচ্ছে কিনা, সেটা দেখে **ESLint**।

কিছু উদাহরণ:

- একটা variable declare করলাম কিন্তু use করলাম না, তখন ESLint warning দেবে।
- Testing-এর জন্য `console.log` লিখলাম, যেটা দিয়ে হয়তো sensitive data leak হতে পারে। এগুলো ধরার জন্যও ESLint দরকার।

ESLint থাকলে project manager যেমন সমস্যা detect করতে পারেন, developer হিসেবে আমিও নিজে সেগুলো খুঁজে ঠিক করতে পারি।

---

## Step 1: Install করা

TypeScript প্রজেক্টের জন্য আমরা **typescript-eslint** ব্যবহার করব: https://typescript-eslint.io/getting-started
(শুধু JavaScript-এর জন্য ESLint-এর আলাদা website আছে।)

```bash
npm install --save-dev eslint @eslint/js typescript typescript-eslint
```

আমার ব্যবহৃত version: `"typescript": "~6.0.0"`, `"typescript-eslint": "^8.70.1"`

> **টিপস:** VS Code-এ file খুললেই error দেখতে চাইলে **ESLint extension** install করতে হবে।

---

## Step 2: Config file বানানো

প্রজেক্টের root-এ `eslint.config.mjs` নামে একটা file বানাব।

Getting started page-এ basic config দেওয়া আছে `tseslint.configs.recommended` দিয়ে। কিন্তু একই page-এর একটু নিচে বলা আছে, `recommended`-এর বদলে `strict` আর `stylistic` ব্যবহার করা যায়। (`strict`-এর মধ্যে recommended-এর সব rules আছে, সাথে আরও কড়া কিছু rules।) আমরা সেটাই করব।

সাথে `console.log` ধরার জন্য `no-console` rule যোগ করব:

```js
import js from "@eslint/js";
import { defineConfig, globalIgnores } from "eslint/config";
import tseslint from "typescript-eslint";

export default defineConfig([
  globalIgnores(["dist/"]), // build folder check করবে না
  {
    files: ["**/*.{js,ts}"],
    extends: [
      js.configs.recommended,
      tseslint.configs.strict,
      tseslint.configs.stylistic,
    ],
    rules: {
      "no-console": "warn", // error চাইলে "error" লিখব
    },
  },
]);
```

> **খেয়াল রাখো:** একই rule দুইবার লেখা যাবে না (যেমন `"no-console": "error"` আর `"no-console": "warn"` একসাথে)। তাহলে শেষেরটাই কাজ করবে। যেকোনো একটা বেছে নিতে হবে।

---

## Step 3: ESLint চালানো

VS Code-এ ESLint-এর error শুধু file খুললেই দেখা যায়। পুরো প্রজেক্টের সব error আর warning একসাথে terminal-এ দেখতে এই command চালাব:

```bash
npx eslint .
```

এরপর যা করব:

- **Warning** গুলো ভালো করে দেখব। যে `console.log` গুলো দরকার সেগুলো রাখব, বাকিগুলো remove করে দেব।
- **Error** গুলো অবশ্যই solve করতে হবে।

কিছু সমস্যা ESLint নিজেই ঠিক করে দিতে পারে:

```bash
npx eslint . --fix
```

---

## Step 4: শুধু `src` folder check করা

Build করার পর ESLint build output (`dist`) সহ check করে ফেলে। ওটা আমাদের লেখা কোড না, generated কোড, তাই check করার দরকার নেই। এজন্য config-এ `globalIgnores(["dist/"])` দিয়েছি। চাইলে শুধু `src` folder-ও check করা যায়:

```bash
npx eslint ./src
```

এটা `package.json`-এর `scripts`-এ যোগ করে দেব (script-এর ভিতরে `npx` লাগে না):

```json
"scripts": {
  "lint": "eslint ./src"
}
```

এখন শুধু এটা চালালেই সব error আর warning পাব:

```bash
npm run lint
```

---

## Step 5: কোনো file-এ rule বন্ধ করা

যেমন আমরা জানি `app.ts` আর `server.ts`-এর `console.log` গুলো ক্ষতিকর না, আর এই file-গুলোতে তেমন নতুন কোড লেখা হবে না। তাই পুরো file-এর জন্য `no-console` বন্ধ করে দিতে পারি।

VS Code-এ **Quick Fix → Disable no-console for the entire file** দিলে file-এর উপরে এটা যোগ হবে:

```ts
/* eslint-disable no-console */
```

> **গুরুত্বপূর্ণ:** নিশ্চিত না হয়ে কখনো rule disable করব না। ESLint use-ই করা হয় যাতে ভুলগুলো ধরা পড়ে।

---

## Rules সম্পর্কে

ESLint website-এ **Rules** page-এ গেলে অনেক rules দেখা যায়। যেমন:

**[`no-unused-vars`](https://typescript-eslint.io/rules/no-unused-vars)**: কোনো unused variable রাখা যাবে না। TypeScript প্রজেক্টে এর version হলো `@typescript-eslint/no-unused-vars`, আর এটা `strict` config-এ আগে থেকেই চালু থাকে।