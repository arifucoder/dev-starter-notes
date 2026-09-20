# Redux শেখার নোট

## Redux কী?

Redux হলো **React** এর জন্য একটি **state management** প্যাকেজ। তবে Redux শুধু React এর জন্যই না, প্রায় যেকোনো JS লাইব্রেরির সাথেই ব্যবহার করা যায়।

মূল শেখার রিসোর্স: [redux-toolkit.js.org](https://redux-toolkit.js.org/)

---

## কেন State Management দরকার?

Frontend ডেভেলপমেন্টের কথা আসলেই state management এর কথা চলে আসে। একজন ব্যবহারকারী যখন সাইট ভিজিট করে, তখন তার প্রতিটি অ্যাকশন (যেমন — কোনো ফর্ম সাবমিট করা বা বাটনে ক্লিক করা) মূলত state পরিবর্তন করে। এই state-ই পরে ডেটাবেজে যায়।

State ঠিকমতো manage করতে পারলে:
- **Developer** হিসেবে আমাদের কাজের চাপ (overhead) কমে যায়।
- **End user** হিসেবে ব্যবহারকারী অনেক smooth অভিজ্ঞতা পায়।

তাই state management খুবই গুরুত্বপূর্ণ একটি বিষয়।

---

## কেন Redux ব্যবহার করব?

State management এর জন্য Redux ছাড়াও আরও অনেক অপশন আছে — যেমন **Zustand, Recoil, MobX, RxJS** ইত্যাদি।

### Redux বেছে নেওয়ার কারণ:
1. Redux সবচেয়ে **পুরনো এবং জনপ্রিয়** লাইব্রেরি — আগে এত বিকল্প ছিলও না। একজন নতুন শিক্ষার্থী হিসেবে দ্রুত চাকরি পেতে হলে জনপ্রিয় জিনিসটা আগে শেখা ভালো। Redux শেখা হয়ে গেলে বাকিগুলো তুলনামূলক সহজেই শেখা যায়।
2. অনেক প্রোডাকশন অ্যাপ্লিকেশন এখনো Redux ব্যবহার করে।
3. দ্রুত value generate করতে পারে (দ্রুত কাজ করা যায়)।

### Redux এর সীমাবদ্ধতা (Drawback):
- Redux এ **boilerplate কোড বেশি** লিখতে হয়। তাই ছোট অ্যাপ্লিকেশনের জন্য Redux ব্যবহার না করাই ভালো।

---

## কেন Redux Toolkit (RTK) শিখব?

আগে **react-redux** নামে একটি প্যাকেজ ছিল, যেটাকে এখন *legacy redux* বলা যায়। এখানে reducer, action ইত্যাদি সব ম্যানুয়ালি বানাতে হতো। এটি ছিল **unopinionated** — অর্থাৎ, ডেভেলপার যেভাবে খুশি সেভাবে কোড লিখতে পারতেন।

**Redux Toolkit** হলো **opinionated** — অর্থাৎ, একটা নির্দিষ্ট নিয়ম মেনে কাজ করতে হয়। Redux Toolkit এর সাথে আরেকটি চমৎকার জিনিস পাওয়া যায় — **RTK Query**।

- **RTK Query** হলো একটি data fetching টুল, অনেকটা **TanStack Query** এর মতো।
- React এ ডেটা ফেচ করার কোনো built-in, প্রপার সলিউশন নেই। আমরা `useState` ব্যবহার করি ঠিকই, কিন্তু এটা প্রকৃত সমাধান না।

**সংক্ষেপে:**
- **Redux Toolkit** → local state management করব।
- **RTK Query** → data fetching করব।

---

## State এর Communication পদ্ধতি

State কমিউনিকেশন মূলত দুই ধরনের হতে পারে:

### ১. Bi-Directional (দ্বিমুখী)
দুটি component এর মধ্যে state দুই দিকেই flow করতে পারে — একটি component থেকে অন্যটিতে, আবার সেটি থেকে প্রথমটিতেও।

### ২. Uni-Directional (একমুখী)
দুটি component এর মধ্যে state শুধু একটি নির্দিষ্ট দিকেই (single flow) প্রবাহিত হয়।

**সমস্যা:** Bi-directional flow তে যখন একাধিক component এর মধ্যে state link করা লাগে, তখন সেটা manage করা অনেক কঠিন হয়ে যায়।

### Uni-Directional flow এর সমস্যা (Prop Drilling)

ধরা যাক আমাদের component structure এরকম:

```
grandparent -> parent -> child
```

- Counter এর state generate হচ্ছে `grandparent` component এ, কিন্তু সেটা দেখাতে হবে `child` এ। তাই React এ **prop drilling** এর মাধ্যমে এটা `parent` হয়ে `child` পর্যন্ত পাঠাতে হয় — যদিও `parent` এর এই ডেটা দরকার নেই, তবুও মাঝখান দিয়ে পাঠাতেই হয়।
- যদি `child` থেকে (plus/minus বাটনের মাধ্যমে) এই state control করতে চাই, তাহলে setter function-ও একইভাবে drill করে পাঠাতে হয় — `child` থেকে `parent`, তারপর `grandparent` পর্যন্ত।
- **Sibling সমস্যা:** ধরা যাক `parent` এর নিচে দুটি `child` component sibling হিসেবে আছে। এই দুই sibling এর মধ্যে communication করতে চাইলে state কে "lift up" করতে হয় — অর্থাৎ প্রথমে `parent` এর কাছে নিয়ে, তারপর দ্বিতীয় child এ পাঠাতে হয়। এমনকি `parent` এর নিজের এই state এর দরকার না থাকলেও।

এই সমস্যাগুলোই Redux এর **Flux Architecture** এর মাধ্যমে সমাধান করে।

---

## Flux Architecture

Flux Architecture এর মূল ধারণা হলো — একটি কেন্দ্রীয় **Store** থাকবে।

- **Store**: সব ডেটা এসে এখানে জমা হয়। Store থেকে ডেটা যায় **View** এর কাছে।
- **View**: প্রতিটি React component-ই একেকটি view। একটি বা একাধিক component/view Store এর সাথে connected থাকতে পারে।

যেহেতু ডেটা centralized (কেন্দ্রীভূত), তাই জটিলতা কমে যায়। কোনো view তে ডেটা generate হলে সেটা store এ রাখা গেলে, অন্য যেকোনো view থেকেই সেটা দেখানো যায়।

**কিন্তু:** Unidirectional flow এ store থেকে view এ ডেটা সরাসরি যেতে পারে, কিন্তু view থেকে store এ সরাসরি ডেটা পাঠানো যায় না। এর সমাধান হলো —

1. View থেকে একটি **Action** generate করা হয়। Action হলো অনেকটা request এর মতো — এটি একটি **plain object**।
2. Action একা সরাসরি store এ ডেটা রাখতে পারে না — এর জন্য দরকার **Dispatcher**।
3. Dispatcher অনেকটা একটি **registry** এর মতো কাজ করে। যত রকম action perform করা সম্ভব, তার সব callback dispatcher এর কাছে থাকে। আমরা যে action generate করি, সেটা dispatcher এর মধ্যে যায়, এরপর dispatcher সেটা store এর মধ্যে রাখে।

এভাবেই পুরো প্রসেসটা **unidirectional** হয়ে যায় — Store থেকে সরাসরি component এ state যায়, কিন্তু View থেকে store এ যেতে হলে অবশ্যই Dispatcher এর মাধ্যমে যেতে হয়।

---

## Redux এর ভেতরের কার্যপ্রণালী (Inner Working)

একটি React-Redux অ্যাপ্লিকেশনে সাধারণত **একটাই কেন্দ্রীয় Store** থাকা উচিত। একাধিক store বানানো সম্ভব হলেও এটা ভালো practice না।

Store এর ভেতরে দুটি জিনিস থাকে:
1. **State** — পুরো অ্যাপ্লিকেশনের ডেটা।
2. **Reducer** — state এ কী পরিবর্তন আসবে, কীভাবে আসবে, সেটা নির্ধারণ করে।

**Reducer** এর কাজ: একটি action আসলে, তার কাছে আগে থেকেই বিদ্যমান state এর access থাকে। যখন কোনো action আসে, reducer সেই অনুযায়ী পরিবর্তন করে একটি নতুন state generate করে।

### উদাহরণ (Counter):

সহজ একটা analogy দিয়ে বোঝা যাক। ধরো, **Store** হলো একটা **ব্যাংক**, আর তোমার UI (View) হলো তোমার **মোবাইলের ব্যাংকিং অ্যাপ**। তুমি অ্যাপে ব্যালেন্স দেখো ঠিকই, কিন্তু টাকা আসলে জমা থাকে ব্যাংকে (Store)। ব্যালেন্স বাড়াতে চাইলে তুমি সরাসরি অ্যাপের সংখ্যাটা এডিট করে দিতে পারো না — তোমাকে একটা **request (Action)** পাঠাতে হয়, ব্যাংক (Store via Reducer) সেটা প্রসেস করে, তারপর নতুন ব্যালেন্স অ্যাপে (View) দেখায়।

এবার আমাদের Counter এর উদাহরণে আসি। ধরা যাক UI তে আছে:

```
-   0   +
```

এখানে **০ (শূন্য)** সংখ্যাটাই আসলে **state**, যেটা store এ জমা আছে।

ধাপে ধাপে পুরো প্রক্রিয়াটা দেখা যাক:

1. **Subscribe:** আমাদের এই view (UI) টা store এর সাথে connected — একে বলে **subscribe** করা। মানে, view সবসময় store এর দিকে "নজর রাখছে" (listen করছে), যাতে state পরিবর্তন হলেই সাথে সাথে নতুন value দেখাতে পারে।
2. **Click → Event:** ব্যবহারকারী `+` বাটনে ক্লিক করলো, এতে একটি **event** trigger হলো।
3. **Dispatch:** ওই event এর handler এর ভেতর আমরা একটি action **dispatch** করি — অর্থাৎ, "আমি increment করতে চাই" — এই মেসেজটা পাঠিয়ে দিই।
4. **Action:** এই action এর মধ্যে একটি **type** থাকে, যেমন — `increment`। এই type দেখেই বোঝা যায় কী করতে হবে।
5. **Reducer পড়ে বোঝে:** Action টা reducer এর কাছে পৌঁছায়। Reducer, action এর `type` দেখে বুঝে যায় কোন logic চালাতে হবে (এক্ষেত্রে "increment" এর logic)।
6. **নতুন State তৈরি:** Reducer তার কাছে থাকা বর্তমান state (`0`) ব্যবহার করে নতুন state তৈরি করে — অর্থাৎ `0` থেকে `1`।
7. **View আপডেট:** Store এর state আপডেট হওয়ার সাথে সাথে, যেহেতু view টা store কে subscribe করে রেখেছিলো, তাই সে ব্যাপারটা টের পায় এবং স্ক্রিনে নতুন value `1` দেখায়।

**সংক্ষেপে ফ্লো:** Click → Event → Action Dispatch → Reducer নতুন State বানায় → Store Update → View তে দেখা যায়।

এটাই পুরো Redux সাইকেল (cycle)।

### মনে রাখার মতো ৩টি মূল বিষয়:

| বিষয় | কাজ |
|---|---|
| **Reducer** | কীভাবে করবে (How) — business logic |
| **Action** | কী করবে (What) — যেমন increment, decrement |
| **Store** | কী জমা রাখবে (What to store) |

### Payload

Action এর মধ্যে একটি জিনিস থাকে — **payload**। ধরা যাক আমরা শুধু ১ না বাড়িয়ে ৫ increment করতে চাই — এই অতিরিক্ত (additional) ডেটাটা payload এর মধ্যে বসিয়ে reducer এর কাছে পাঠানো হয়। তখন reducer ১ না বাড়িয়ে ৫ বাড়াবে।

---

## Extra / গুরুত্বপূর্ণ কিছু বাড়তি তথ্য

- **Redux Toolkit (RTK)** এখন Redux ব্যবহারের **অফিসিয়াল ও রিকমেন্ডেড উপায়**। প্লেইন/legacy Redux দিয়ে নতুন প্রজেক্ট শুরু করা এখন সাধারণত নিরুৎসাহিত করা হয়।
- RTK এর `createSlice` ফাংশন একসাথে action ও reducer জেনারেট করে দেয়, ফলে boilerplate অনেক কমে যায় — এটাই RTK এর সবচেয়ে বড় সুবিধা এবং plain Redux এর boilerplate সমস্যার সমাধান।
- RTK এর ভেতরে **Immer** লাইব্রেরি ব্যবহার করা হয়, যার ফলে reducer এর ভেতরে state কে সরাসরি "mutate" করার মতো কোড লেখা গেলেও, ভেতরে ভেতরে এটা immutably (একটি নতুন state object তৈরি করে) কাজ করে।
- Redux এর তিনটি মূল নীতি (Three Principles):
  1. **Single source of truth** — পুরো অ্যাপের state একটাই store এ থাকে।
  2. **State is read-only** — state পরিবর্তনের একমাত্র উপায় action dispatch করা।
  3. **Changes are made with pure functions** — reducer সবসময় pure function হতে হবে (same input দিলে same output দেবে, কোনো side-effect থাকবে না)।
- ছোট বা মাঝারি অ্যাপ্লিকেশনে global state এর প্রয়োজন না থাকলে React এর নিজস্ব **Context API** অথবা **Zustand** এর মতো হালকা লাইব্রেরি দিয়েও কাজ চালানো যায়।
- **Redux DevTools** নামে ব্রাউজার এক্সটেনশন আছে, যেটা দিয়ে state এর প্রতিটি পরিবর্তন time-travel debugging সহ দেখা যায় — শেখার সময় এটা ব্যবহার করলে অনেক সুবিধা হয়।
- **Async কাজ** (যেমন API call) এর জন্য plain Redux এ **middleware** (যেমন `redux-thunk`) লাগত। RTK Query এই async data-fetching এর ঝামেলাটাই সহজ করে দেয় — caching, loading/error state, re-fetching ইত্যাদি built-in ভাবে হ্যান্ডল করে।

**রেফারেন্স:**
- [https://redux-toolkit.js.org/](https://redux-toolkit.js.org/)
- [https://redux.js.org/tutorials/essentials/part-1-overview-concepts](https://redux.js.org/tutorials/essentials/part-1-overview-concepts)