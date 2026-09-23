# Shadcn Dark Mode Setup (Vite + React)

> **Reference:** https://ui.shadcn.com/docs/dark-mode/vite

---

## ০. আগে যা লাগবে

Toggle button বানাতে shadcn-এর `button` আর `dropdown-menu` component লাগবে। Project-এ না থাকলে আগে add করে নাও:

```sh
npx shadcn@latest add button dropdown-menu
```

> Icon-এর জন্য `lucide-react` লাগে, যেটা সাধারণত `shadcn init` করার সময়ই install হয়ে যায়। না থাকলে: `npm install lucide-react`

---

## ১. Theme Provider বানানো

`src`-এর ভেতরে `providers` নামে একটা folder বানাও, তার ভেতরে `theme-provider.tsx`।

### `src/providers/theme-provider.tsx`

```tsx
import { createContext, useContext, useEffect, useState } from "react"

type Theme = "dark" | "light" | "system"

type ThemeProviderProps = {
  children: React.ReactNode
  defaultTheme?: Theme
  storageKey?: string
}

type ThemeProviderState = {
  theme: Theme
  setTheme: (theme: Theme) => void
}

const initialState: ThemeProviderState = {
  theme: "system",
  setTheme: () => null,
}

const ThemeProviderContext = createContext<ThemeProviderState>(initialState)

export function ThemeProvider({
  children,
  defaultTheme = "system",
  storageKey = "vite-ui-theme",
  ...props
}: ThemeProviderProps) {
  const [theme, setTheme] = useState<Theme>(
    () => (localStorage.getItem(storageKey) as Theme) || defaultTheme
  )

  useEffect(() => {
    const root = window.document.documentElement

    root.classList.remove("light", "dark")

    if (theme === "system") {
      const systemTheme = window.matchMedia("(prefers-color-scheme: dark)")
        .matches
        ? "dark"
        : "light"

      root.classList.add(systemTheme)
      return
    }

    root.classList.add(theme)
  }, [theme])

  const value = {
    theme,
    setTheme: (theme: Theme) => {
      localStorage.setItem(storageKey, theme)
      setTheme(theme)
    },
  }

  return (
    <ThemeProviderContext.Provider {...props} value={value}>
      {children}
    </ThemeProviderContext.Provider>
  )
}

export const useTheme = () => {
  const context = useContext(ThemeProviderContext)

  if (context === undefined)
    throw new Error("useTheme must be used within a ThemeProvider")

  return context
}
```

### এটা কী করছে?

- **`theme` state** → বর্তমান theme (`light` / `dark` / `system`) রাখে। প্রথমবার `localStorage` থেকে আগের পছন্দ পড়ে, না পেলে `defaultTheme` ব্যবহার করে।
- **`useEffect`** → theme বদলালেই `<html>` tag থেকে পুরনো `light`/`dark` class সরিয়ে নতুনটা বসায়। `system` হলে user-এর OS-এর setting দেখে ঠিক করে।
- **`setTheme`** → নতুন theme `localStorage`-এ save করে, তাই page reload করলেও theme মনে থাকে।
- **`useTheme()`** → যেকোনো component থেকে theme পড়া বা বদলানোর জন্য custom hook।

---

## ২. `main.tsx`-এ ThemeProvider বসানো

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";

import { Provider } from "react-redux";
import { store } from "./redux/store.ts";
import { RouterProvider } from "react-router";
import router from "./routes/index.tsx";
import { ThemeProvider } from "./providers/theme-provider.tsx";

createRoot(document.getElementById("root")!).render(
	<StrictMode>
		<ThemeProvider defaultTheme="dark" storageKey="vite-ui-theme">
			<Provider store={store}>
				<RouterProvider router={router} />
			</Provider>
		</ThemeProvider>
	</StrictMode>,
);
```

`ThemeProvider` সবার বাইরে বসানো হয়েছে, যাতে পুরো app (Redux, Router-এর সব page) theme access করতে পারে। `defaultTheme="dark"` মানে প্রথমবার app খুললে dark mode থাকবে।

---

## ৩. Mode Toggle Button বানানো

### `src/components/mode-toggle.tsx`

```tsx
import { Moon, Sun } from "lucide-react";

import { Button } from "@/components/ui/button";
import {
	DropdownMenu,
	DropdownMenuContent,
	DropdownMenuItem,
	DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import { useTheme } from "@/providers/theme-provider";

export function ModeToggle() {
	const { setTheme } = useTheme();

	return (
		<DropdownMenu>
			<DropdownMenuTrigger asChild>
				<Button variant="outline" size="icon">
					<Sun className="h-[1.2rem] w-[1.2rem] scale-100 rotate-0 transition-all dark:scale-0 dark:-rotate-90" />
					<Moon className="absolute h-[1.2rem] w-[1.2rem] scale-0 rotate-90 transition-all dark:scale-100 dark:rotate-0" />
					<span className="sr-only">Toggle theme</span>
				</Button>
			</DropdownMenuTrigger>
			<DropdownMenuContent align="end">
				<DropdownMenuItem onClick={() => setTheme("light")}>Light</DropdownMenuItem>
				<DropdownMenuItem onClick={() => setTheme("dark")}>Dark</DropdownMenuItem>
				<DropdownMenuItem onClick={() => setTheme("system")}>System</DropdownMenuItem>
			</DropdownMenuContent>
		</DropdownMenu>
	);
}
```

> **Note:** shadcn-এর official doc-এ theme provider রাখা হয় `components/theme-provider.tsx`-এ, কিন্তু আমরা রেখেছি `providers/` folder-এ — তাই import path `@/providers/theme-provider` দিতে হবে (যেটা তুমি ঠিকই দিয়েছ)।

---

## ৪. যেখানে Toggle দেখাতে চাও সেখানে বসানো

যেমন `Navbar.tsx`-এ:

```tsx
import { ModeToggle } from "@/components/mode-toggle";

<ModeToggle />
```

> **ছোট পরামর্শ:** তুমি `"../mode-toggle"` relative path লিখেছিলে — এটা শুধু তখনই কাজ করবে যদি file-টা `components/`-এর ঠিক একটা sub-folder-এ থাকে (যেমন `components/layout/Navbar.tsx`)। অন্য জায়গা থেকে import করলে ভেঙে যাবে। তাই `@/components/mode-toggle` alias ব্যবহার করা বেশি নিরাপদ — যেকোনো file থেকে একই path কাজ করবে।

---

## ⚠️ Dark Mode কাজ না করলে যা Check করবে

Tailwind v4-এ `dark:` class by default OS-এর setting (`prefers-color-scheme`) দেখে কাজ করে, `<html>`-এর `dark` class দেখে না। তাই `src/index.css`-এ এই line টা থাকতে হবে:

```css
@custom-variant dark (&:is(.dark *));
```

`shadcn init` করলে এটা সাধারণত নিজে থেকেই যোগ হয়ে যায় — তবু toggle চাপলে কিছু না বদলালে প্রথমে এটাই check করবে।