# Redux Setup — Counter উদাহরণ দিয়ে

রেফারেন্স: [Redux Toolkit — Quick Start](https://redux-toolkit.js.org/tutorials/quick-start)

---

## ১. প্রজেক্ট তৈরি

প্রথমে একটা সাধারণ React প্রজেক্ট বানিয়ে নিতে হবে:

```sh
npm create vite@latest my-react-app -- --template react-ts
```

এরপর `App.tsx` এ গিয়ে একটা Counter UI বানিয়ে নেওয়া হলো:

```tsx
<>
  <h1>Counter with Redux</h1>
  <div className="flex gap-3 justify-center items-center">
    <button className="btn bg-blue-500 text-white p-2 rounded-sm cursor-pointer">
      Increment
    </button>
    <div className="text-2xl">0</div>
    <button className="btn bg-blue-500 text-white p-2 rounded-sm cursor-pointer">
      Decrement
    </button>
  </div>
</>
```

এখনো এটা শুধু একটা স্ট্যাটিক UI — এখনো Redux যুক্ত করা হয়নি।

---

## ২. Redux ইন্সটল করা

```sh
npm install @reduxjs/toolkit react-redux
```

এরপর, একটা ভালো folder structure রাখার জন্য `src` এর ভেতরে `redux` নামে একটা ফোল্ডার বানিয়ে নেওয়া হয়।

---

## ৩. Store বানানো (`redux/store.ts`)

```ts
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./features/counter/counterSlice";

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

**একটু ব্যাখ্যা:**
- `configureStore` দিয়েই মূল **store** তৈরি হয়। **Reducer ছাড়া store বানানো সম্ভব না** — কারণ store এর কাজই হলো state আর reducer গুলোকে একত্র করে রাখা।
- `RootState` টাইপ বলে দেয় — পুরো অ্যাপে যত state আছে, সবকিছু আসলে দেখতে কেমন (store এর `getState()` মেথড থেকে এই টাইপটা বানানো হয়েছে)। এটা পরে TypeScript এ state access করার সময় কাজে লাগবে।
- `AppDispatch` টাইপ পরে `dispatch` ব্যবহারের সময় দরকার হবে।

> এখানে `counterReducer` নামে import করা হয়েছে, কিন্তু আসল ফাইলে এটা `counterSlice.reducer` হিসেবে **default export** করা ছিল। Default export এর সুবিধা হলো — import করার সময় যেকোনো নাম দেওয়া যায়, তাই `counterReducer` নামে ইচ্ছামতো রাখা হয়েছে।

---

## ৪. App কে Provider দিয়ে wrap করা (`main.tsx`)

Store ব্যবহার করার জন্য পুরো `<App />` কে **`Provider`** (react-redux থেকে আসে) দিয়ে wrap করে দিতে হয়:

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import App from "./App.tsx";
import { Provider } from "react-redux";
import { store } from "./redux/store.ts";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </StrictMode>,
);
```

### Default vs Named Export (ছোট্ট নোট)
- **Default export** করলে import করার সময় নিজের পছন্দমতো নাম দেওয়া যায়, আর তা **curly braces `{}` ছাড়া** import করতে হয়। একটা ফাইল থেকে সর্বোচ্চ **একটাই** default export হতে পারে।
- **Named export** এর ক্ষেত্রে import এর সময় **curly braces `{}`** লাগবেই, এবং নামও ঠিক একই রাখতে হবে (চাইলে `as` দিয়ে rename করা যায়)। একটা ফাইল থেকে **একাধিক** named export করা যায়।
- Redux এ সাধারণত **named export** করাই ভালো অভ্যাস (যেমন actions গুলো)।

---

## ৫. Redux DevTools দিয়ে চেক করা

Redux ঠিকমতো connect হলো কিনা, তা ধাপে ধাপে যাচাই করে নেওয়া ভালো। এর জন্য ব্রাউজার এক্সটেনশন আছে:

👉 [Redux DevTools (Chrome Extension)](https://chromewebstore.google.com/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en)

এই extension দিয়ে ব্রাউজারের console এ একটা "Redux" ট্যাব পাওয়া যায়, যেখানে debug করা যায়। সেখানে যদি **`@@INIT`** অ্যাকশনটা দেখা যায়, তার মানে Redux ঠিকমতো connect হয়ে গেছে। ✅

---

## ৬. Reducer / Slice বানানো

Store connect হয়ে গেলে এবার আসল reducer বানাতে হবে।

এর জন্য `redux` ফোল্ডারের ভেতরে `features` নামে একটা ফোল্ডার, তার ভেতরে `counter` নামে আরেকটা ফোল্ডার, এবং সেখানে `counterSlice.ts` ফাইল বানানো হয়।

> **"Slice" কেন বলা হয়?** পুরো অ্যাপ্লিকেশনকে একটা **পিৎজার** সাথে তুলনা করা যায় — প্রতিটা feature (যেমন counter, user, cart) হলো পিৎজার একেকটা **slice (টুকরা)**। প্রতিটা slice নিজের state আর reducer নিজেই সামলায়।

**`counterSlice.ts` (প্রথম ধাপ):**

```ts
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  count: 0,
};

const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {},
});

export default counterSlice.reducer;
```

এখান থেকে শুধু `reducer` অংশটুকু export করা হয়, কারণ এটাই store এর ভেতরে বসবে (উপরে `store.ts` এ যেমন দেখানো হয়েছে)।

Store এর ভেতরে import করার পর আবার DevTools এর **State tree** ট্যাবে গিয়ে দেখা যাবে `counter` নামে state ঠিকমতো যুক্ত হয়েছে কিনা। যদি কিছু না দেখা যায়, তাহলে উপরের dropdown এ চেক করতে হবে সঠিক project/application সিলেক্ট করা আছে কিনা।

---

## ৭. Actions বানানো

`reducers` অবজেক্টের ভেতরে যত ফাংশন লেখা হয় (নাম যা খুশি দেওয়া যায়), সেগুলোই আসলে **actions**। এই ফাংশনগুলোর প্রথম প্যারামিটার হিসেবে reducer পুরো **state** এর access পায়।

**`counterSlice.ts` (increment/decrement সহ):**

```ts
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  count: 0,
};

const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment: (state) => {
      state.count += 1;
    },
    decrement: (state) => {
      state.count -= 1;
    },
  },
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;
```

এই `increment` আর `decrement` ফাংশনগুলোর ভেতরেই লেখা থাকে — ইউজার অ্যাকশন নিলে (যেমন বাটনে ক্লিক) ঠিক কী business logic চলবে। এগুলো অবশ্যই **named export** করে দিতে হবে, যাতে অন্য ফাইল থেকে ইম্পোর্ট করে ব্যবহার করা যায়।

---

## ৮. Custom Hook বানানো (`redux/hook.ts`)

TypeScript এ `dispatch` আর `state` এর টাইপ ম্যানুয়ালি বারবার লেখার ঝামেলা এড়াতে একটা আলাদা hook ফাইল বানিয়ে নেওয়া হয়:

```ts
import { useDispatch, useSelector } from "react-redux";
import type { AppDispatch, RootState } from "./store";

export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
```

এতে বারবার `(state: RootState) => ...` লিখে টাইপ ঠিক করার দরকার পড়ে না — `useAppSelector` আর `useAppDispatch` ব্যবহার করলেই টাইপ স্বয়ংক্রিয়ভাবে ঠিক থাকে।

---

## ৯. Component থেকে Dispatch করা (`App.tsx`)

```tsx
import { decrement, increment } from "./redux/features/counter/counterSlice";
import type { RootState } from "./redux/store";
import { useAppDispatch, useAppSelector } from "./redux/hook";

function App() {
  const dispatch = useAppDispatch();
  const { count } = useAppSelector((state: RootState) => state.counter);

  const handleIncrement = () => {
    dispatch(increment()); // ⚠️ ফাংশনটা obossoi call করতে হবে — শুধু "increment" লিখলে কাজ করবে না
  };
  const handleDecrement = () => {
    dispatch(decrement());
  };

  return (
    <div>
      <h1>Counter</h1>
      <div className="flex justify-center items-center gap-4">
        <button
          onClick={handleIncrement}
          className="bg-blue-500 py-2 px-3 rounded-sm text-white cursor-pointer hover:bg-blue-600"
        >
          Increment
        </button>
        <div className="text-xl">{count}</div>
        <button
          onClick={handleDecrement}
          className="bg-blue-500 py-2 px-3 rounded-sm text-white cursor-pointer hover:bg-blue-600"
        >
          Decrement
        </button>
      </div>
    </div>
  );
}

export default App;
```

**খেয়াল রাখার বিষয়:**
- `useAppSelector` দিয়ে store থেকে দরকারি state (এখানে `count`) বের করে আনা হয়।
- `useAppDispatch` দিয়ে action dispatch করার ফাংশন পাওয়া যায়।
- `dispatch(increment())` — এখানে `increment()` কে অবশ্যই **কল** করতে হবে (bracket `()` সহ)। শুধু `dispatch(increment)` লিখলে কাজ করবে না — এটা খুবই কমন একটা ভুল।

---

## ১০. Dynamic Payload — হার্ডকোডেড না রেখে

এতক্ষণ প্রতিবার শুধু **১** করে বাড়ছিল (হার্ডকোডেড)। এবার এটাকে **dynamic** করা হলো, যাতে ইচ্ছামতো সংখ্যা দিয়ে বাড়ানো যায়।

**`counterSlice.ts`:**

```ts
reducers: {
  increment: (state, action) => {
    state.count += action.payload;
  },
  decrement: (state) => {
    state.count -= 1;
  },
},
```

**`App.tsx`:**

```tsx
const handleIncrement = (amount: number) => {
  dispatch(increment(amount));
};

// ...

<button onClick={() => handleIncrement(5)}>Increment by 5</button>
<button onClick={() => handleIncrement(1)}>Increment</button>
```

### 🧱 Payload বোঝার সহজ উপায়

**Action** কে ধরে নাও একটা **ট্রাক**, আর **payload** হলো ট্রাকে বোঝাই করা **ইট (bricks)**। ট্রাক (action) শুধু জিনিসটা বহন করে নিয়ে যায়, আর ভেতরে কী মাল (payload) আছে সেটাই reducer এর কাছে আসল কাজের ডেটা। এই payload এর মধ্যে সংখ্যা, স্ট্রিং, এমনকি সরাসরি কোনো অবজেক্টও পাঠানো যায় — এটা অনেক শক্তিশালী একটা ফিচার।

---

## শেষ কথা

> পরবর্তী লেভেলের ডেভেলপার হতে চাইলে অবশ্যই অফিসিয়াল ডকুমেন্টেশন পড়ার অভ্যাস করতে হবে।

**রেফারেন্স:** [https://redux-toolkit.js.org/tutorials/quick-start](https://redux-toolkit.js.org/tutorials/quick-start)