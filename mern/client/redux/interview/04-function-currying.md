# Function Currying

## সাধারণ Function

আমরা সাধারণত এভাবে function লিখি — একসাথে সবগুলো parameter নিয়ে:

```ts
// normal function
const add = (a, b) => a + b;
```

এটা ঠিকই আছে, কাজও করে। কিন্তু একটা ভালো নিয়ম হলো — **function এ যত কম parameter থাকবে, ততই ভালো**। কারণ parameter কম থাকলে function টা তুলনামূলক বেশি **independent, reusable, আর সহজে বোঝা যায়** — dependency আর জটিলতা কমে যায়।

**Currying** হলো এমন একটা টেকনিক, যেখানে একাধিক parameter নেওয়া একটা function কে ভেঙে, **একবারে একটা করে parameter নেয়** এমন কয়েকটা function এর chain এ রূপান্তর করা হয়।

---

## Curried Function

```ts
// curried function
const add = (a) => (b) => a + b;

console.log(add(3)(5)); // 8
```

এটাকে সাধারণ (non-arrow) function দিয়ে লিখলে দেখতে এরকম হবে — এতে বোঝা সহজ হয় যে ভেতরে ভেতরে আসলে কী ঘটছে:

```ts
function add(a) {
  return function (b) {
    return a + b;
  };
}
```

এখানে `add(3)` কল করলে সেটা প্রথমে `a = 3` মনে রেখে একটা **নতুন function রিটার্ন করে**। এরপর সেই রিটার্ন হওয়া function কে `(5)` দিয়ে কল করলে তখন `b = 5` নিয়ে হিসাব হয়ে `8` রিটার্ন হয়। অর্থাৎ, একসাথে দুটো parameter না নিয়ে, একটার পর একটা করে parameter নেওয়া হচ্ছে।

---

## Real-life উদাহরণ: Discount ক্যালকুলেশন

### সমস্যা — সাধারণ function দিয়ে

```ts
const totalPrice = (amount, discount) => amount - amount * discount;

console.log(totalPrice(100, 0.3));
console.log(totalPrice(59, 0.3));
```

এখানে প্রতিবার calculate করার সময় `discount` (0.3) বারবার দিতে হচ্ছে, যদিও discount টা হয়তো একই থাকছে (যেমন — একটা নির্দিষ্ট সেল ক্যাম্পেইনে সব প্রোডাক্টের discount 30%)। এই বারবার একই ভ্যালু পাস করাটাই এখানে অপ্রয়োজনীয় ঝামেলা (**repetition**)।

### সমাধান — Currying দিয়ে

```ts
const totalPrice = (discount) => (amount) => amount - amount * discount;

const withDiscount = totalPrice(0.3); // discount ফিক্স করে দেওয়া হলো

console.log(withDiscount(100));
console.log(withDiscount(200));
console.log(withDiscount(250));
```

এখানে প্রথমে `totalPrice(0.3)` কল করে `discount = 0.3` টা একবার fix করে একটা নতুন function (`withDiscount`) বানিয়ে নেওয়া হলো। এরপর যতবার খুশি শুধু `amount` দিয়ে `withDiscount(...)` কল করা যাচ্ছে, বারবার discount দেওয়ার দরকার নেই। এভাবে multi-parameter function কে ভেঙে একবারে **এক parameter নেওয়া** function এ রূপান্তর করাই হলো currying এর মূল ধারণা।

---

## আরও কিছু উদাহরণ

### ১. Greeting Message বানানো

```ts
const greet = (greeting) => (name) => `${greeting}, ${name}!`;

const sayHello = greet("Hello");

console.log(sayHello("Mir")); // Hello, Mir!
console.log(sayHello("Mezba")); // Hello, Mezba!
```

এখানে `greeting` ("Hello") একবার ফিক্স করে দেওয়ার পর, শুধু `name` পাল্টে পাল্টে বারবার ব্যবহার করা যাচ্ছে — অনেকটা একটা রেডিমেড টেমপ্লেট function তৈরি করার মতো।

### ২. Redux Action এর সাথে ছোট্ট কানেকশন

আগের নোটে আমরা `handleIncrement(amount)` এর মতো ফাংশন দেখেছিলাম, যেখানে বাটনে ক্লিক করলে ভিন্ন ভিন্ন amount দিয়ে dispatch করা হতো। এটাও অনেকটা currying-এর মতোই ধারণা — একটা ভ্যালু (এখানে amount) আগে থেকেই বাঁধা (bind) করে রাখা:

```ts
const multiplyBy = (factor) => (num) => num * factor;

const double = multiplyBy(2);
const triple = multiplyBy(3);

console.log(double(10)); // 20
console.log(triple(10)); // 30
```

এখানে `double` আর `triple` — দুটোই মূলত একই `multiplyBy` function থেকে তৈরি, শুধু `factor` ভিন্ন ভিন্নভাবে fix করে দেওয়া হয়েছে। এভাবে currying দিয়ে একই মূল logic থেকে ছোট ছোট, নির্দিষ্ট কাজের জন্য বিশেষায়িত (specialized) function বানানো যায় — একে বলে **partial application**।

---

## সংক্ষেপে

- **Currying** = একাধিক parameter নেওয়া function কে, একবারে একটা parameter নেয় এমন কয়েকটা function এর chain এ রূপান্তর করা।
- সুবিধা:
  - কম parameter → function সহজে বোঝা ও পুনর্ব্যবহার (reuse) করা যায়।
  - কমন ভ্যালু (যেমন discount, factor) একবার fix করে, বারবার নতুন করে দেওয়া লাগে না।
  - একই মূল function থেকে বিশেষায়িত (specialized) ছোট function তৈরি করা যায় (**partial application**)।