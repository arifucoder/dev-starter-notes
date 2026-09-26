# JavaScript Runtime: কী কী আছে?

এই পুরো ব্যবস্থাটাকে বলে **JavaScript runtime**। এতে মোট ৬টা অংশ আছে, তিনটা "এলাকায়" ভাগ করা।

![JavaScript runtime-এর অংশগুলো](./js-runtime.png)

> ছবিটা দেখতে `js-runtime.svg` ফাইলটা এই `.md` ফাইলের একই ফোল্ডারে রাখতে হবে।

## JS engine (যেমন V8) এর ভেতরে দুটো জিনিস

- **Call stack:** এখন যে কাজ চলছে তার জায়গা। একটাই আছে, তাই JS একবারে একটা কাজ করে। (গল্পে: মায়ের হাত)
- **Heap:** memory-র বড় গুদাম। Object, array, variable, আর `await`-এ থেমে থাকা function-এর বুকমার্ক এখানে থাকে। (গল্পে: রেসিপি বই রাখার তাক)

## Browser / Node.js যা দেয়

- **Web APIs (browser) / libuv (Node):** timer, network request, file পড়ার মতো অপেক্ষার কাজ পেছনে করে দেয়। (গল্পে: রাইস কুকার)

## দুটো queue

- **Microtask queue:** Promise resolve হলে আর `await`-এর পরের অংশ এখানে বসে। এটা **VIP লাইন**।
- **Callback queue (macrotask):** `setTimeout`, click event ইত্যাদির callback এখানে বসে।

## Event loop

বারবার দেখে call stack খালি কিনা। খালি হলে আগে **microtask queue পুরোটা খালি করে**, তারপর callback queue থেকে একটা নেয়। (গল্পে: দরজায় দাঁড়ানো ছোট ভাই)

## VIP লাইনের প্রমাণ

```javascript
setTimeout(() => console.log("setTimeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
console.log("সাধারণ কাজ");
```

Output:

```
সাধারণ কাজ
Promise
setTimeout
```

আগে call stack-এর সাধারণ কাজ, তারপর VIP লাইনের Promise, সবশেষে setTimeout। দুটোই "০ সেকেন্ডের" হলেও Promise আগে যায়, কারণ microtask queue-এর অগ্রাধিকার বেশি।