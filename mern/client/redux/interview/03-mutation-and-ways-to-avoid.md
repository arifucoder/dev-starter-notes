# Mutation ও এটা এড়ানোর উপায় (Immer সহ)

---

## Mutation মানে কী?

**Mutation** মানে হলো — কোনো কিছুর **আসল রূপ (original form) থেকে পরিবর্তিত হয়ে যাওয়া**। প্রোগ্রামিং এ, **mutate** করা মানে হলো — কোনো existing object বা array কে **সরাসরি পরিবর্তন করা**, নতুন কোনো object/array তৈরি না করেই।

JavaScript এ **object** আর **array** হলো **reference type** ডেটা। এই ধরনের ডেটা সহজেই mutate হয়ে যেতে পারে, যদি আমরা সতর্ক না থাকি।

---

## Reference Type কীভাবে Mutate হয়?

```ts
const employee = {
  name: "Mir",
  address: { country: "Bangladesh", city: "Dhaka" },
};

const employee2 = employee;
employee2.name = "Mezba";

console.log(employee);
console.log(employee2);
```

এখানে `employee2 = employee` লেখার মানে হলো — নতুন কোনো object তৈরি হয়নি, বরং `employee2` শুধু `employee` এর মতো **একই মেমোরি রেফারেন্স** ধরে রেখেছে। (JavaScript অবজেক্টগুলোকে **heap memory** তে রাখে এবং ভ্যারিয়েবলে শুধু তার একটা রেফারেন্স/ঠিকানা জমা থাকে।)

তাই `employee2.name = "Mezba"` লিখলে, আসলে `employee` কেও mutate করে ফেলা হচ্ছে — কারণ দুটোই একই object কে point করছে। ফলে দুটোতেই `name: "Mezba"` দেখাবে।

যেহেতু দুটো একই রেফারেন্স ধরে আছে:

```ts
console.log(employee === employee2);
// true
```

---

## সমাধান: Spread Operator দিয়ে নতুন Object বানানো

```ts
const employee2 = {
  ...employee,
  name: "Mezba",
};
```

এখানে `{ ...employee }` করার মাধ্যমে **নতুন একটা object তৈরি হচ্ছে**, এবং `employee` এর ভেতরের জিনিসগুলো এই নতুন object এ **কপি** হয়ে যাচ্ছে। এখন `employee2` আর `employee` এর মতো একই রেফারেন্স ধরে নেই।

তাই এবার:

```ts
console.log(employee === employee2);
// false
```

---

## ⚠️ কিন্তু সমস্যা: Shallow Copy

উপরের spread (`...employee`) আসলে একটা **shallow copy** — মানে এটা শুধু **প্রথম লেভেলের** প্রোপার্টিগুলোই নতুন করে কপি করে। কিন্তু `address` এর মতো **nested object** এর ক্ষেত্রে এটা এখনো একই পুরনো রেফারেন্স ধরে রাখে।

তাই যদি এভাবে লেখা হয়:

```ts
employee2.address.city = "Chittagong";
```

তাহলে দেখা যাবে, `employee` (অরিজিনাল object) এও `city` পরিবর্তন হয়ে **Dhaka থেকে Chittagong** হয়ে গেছে! কারণ `employee2.address` এখনও `employee.address` এর মতো একই object কে point করছে — এটাই **shallow copy এর সমস্যা**।

এখানেই আসে **Deep Copy** আর **Shallow Copy** এর ধারণা:

- **Shallow Copy:** শুধু বাইরের লেভেল কপি হয়, ভেতরের nested object/array গুলো এখনও পুরনো রেফারেন্সই ধরে রাখে।
- **Deep Copy:** ভেতরের প্রতিটা লেভেল, এমনকি nested object/array গুলোও সম্পূর্ণভাবে নতুন করে কপি হয়।

---

## Nested Object সঠিকভাবে কপি করা

Nested object কে mutate হওয়া থেকে বাঁচাতে, তার ভেতরেও আলাদা করে spread (`...`) করে দিতে হয়:

```ts
const employee2 = {
  ...employee,
  name: "Mezba",
  address: {
    ...employee.address,
    city: "Chittagong",
  },
};
```

এখানে `address` এর জন্যও আলাদা করে `{ ...employee.address, city: "Chittagong" }` লেখা হয়েছে — যাতে `address` ও একটা নতুন object হয়, পুরনো রেফারেন্স না থাকে।

**মূলনীতি:** যত লেভেলের nested object থাকবে, প্রতিটা লেভেলেই dot notation ধরে ধরে spread করে কপি করে আনতে হবে — নাহলে সেই লেভেলটা mutate হয়েই যাবে। মূল ডেটার (original data) **integrity** বজায় রাখা জরুরি — একে কখনো নষ্ট হতে দেওয়া যাবে না।

---

## Redux আর Mutation

**Redux এর state কখনো সরাসরি mutate করা যায় না।** এটা Redux এর একটা কড়া নিয়ম — state পরিবর্তন করতে হলে সবসময় একটা **নতুন state object** তৈরি করে দিতে হয়।

কিন্তু প্রতিটা reducer এ nested object এর জন্য ম্যানুয়ালি এভাবে বারবার spread করে করে কপি করাটা বেশ ঝামেলার এবং ভুল হওয়ার সম্ভাবনাও বেশি। এই সমস্যা সমাধানের জন্য Redux Toolkit ভেতরে **Immer** নামে একটা প্যাকেজ ব্যবহার করে, যাতে ডেভেলপার হিসেবে আমরা mutate করার মতো সহজ সিনট্যাক্সে কোড লিখলেও, আসলে কখনোই আসল state mutate হয় না।

### Immer এর উদাহরণ:

```ts
import { produce } from "immer";

const employee = {
  name: "Mir",
  address: { country: "Bangladesh", city: "Dhaka" },
};

const employee2 = produce(employee, (draft) => {
  draft.name = "Mezba";
  draft.address.city = "Chittagong";
});

console.log(employee); // অপরিবর্তিত থাকবে (Dhaka)
console.log(employee2); // নতুন, পরিবর্তিত ভ্যালু (Chittagong)
```

এখানে `produce` ফাংশনের ভেতরে `draft` নামের একটা **অস্থায়ী কপি** পাওয়া যায়, যেটাকে আমরা ইচ্ছামতো সরাসরি "mutate" করার মতো করে লিখতে পারি (`draft.address.city = "Chittagong"`)। কিন্তু Immer পেছন থেকে বুদ্ধি করে **আসল `employee` object কে স্পর্শ না করেই**, একটা সম্পূর্ণ নতুন, immutably-আপডেট করা object (`employee2`) তৈরি করে দেয়।

এই কারণেই Redux Toolkit এর reducer এ `state.count += 1` এর মতো mutate-দেখতে কোড লিখলেও, ভেতরে ভেতরে Immer এটাকে সঠিকভাবে একটা নতুন, immutable state এ রূপান্তর করে দেয়।

📖 আরও পড়ুন: [https://immerjs.github.io/immer/](https://immerjs.github.io/immer/)

---

## সংক্ষেপে

- **Mutation** = আসল object/array কে সরাসরি পরিবর্তন করে ফেলা।
- সাধারণ `=` দিয়ে assign করলে **reference copy** হয় — mutation এর ঝুঁকি থাকে।
- `{ ...obj }` দিয়ে **shallow copy** হয় — কিন্তু nested object এখনও mutate হতে পারে।
- Nested object ঠিকভাবে কপি করতে হলে প্রতিটা লেভেলে আলাদা করে spread করতে হয় (**deep copy**)।
- **Redux এ state সরাসরি mutate করা নিষেধ** — এবং এটা সহজ করার জন্যই Redux Toolkit ভেতরে **Immer** ব্যবহার করে।