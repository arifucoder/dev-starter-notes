# Redux Reducer ও Pure Function

Redux এর reducer সবসময় **pure function** হতে হয়। এটা Redux এর একটা মূল নিয়ম। চলো সহজভাবে বোঝা যাক।

---

## Functional Programming কী?

**Functional Programming (FP)** এমন একটা প্রোগ্রামিং স্টাইল, যেখানে সবকিছুকে **function** এর সাথে তুলনা করে দেখা হয় — অনেকটা গণিতের (Math) ফাংশনের মতো। অর্থাৎ, একটা নির্দিষ্ট ইনপুট দিলে সবসময় একটা নির্দিষ্ট, প্রেডিক্টেবল আউটপুট পাওয়া যাবে।

---

## Pure Function কী?

**Pure function** হলো এমন একটা function, যেখানে:

1. **একই ইনপুট দিলে সবসময় একই আউটপুট পাওয়া যায়** — কোনো ব্যতিক্রম নেই।
2. Function টা তার বাইরের কোনো কিছু (external state, variable) পরিবর্তন করে না — কোনো **side effect** থাকে না।

### উদাহরণ:

```ts
const add = (a, b) => a + b;
```

এখানে `add(5, 2)` যতবারই কল করা হোক না কেন, ফলাফল সবসময় `7`-ই হবে। ইনপুট একই থাকলে আউটপুটও কখনো বদলাবে না:

```ts
add(0, 1); // 1
add(1, 1); // 2
```

এটাই Math এর ফাংশনের মতো — নির্দিষ্ট ইনপুটের জন্য নির্দিষ্ট আউটপুট, প্রতিবার।

---

## Reducer কি তাহলে Pure Function?

Redux Toolkit এর reducer-এ প্রায়ই এভাবে লেখা হয়:

```ts
reducers: {
  increment: (state) => {
    state.count += 1;
  },
  decrement: (state) => {
    state.count -= 1;
  },
},
```

প্রথম দেখায় মনে হতে পারে, এখানে `state` কে সরাসরি **mutate (পরিবর্তন)** করা হচ্ছে (`state.count += 1`), তাহলে এটা কীভাবে pure function হলো? স্বাভাবিকভাবে যেকোনো ভ্যারিয়েবল সরাসরি পরিবর্তন করাটা impure এর লক্ষণ।

**আসল ব্যাপারটা হলো:** Redux Toolkit এর ভেতরে **Immer** নামে একটা লাইব্রেরি কাজ করে। এই Immer-ই আসল কাজটা করে — আমরা দেখতে "mutate" করার মতো কোড লিখলেও, Immer পেছনে পেছনে আসল state কে হাত না দিয়ে, তার একটা নতুন কপি বানিয়ে সেই কপিতে পরিবর্তনগুলো বসিয়ে দেয়। ফলে:

- একই `state` আর একই `action` দিলে সবসময় একই নতুন `state` তৈরি হয় (predictable)।
- Function এর বাইরে অন্য কোথাও কোনো সরাসরি প্রভাব পড়ে না।

তাই Redux Toolkit এর reducer দেখতে mutate করার মতো মনে হলেও, কার্যত এটা **pure function** হিসেবেই কাজ করে।

---

## Impure Function এর উদাহরণ

Impure function হলো এমন function, যেখানে একই ইনপুট দেওয়া সত্ত্বেও **আউটপুট একেকবার একেক রকম হয়**, অথবা function টা বাইরের কোনো ভ্যারিয়েবল সরাসরি পরিবর্তন করে দেয়।

### উদাহরণ ১: বাইরের ভ্যারিয়েবল পরিবর্তন করা

```ts
let total = 0;
const addToTotal = (amount) => (total = total + amount);
```

এখানে `addToTotal` ফাংশনটা বাইরের `total` ভ্যারিয়েবলকে সরাসরি পরিবর্তন করছে — এটা একটা **side effect**, তাই এটা impure।

### উদাহরণ ২: অনির্দিষ্ট (unpredictable) আউটপুট

```ts
const updateDate = () => {
  return new Date();
};

const randomNumber = (amount) => {
  return amount + Math.random();
};
```

- `updateDate()` প্রতিবার কল করলে ভিন্ন ভিন্ন সময় রিটার্ন করবে।
- `randomNumber(5)` প্রতিবার কল করলে ভিন্ন ভিন্ন মান রিটার্ন করবে (কারণ `Math.random()` প্রতিবার নতুন সংখ্যা দেয়)।

একই ইনপুট (`amount = 5`) দেওয়া সত্ত্বেও আউটপুট বারবার বদলে যাচ্ছে — তাই এগুলো **impure function**।

---

## সংক্ষেপে

| | Pure Function | Impure Function |
|---|---|---|
| একই ইনপুটে আউটপুট | সবসময় একই | ভিন্ন হতে পারে |
| বাইরের ভ্যারিয়েবল পরিবর্তন | করে না | করতে পারে |
| Randomness / সময় নির্ভর | না | হতে পারে (`Math.random()`, `new Date()`) |

Functional Programming এ **impure function এড়িয়ে চলাই নিয়ম**। আর এই কারণেই Redux বলে দেয় — reducer সবসময় pure function হতে হবে, যাতে state পরিবর্তন predictable ও নির্ভরযোগ্য (reliable) থাকে।