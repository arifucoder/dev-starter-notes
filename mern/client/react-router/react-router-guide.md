# React Router

> **Reference:** https://reactrouter.com/home

---

## React Router-এর ৩টা Mode

React Router-এ মূলত ৩টা option/mode আছে:

1. **Declarative**
2. **Data**
3. **Framework**

---

## Remix আর React Router-এর সম্পর্ক (একটু ইতিহাস)

React Router যেই team বানিয়েছে, তারাই **Remix** নামের আরেকটা framework-ও বানিয়েছিল, পরে **Shopify** Remix-কে কিনে নেয়। **২০২৪ সালে** Remix v2-এর সব feature **React Router v7**-এর সাথে merge হয়ে যায়, আর তখনই React Router-এ **"Framework Mode"** যোগ হয়। এরপর **২০২৫ সালে** Remix team আবার নতুন করে **Remix v3** আনার ঘোষণা দেয়, যেটা React একদম বাদ দিয়ে **Preact-এর একটা fork** ব্যবহার করছে — অর্থাৎ এখন Remix আর React Router সম্পূর্ণ আলাদা ecosystem হয়ে গেছে।

Framework mode আসলে পুরোপুরি **Vite দিয়ে React install করার মতোই** একটা নির্দিষ্ট project structure follow করতে বাধ্য করে — এটা একটা routing library-এর জন্য অনেক বেশি robust/opinionated হয়ে যায়। তাই আমরা **Framework mode ব্যবহার করব না।**

---

## Declarative vs Data Mode

- **Declarative** → অনেক আগের/পুরনো ধরনের ব্যবহার, লেখা সহজ কিন্তু control তুলনামূলক কম।
- **Data** → routing-এর উপর আরও বেশি control পাওয়া যায় (যেমন: dynamic sidebar দেখানো, route-এর সাথে data loading যুক্ত করা ইত্যাদি)।

আমরা **Data mode** ব্যবহার করব।

---

## Install করা

> Guide: https://reactrouter.com/start/data/installation

```sh
npm i react-router
```

---

## Router Setup

`src` folder-এর ভেতরে `routes` নামে একটা folder বানাও, তার ভেতরে `index.tsx` (JSX render করা লাগবে তাই `.tsx`)।

### `routes/index.tsx`

```tsx
import { createBrowserRouter } from "react-router";

const router = createBrowserRouter([
	{
		path: "/",
		element: <div>Hello World</div>,
	},
]);

export default router;
```

### `main.tsx`-এ Provider-এর ভেতরে বসানো

```tsx
createRoot(document.getElementById("root")!).render(
	<StrictMode>
		<Provider store={store}>
			<RouterProvider router={router} />
		</Provider>
	</StrictMode>,
);
```

খেয়াল রাখতে হবে — **`RouterProvider`-কে অবশ্যই Redux-এর `Provider`-এর ভেতরে (child হিসেবে) বসাতে হবে**, বাইরে না। কারণ `RouterProvider` দিয়ে যত page/component render হবে (menu, layout, সব route-এর সব component), সেগুলো সবই `Provider`-এর ভিতরে থাকলে তবেই Redux store পুরোপুরি access করতে পারবে — অর্থাৎ যেকোনো page/component থেকে `useSelector`, `useDispatch` ঠিকভাবে কাজ করবে। `RouterProvider`-কে `Provider`-এর বাইরে বসালে ভিতরের কোনো page/component Redux store access করতে পারবে না।

---

## `element` vs `Component`

```tsx
import App from "@/App";
import { createBrowserRouter } from "react-router";

const router = createBrowserRouter([
	{
		path: "/",
		// element: <App />,
		Component: App,
	},
]);

export default router;
```

- **`element`** → এখানে সরাসরি JSX (`<App />`) দিতে হয়। ছোটখাটো কিছু দেখাতে হলে এটা ব্যবহার করা যায়।
- **`Component`** → এখানে component-টার **reference** (JSX না করেই, `App`) দিলেই হয়। পুরো একটা component render করতে হলে `Component` ব্যবহার করাই ভালো।

দুইটার যেকোনোটা ব্যবহার করা যায়, তবে পুরো component render করার সময় `Component` বেশি clean।

---

## Nested Routes (Layout-এর ভেতরে Page আনা)

Header/Footer-সহ একটা layout-এর ভেতরে আলাদা আলাদা page দেখাতে চাইলে `children` ব্যবহার করা হয়:

```tsx
import App from "@/App";
import Task from "@/pages/Task";
import Users from "@/pages/Users";
import { createBrowserRouter } from "react-router";

const router = createBrowserRouter([
	{
		path: "/",
		Component: App, // এটা মূলত layout, এর children গুলোই আসলে page
		children: [
			{
				path: "tasks",
				Component: Task,
			},
			{
				path: "users",
				Component: Users,
			},
		],
	},
]);

export default router;
```

Child route-এর `path`-এর শুরুতে `/` দিতে হয় না — parent-এর path-এর সাথে নিজে থেকেই জুড়ে যায়। তাই এখানে final URL হবে `/tasks` আর `/users`।

---

## `index: true` দিয়ে Default (Home) Page ঠিক করা

```tsx
const router = createBrowserRouter([
	{
		Component: App, // path নেই → এটা শুধু একটা layout route
		children: [
			{
				index: true, // "/" এ গেলে Task দেখাবে
				Component: Task,
			},
			{
				path: "tasks", // "/tasks" এ গেলেও Task দেখাবে
				Component: Task,
			},
			{
				path: "users",
				Component: Users,
			},
		],
	},
]);
```

- **`index: true`** দিলে সেই route-টাই হয়ে যায় parent-এর **default/home** page। তাই `/` এ গেলে layout (Navbar ইত্যাদি)-সহ `Task` page load হবে।
- অনেক সময় আমরা চাই — home page (`/`)-এ যেই page দেখাচ্ছে, সেটা তার নিজের URL (`/tasks`)-এও পাওয়া যাক। তখন `index: true` route-এর পাশাপাশি `path: "tasks"` দিয়ে একই `Task` component আরেকবার যোগ করতে হয়। এতে `/` আর `/tasks` — দুই জায়গাতেই `Task` page দেখাবে।
- Parent route-এ `path` না দিলে সেটাকে বলে **layout route** — এর নিজের কোনো URL থাকে না, এটা শুধু children-দের চারপাশে layout (Navbar, Footer) দেওয়ার কাজ করে। তাই children-দের URL হবে সরাসরি `/`, `/tasks`, `/users`।

---

## `App.tsx` — Layout

```tsx
import { Outlet } from "react-router";
import Navbar from "./components/layout/Navbar";

function App() {
	return (
		<>
			<Navbar />
			<Outlet />
		</>
	);
}

export default App;
```

`Outlet` কে সহজভাবে বুঝতে চাইলে — একটা বড় shopping mall-এর ভেতরে ছোট ছোট আলাদা দোকান যেমন থাকে, সেই দোকানগুলোর জায়গাটাই হলো `Outlet` — মূল layout (mall) একই থাকে, শুধু ভেতরের content (দোকান/page) পরিবর্তন হতে থাকে।

---

## `Link` / `NavLink` দিয়ে Navigation

```tsx
<NavLink to="/users">Users</NavLink>
<NavLink to="/tasks">Tasks</NavLink>
```

- **`Link`** → শুধু page navigate করে, কোনো extra styling দেয় না।
- **`NavLink`** → `Link`-এর মতোই কাজ করে, কিন্তু এর সাথে বাড়তি সুবিধা হলো — কোন link-টা এখন **active** (বর্তমান page), সেটা বুঝে নিজে থেকেই একটা `active` class যোগ করে দেয়, ফলে active menu item আলাদা style দেওয়া সহজ হয়।

> **ছোট Tip:** `<NavLink to="/">` সব page-এই active দেখাবে, কারণ সব URL-ই `/` দিয়ে শুরু হয়। এটা ঠিক করতে `end` prop দিতে হয়: `<NavLink to="/" end>Home</NavLink>` — তখন শুধু ঠিক `/`-এ থাকলেই active হবে।