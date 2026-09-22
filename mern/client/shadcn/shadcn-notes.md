# Shadcn কী?

**Shadcn (shadcn/ui)** একটা component collection, যেটা **Tailwind CSS** (design/styling) আর **Radix UI**-এর (headless interaction/accessibility) মাঝামাঝি জায়গায় কাজ করে — অর্থাৎ এটা একসাথে দেয়:

**Tailwind (design system) + Radix (interaction/accessibility) = Shadcn**

সাধারণ npm package-এর মতো এটা `node_modules`-এ install হয়ে থাকে না — বরং shadcn CLI দিয়ে component-এর **আসল source code**-টা সরাসরি তোমার project-এ copy হয়ে আসে। ফলে component-টা পুরোপুরি তোমার নিয়ন্ত্রণে থাকে, চাইলে নিজের মতো edit করা যায়।

> **Installation guide:** http://ui.shadcn.com/docs/installation/vite

---

## UI Library-দের ৩টা ধরন

Frontend-এ UI বানানোর জন্য মোটামুটি ৩ ধরনের library/tool থাকে:

### ১. CSS Framework (শুধু Design/Style দেয়)
এগুলো শুধু ready-made class/style দেয়, কোনো interaction logic (dropdown খোলা-বন্ধ, focus handle করা ইত্যাদি) দেয় না।

| Name | Definition |
|---|---|
| **Tailwind CSS** | Utility-first CSS framework — ছোট ছোট class (`flex`, `p-4`, `text-red-500`) দিয়ে সরাসরি HTML-এই design করা যায়। |
| **Bootstrap (CSS)** | পুরনো ও জনপ্রিয় CSS framework, pre-built class (`btn`, `container`, `row`) দিয়ে দ্রুত layout/design করা যায়। |
| **Bulma** | Tailwind-এর মতোই আরেকটা utility/class-based CSS framework, তুলনামূলক কম জনপ্রিয় কিন্তু simple। |

### ২. Headless UI (শুধু Interaction/Accessibility দেয়, Design দেয় না)
এগুলো component-এর **behavior** (যেমন: popup খোলা-বন্ধ হওয়া, keyboard navigation, focus trap, accessibility) সামলায়, কিন্তু কোনো ready-made **design/style** দেয় না — চোখে দেখতে একদম plain/basic লাগে, ডিজাইন নিজেকে করতে হয়।

| Name | Definition |
|---|---|
| **Radix UI (Primitives)** | সবচেয়ে জনপ্রিয় headless UI library — Dialog, Dropdown, Tooltip-এর মতো component-এর interaction ও accessibility handle করে, design বলতে গেলে থাকেই না। |
| **React Aria (Adobe)** | Adobe-এর বানানো headless UI hook library, accessibility-তে খুবই strong, বড় বড় enterprise app-এ ব্যবহৃত হয়। |
| **Headless UI (Tailwind Labs)** | Tailwind-এর নিজেদের বানানো headless component library, মূলত Tailwind-এর সাথে ব্যবহারের জন্যই তৈরি। |
| **Ariakit** | আরেকটা lightweight headless component library, Radix-এর মতোই accessibility-focused। |

### ৩. Component Library (Design + Interaction — দুটোই Ready-made)
এগুলোতে design আর interaction — দুটোই আগে থেকে তৈরি করা থাকে, শুধু import করে ব্যবহার করলেই হয়।

| Name | Definition |
|---|---|
| **Bootstrap (Components)** | CSS framework-এর পাশাপাশি ready-made styled component (Modal, Navbar, Carousel)-ও দেয়। |
| **Material UI (MUI)** | Google-এর Material Design অনুযায়ী বানানো React component library, enterprise/dashboard app-এ খুব জনপ্রিয়। |
| **Ant Design** | চাইনিজ কোম্পানি Alibaba-এর বানানো, admin panel/dashboard বানানোর জন্য খুব জনপ্রিয় একটা component library। |
| **Chakra UI** | Simple, accessible এবং customizable React component library — ভালো developer experience-এর জন্য পরিচিত। |

---

## Shadcn কেন আলাদা/জনপ্রিয়?

- Design-এর জন্য নিচে **Tailwind CSS**, আর interaction/accessibility-এর জন্য নিচে **Radix UI** ব্যবহার করে — অর্থাৎ দুইটার সেরা দিকগুলো একসাথে পাওয়া যায়।
- Component code সরাসরি project-এ **copy** হয়ে আসে (dependency হিসেবে না), তাই পুরোপুরি customize করা যায় — কোনো library-র "black box" এর ভেতরে আটকে থাকতে হয় না।