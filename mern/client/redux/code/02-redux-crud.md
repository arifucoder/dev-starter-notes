# Redux দিয়ে Todo (Task Manager) অ্যাপ বানানো

এই নোটে আমরা Redux Toolkit ব্যবহার করে একটা ছোট **Task Manager** অ্যাপ বানানোর পুরো প্রসেসটা ধাপে ধাপে দেখব — Task তৈরি, দেখানো, complete টগল করা, ডিলিট করা এবং priority অনুযায়ী filter করা।

---

## ১. Task Slice বানানো

`src/redux/features/task/taskSlice.ts` ফাইলে শুরুতে একটা বেসিক স্ট্রাকচার বানানো হলো:

```ts
import type { RootState } from "@/redux/store";
import { createSlice } from "@reduxjs/toolkit";

interface ITask {
  id: string;
  title: string;
  description: string;
  dueDate: string;
  isCompleted: boolean;
  priority: "high" | "medium" | "low";
}

interface IInitialState {
  tasks: ITask[];
  filter: "all" | "high" | "medium" | "low";
}

const initialState: IInitialState = {
  tasks: [
    {
      id: "dasdas",
      title: "initialize frontend",
      description: "create home page and routing",
      dueDate: "2026-11-26",
      isCompleted: false,
      priority: "high",
    },
  ],
  filter: "all",
};

const taskSlice = createSlice({
  name: "task",
  initialState,
  reducers: {},
});

export const selectTasks = (state: RootState) => {
  return state.todo.tasks;
};

export const selectFilter = (state: RootState) => {
  return state.todo.filter;
};

export default taskSlice.reducer;
```

শুরুর জন্য একটা sample task হার্ডকোড করে রাখা হয়েছে, পরে এটা ডাইনামিক করা হবে।

### 🧠 Selector Function কেন আলাদা করে লেখা হলো?

`selectTasks` আর `selectFilter` — এই ফাংশনগুলোকে বলে **Selector Function**। এগুলো না বানিয়ে সরাসরি component এ গিয়েও লেখা যেত:

```ts
const { tasks } = useAppSelector((state) => state.todo.tasks);
```

কিন্তু selector আলাদা করে বানানোর সুবিধা হলো — component এ শুধু লিখতে হয়:

```ts
const tasks = useAppSelector(selectTasks);
```

এটাকে বলে **modularity** — state থেকে ডেটা বের করার লজিকটা component এর বাইরে, নিজের জায়গায় গোছানো থাকে, এবং দরকার হলে যেকোনো component থেকে reuse করা যায়। `task` আর `filter` একই slice এ থাকলেও এদের জন্য selector আলাদা রাখা হয়েছে, কারণ কাজ করার সময় সবসময় এই দুটোকে একসাথে দরকার নাও হতে পারে।

### 🎯 Selector Function এর সুবিধাগুলো (একটু বিস্তারিত)

1. **State এর গঠন (shape) থেকে Component আলাদা থাকে (Decoupling):**
   Component কে জানতেই হয় না যে `tasks` আসলে store এর ভেতর কোথায়, কীভাবে সাজানো আছে (`state.todo.tasks`)। Component শুধু জানে — `selectTasks` কল করলেই tasks পাওয়া যাবে। ফলে ভবিষ্যতে কখনো state এর গঠন পাল্টাতে হলে (যেমন `todo` কী পাল্টে অন্য নাম দেওয়া, বা ডেটা আলাদাভাবে সাজানো), শুধু selector ফাংশনের ভেতরটা বদলালেই হয় — প্রতিটা component এ গিয়ে `state.todo.tasks` খুঁজে খুঁজে বদলানো লাগে না।

2. **কোড পুনরায় ব্যবহার (Reusability):**
   একই selector একাধিক component থেকে ব্যবহার করা যায় — একই লজিক বারবার না লিখে।

3. **জটিল/derived ডেটা বের করার জায়গা একটাই থাকে:**
   এই নোটেই পরে দেখা যাবে, filter অনুযায়ী task বাছাই করার লজিকটাও (`selectTasks` এর ভেতরে `if/filter`) সরাসরি এই selector এর মধ্যেই রাখা হয়েছে, component এর মধ্যে না। ফলে component পরিষ্কার থাকে, আর "কীভাবে সঠিক task গুলো বের করা হয়" — এই জটিল হিসেবটা একটামাত্র জায়গায় (single source of truth) কেন্দ্রীভূত থাকে।

4. **টেস্ট করা সহজ হয় (Testability):**
   Selector একটা সাধারণ ফাংশন (`state` নিয়ে ডেটা রিটার্ন করে), তাই কোনো component render না করেই আলাদাভাবে টেস্ট করা যায় — শুধু একটা mock state পাস করে দিলেই হয়।

5. **পারফরম্যান্স অপ্টিমাইজেশনের সুযোগ থাকে:**
   ভবিষ্যতে চাইলে `reselect` এর মতো লাইব্রেরি দিয়ে এই selector গুলোকে **memoize** করা যায় — অর্থাৎ, একই ইনপুট (state) দিয়ে বারবার কল হলে আগের হিসাব করা রেজাল্টই আবার ব্যবহার করে ফেলে, নতুন করে হিসাব করে না। এতে বড় অ্যাপ্লিকেশনে অপ্রয়োজনীয় re-render কমে পারফরম্যান্স ভালো থাকে। এই সুবিধাটা কিন্তু শুধু তখনই পাওয়া যায়, যখন selector আলাদা ফাংশন হিসেবে লেখা থাকে — সরাসরি component এর ভেতরে ইনলাইন করে লিখলে এটা করা যায় না।

> ⚠️ **একটা ব্যাপার খেয়াল করার মতো:** slice এর `name` দেওয়া হয়েছে `"task"`, কিন্তু নিচে `store.ts` এ store এর ভেতরে এই reducer কে রাখা হয়েছে `todo` নামে key দিয়ে (`todo: taskReducer`)। এই কারণেই selector এর ভেতরে `state.todo.tasks` লিখতে হচ্ছে, `state.task.tasks` না। slice এর `name` মূলত internal action type (যেমন `task/addTask`) তৈরিতে ব্যবহৃত হয় — store এর key আলাদাভাবে ঠিক করা হয়। কনফিউশন এড়াতে সাধারণত slice এর নাম আর store এর key একই রাখাই ভালো অভ্যাস।

---

## ২. Store এ Task Reducer যোগ করা

`src/redux/store.ts`:

```ts
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./features/counter/counterSlice";
import taskReducer from "./features/task/taskSlice";

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    todo: taskReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

এই কাজটুকু হয়ে গেলে ব্রাউজারের Redux DevTools এ গিয়ে দেখলে `todo` state এর ভেতরে sample task টা দেখা যাবে।

---

## ৩. Task Card (Static UI প্রথমে)

`components/module/tasks/TaskCard.tsx` এ শুরুতে একটা static (হার্ডকোডেড) কার্ড বানানো হলো, যাতে UI কেমন দেখাবে সেটা আগে ঠিক করা যায়:

```tsx
import { Button } from "@/components/ui/button";
import { Checkbox } from "@/components/ui/checkbox";
import { Trash2 } from "lucide-react";

export default function TaskCard() {
  return (
    <div className="border px-5 py-3 rounded-md">
      <div className="flex justify-between items-center">
        <div className="flex gap-2 items-center">
          <div className="size-3 rounded-full bg-green-500"></div>
          <h1>Task Title</h1>
        </div>

        <div className="flex gap-3 items-center">
          <Button variant="link" className="p-0 text-red-500">
            <Trash2 />
          </Button>
          <Checkbox />
        </div>
      </div>

      <p className="mt-5">Task Description</p>
    </div>
  );
}
```

এই static কার্ডটাকে Task page এ `<TaskCard />` হিসেবে বসিয়ে UI যাচাই করে নেওয়া হয়েছে। পরের ধাপে এটাকে ডাইনামিক ডেটা দেখানোর উপযোগী করা হবে।

---

## ৪. Task যোগ করা (Add Task)

### ID Generate করা

নতুন task এর জন্য ইউনিক id লাগবে। এর জন্য দুইটা উপায় দেখানো হয়েছিল:

- **`uuid` প্যাকেজ:** [npmjs.com/package/uuid](https://www.npmjs.com/package/uuid)
- **`nanoid`:** এটা `@reduxjs/toolkit` এর সাথেই built-in আসে, তাই আলাদা করে ইন্সটল করা লাগে না — তাই এটাই ব্যবহার করা হয়েছে।

### `taskSlice.ts` আপডেট করা

```ts
import type { RootState } from "@/redux/store";
import { createSlice, nanoid, type PayloadAction } from "@reduxjs/toolkit";

export interface ITask {
  id: string;
  title: string;
  description: string;
  dueDate: string;
  isCompleted: boolean;
  priority: "high" | "medium" | "low";
}

interface IInitialState {
  tasks: ITask[];
  filter: "all" | "high" | "medium" | "low";
}

const initialState: IInitialState = {
  tasks: [
    {
      id: "rte3HSCAwfZFOTnfHcKFl",
      isCompleted: false,
      title: "Quibusdam dolor ut a",
      description: "Veniam enim consequ",
      dueDate: "1978-04-21",
      priority: "medium",
    },
  ],
  filter: "all",
};

type DraftTask = Pick<ITask, "title" | "description" | "dueDate" | "priority">;

const createTask = (taskData: DraftTask): ITask => {
  return { id: nanoid(), isCompleted: false, ...taskData };
};

const taskSlice = createSlice({
  name: "task",
  initialState,
  reducers: {
    addTask: (state, action: PayloadAction<DraftTask>) => {
      const taskData = createTask(action.payload);
      state.tasks.push(taskData);
    },
  },
});

export const selectTasks = (state: RootState) => {
  return state.todo.tasks;
};
export const selectFilter = (state: RootState) => {
  return state.todo.filter;
};

export const { addTask } = taskSlice.actions;
export default taskSlice.reducer;
```

> ✏️ **ছোট্ট একটা ভুল ঠিক করা হলো:** আগের ভার্সনে sample task এ `priority: "Medium"` (বড় হাতের M দিয়ে) লেখা ছিল, কিন্তু `ITask` টাইপে `priority` এর সম্ভাব্য মান শুধু `"high" | "medium" | "low"` (ছোট হাতের অক্ষরে)। এতে TypeScript error দেখাবে। তাই এখানে ছোট হাতের `"medium"` করে ঠিক করে দেওয়া হয়েছে।

### 🧩 `createTask` হেল্পার ফাংশন কেন?

Task তৈরির লজিকটা (id বসানো, `isCompleted: false` সেট করা) সরাসরি reducer এর ভেতরে লেখাও যেত:

```ts
addTask: (state, action: PayloadAction<ITask>) => {
  const id = nanoid();
  state.tasks.push({ ...action.payload, id, isCompleted: false });
},
```

কিন্তু `createTask` নামে আলাদা একটা ফাংশনে এই কাজটুকু বের করে নিলে reducer টা ছোট, পরিষ্কার আর পড়তে সহজ থাকে — এটা যেন শুধু "task যোগ করো" এই কাজটাতেই ফোকাস করে, "task কীভাবে বানানো হয়" সেই ডিটেইলে না জড়ায়। এটাও অনেকটা আগের নোটে দেখা **function এর দায়িত্ব ভাগ করে দেওয়ার** ধারণার মতোই।

---

## ৫. Add Task Modal (Form)

`components/module/task/AddTaskModal.tsx`:

```tsx
import { Button } from "@/components/ui/button";
import {
  Dialog,
  DialogClose,
  DialogContent,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Field, FieldError, FieldGroup, FieldLabel } from "@/components/ui/field";
import { Input } from "@/components/ui/input";
import {
  InputGroup,
  InputGroupAddon,
  InputGroupText,
  InputGroupTextarea,
} from "@/components/ui/input-group";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { addTask, type ITask } from "@/redux/features/task/taskSlice";
import { useAppDispatch } from "@/redux/hook";
import { Controller, useForm, type SubmitHandler } from "react-hook-form";

export function AddTaskModal() {
  const form = useForm<ITask>({
    defaultValues: {
      title: "",
      description: "",
      dueDate: "",
      priority: "medium",
    },
  });

  const dispatch = useAppDispatch();
  const onSubmit: SubmitHandler<ITask> = (data) => {
    dispatch(addTask(data));
  };

  return (
    <Dialog>
      <DialogTrigger render={<Button>Add Task</Button>} />
      <DialogContent className="sm:max-w-md">
        <form onSubmit={form.handleSubmit(onSubmit)}>
          <DialogHeader>
            <DialogTitle>Add Task</DialogTitle>
          </DialogHeader>

          <FieldGroup className="py-4">
            <Controller
              name="title"
              control={form.control}
              rules={{ required: "Title is required" }}
              render={({ field, fieldState }) => (
                <Field data-invalid={fieldState.invalid}>
                  <FieldLabel htmlFor="task-title">Title</FieldLabel>
                  <Input
                    {...field}
                    id="task-title"
                    aria-invalid={fieldState.invalid}
                    placeholder="Initialize frontend"
                    autoComplete="off"
                  />
                  {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
                </Field>
              )}
            />

            <Controller
              name="description"
              control={form.control}
              rules={{ maxLength: { value: 100, message: "Max 100 characters" } }}
              render={({ field, fieldState }) => (
                <Field data-invalid={fieldState.invalid}>
                  <FieldLabel htmlFor="task-description">Description</FieldLabel>
                  <InputGroup>
                    <InputGroupTextarea
                      {...field}
                      id="task-description"
                      placeholder="Create home page and routing"
                      rows={4}
                      className="min-h-20 resize-none"
                      aria-invalid={fieldState.invalid}
                    />
                    <InputGroupAddon align="block-end">
                      <InputGroupText className="tabular-nums">
                        {field.value.length}/100 characters
                      </InputGroupText>
                    </InputGroupAddon>
                  </InputGroup>
                  {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
                </Field>
              )}
            />

            <Controller
              name="dueDate"
              control={form.control}
              rules={{ required: "Due date is required" }}
              render={({ field, fieldState }) => (
                <Field data-invalid={fieldState.invalid}>
                  <FieldLabel htmlFor="task-dueDate">Due Date</FieldLabel>
                  <Input {...field} id="task-dueDate" type="date" aria-invalid={fieldState.invalid} />
                  {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
                </Field>
              )}
            />

            <Controller
              name="priority"
              control={form.control}
              render={({ field }) => (
                <Field>
                  <FieldLabel htmlFor="task-priority">Priority</FieldLabel>
                  <Select value={field.value} onValueChange={field.onChange}>
                    <SelectTrigger id="task-priority" className="w-full">
                      <SelectValue />
                    </SelectTrigger>
                    <SelectContent>
                      <SelectItem value="high">High</SelectItem>
                      <SelectItem value="medium">Medium</SelectItem>
                      <SelectItem value="low">Low</SelectItem>
                    </SelectContent>
                  </Select>
                </Field>
              )}
            />
          </FieldGroup>

          <DialogFooter>
            <DialogClose
              render={
                <Button type="button" variant="outline">
                  Cancel
                </Button>
              }
            />
            <Button type="submit">Save Task</Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

> ✏️ **যা ঠিক করা হয়েছে:**
> 1. `defaultValues` এ আগে `priority: "Medium"` ছিল, টাইপের সাথে মিলিয়ে `"medium"` করা হয়েছে (উপরের কারণেই)।
> 2. `onSubmit` এর টাইপ আগে `SubmitHandler<FieldValues>` লেখা ছিল, যা অনেকটা "generic/any" এর মতো — কোনো নির্দিষ্ট শেইপ চেক করে না। যেহেতু `useForm<ITask>` দিয়ে form টা আগে থেকেই `ITask` টাইপ করা আছে, তাই `onSubmit` কেও `SubmitHandler<ITask>` করে দেওয়া হয়েছে এবং `data as ITask` এর মতো জোর করে টাইপ-কাস্ট করার দরকার নেই — TypeScript নিজে থেকেই সঠিক টাইপ বুঝে নেবে।

---

## ৬. Task Card ডাইনামিক করা + `cn` ফাংশন

`components/module/task/TaskCard.tsx`:

```tsx
import { Button } from "@/components/ui/button";
import { Checkbox } from "@/components/ui/checkbox";
import type { ITask } from "@/redux/features/task/taskSlice";
import { cn } from "@/lib/utils";
import { Trash2 } from "lucide-react";

interface IProps {
  task: ITask;
}

export default function TaskCard({ task }: IProps) {
  return (
    <div className="border px-5 py-3 rounded-md">
      <div className="flex justify-between items-center">
        <div className="flex gap-2 items-center">
          <div
            className={cn("size-3 rounded-full", {
              "bg-green-500": task.priority === "low",
              "bg-yellow-500": task.priority === "medium",
              "bg-red-500": task.priority === "high",
            })}
          ></div>
          <h1>{task.title}</h1>
        </div>

        <div className="flex gap-3 items-center">
          <Button variant="link" className="p-0 text-red-500">
            <Trash2 />
          </Button>
          <Checkbox checked={task.isCompleted} />
        </div>
      </div>

      <p className="mt-5">{task.description}</p>
    </div>
  );
}
```

> ✏️ **দুটো জিনিস ঠিক করা হয়েছে:**
> 1. `import { cn } from "cn";` — সরাসরি এভাবে না লিখে, `import { cn } from "@/lib/utils";` লেখা হয়েছে। কারণ যদিও `"cn"` নামে সত্যিই একটা npm প্যাকেজ আছে (নিচে ব্যাখ্যা করা হলো), তবু প্রজেক্টে সেটাকে সরাসরি প্রতিটা কম্পোনেন্টে import না করে, প্রথমে `src/lib/utils.ts` ফাইলে একবার re-export করে রাখা হয় — এটাই বেশি প্রচলিত ও ভালো অভ্যাস।
> 2. `<Checkbox />` এ `task.isCompleted` এর সাথে বাইন্ড করা ছিল না, তাই টাস্ক আসলে সম্পন্ন (complete) থাকলেও checkbox সবসময় খালি দেখাতো। তাই `checked={task.isCompleted}` যোগ করা হয়েছে।

### 🧠 `cn` ফাংশন আসলে কী করে?

`"cn"` নামে npm-এ সত্যিই একটা প্যাকেজ আছে, যেটা `clsx` আর `tailwind-merge` — এই দুটো লাইব্রেরির কাজ একসাথে করে দেয়। প্রজেক্টে সাধারণত এটাকে সরাসরি প্রতিটা কম্পোনেন্টে import না করে, একবার `src/lib/utils.ts` ফাইলে re-export করে রাখা হয়:

```ts
// src/lib/utils.ts
export { cn } from "cn";
```

এরপর বাকি সব কম্পোনেন্ট থেকে এটাকে `@/lib/utils` থেকেই import করা হয়:

```ts
import { cn } from "@/lib/utils";
```

**এভাবে একটা জায়গা থেকে re-export করে রাখার সুবিধা:**
- ভবিষ্যতে কখনো `cn` এর সোর্স প্যাকেজ পাল্টাতে হলে (যেমন অন্য কোনো ইউটিলিটি ব্যবহার করতে হলে), শুধু `utils.ts` ফাইলটাই বদলালেই হবে — পুরো প্রজেক্ট জুড়ে প্রতিটা `import` লাইন খুঁজে খুঁজে বদলাতে হবে না।
- বাকি সব ইউটিলিটি ফাংশনের (যদি ভবিষ্যতে আরও যোগ হয়) সাথে `cn` ও একই জায়গা (`@/lib/utils`) থেকে পাওয়া যায় — খুঁজে বের করা সহজ হয়।

**`cn` আসলে কী কাজ করে (concept হিসেবে):**
- **`clsx` অংশ:** শর্তসাপেক্ষে (conditionally) ক্লাসনেম জোড়া দিতে সাহায্য করে। যেমন উপরের উদাহরণে, `task.priority === "low"` সত্যি হলে তখনই `"bg-green-500"` ক্লাসটা যোগ হবে, নাহলে হবে না।
- **`tailwind-merge` অংশ:** Tailwind এর ক্ষেত্রে একই ধরনের একাধিক ক্লাস (যেমন `bg-green-500` আর `bg-red-500`) একসাথে দিলে কনফ্লিক্ট হতে পারে — কোনটা আগে প্রাধান্য পাবে বোঝা যায় না। এটা বুদ্ধি করে বুঝে নেয় কোন ক্লাসটা আসলে থাকা উচিত, আর বাকিগুলো বাদ দিয়ে দেয়।

সংক্ষেপে: **`cn`** ব্যবহার করলে শর্তসাপেক্ষে ক্লাস বসানো যায়, আর Tailwind ক্লাসগুলোর মধ্যে কনফ্লিক্টও এড়ানো যায় — উপরের কোডে ঠিক এই কাজেই এটা ব্যবহার করা হয়েছে (priority অনুযায়ী রঙ বদলানোর জন্য)।

---

## ৭. Task Page এ সব একসাথে দেখানো

```tsx
import { AddTaskModal } from "@/components/module/tasks/AddTaskModal";
import TaskCard from "@/components/module/tasks/TaskCard";
import { selectTasks } from "@/redux/features/task/taskSlice";
import { useAppSelector } from "@/redux/hook";

const Task = () => {
  const tasks = useAppSelector(selectTasks);
  return (
    <div className="mx-auto max-w-7xl px-5 mt-20">
      <div className="flex justify-between items-center">
        <h1 className="text-2xl">Tasks</h1>
        <AddTaskModal />
      </div>

      <div className="space-y-5 mt-5">
        {tasks.map((task) => (
          <TaskCard key={task.id} task={task} />
        ))}
      </div>
    </div>
  );
};

export default Task;
```

---

## ৮. Task Complete টগল করা

`TaskCard.tsx` এ checkbox এ ক্লিক করলে dispatch করতে হবে:

```tsx
<Checkbox checked={task.isCompleted} onCheckedChange={() => dispatch(toggleCompleteState(task.id))} />
```

> ✏️ **একটা ছোট সংশোধন:** আগে `onClick` ব্যবহার করা হয়েছিল, কিন্তু `Checkbox` কম্পোনেন্টে সাধারণত `onClick` এর বদলে `onCheckedChange` ব্যবহার করাই বেশি নির্ভরযোগ্য (এটা checkbox এর checked/unchecked state পরিবর্তনের জন্যই বানানো হয়েছে)।

`taskSlice.ts` এর `reducers` এ:

```ts
toggleCompleteState: (state, action: PayloadAction<string>) => {
  const task = state.tasks.find((task) => task.id === action.payload);
  if (task) {
    task.isCompleted = !task.isCompleted;
  }
},
```

> ✏️ **একটু পরিষ্কার করা হয়েছে:** আগের ভার্সনে `forEach` এর ভেতরে একটা ternary (`task.id === action.payload ? (...) : task`) দিয়ে লেখা ছিল, যেটা প্রতিটা task এর জন্য লুপ চালায় এবং ম্যাচ না হলে `task` রিটার্ন করে যেটা আসলে কোনো কাজেই লাগে না (আউটপুট কোথাও ব্যবহার হচ্ছে না)। কোডটা কাজ করলেও পড়তে একটু জটিল। তার বদলে `find()` দিয়ে সরাসরি নির্দিষ্ট task টা খুঁজে বের করে তার `isCompleted` পাল্টে দেওয়াটা বেশি পরিষ্কার ও সহজবোধ্য।

অবশ্যই `toggleCompleteState` কে `taskSlice.actions` থেকে export করে দিতে হবে।

---

## ৯. Task ডিলিট করা

`TaskCard.tsx`:

```tsx
<Button variant="link" className="p-0 text-red-500" onClick={() => dispatch(deleteTask(task.id))}>
  <Trash2 />
</Button>
```

`taskSlice.ts`:

```ts
deleteTask: (state, action: PayloadAction<string>) => {
  state.tasks = state.tasks.filter((task) => task.id !== action.payload);
},
```

এটাকেও export করে দিতে হবে।

---

## ১০. Priority অনুযায়ী Filter করা

`taskSlice.ts` এ নতুন reducer:

```ts
updateFilter: (state, action: PayloadAction<"all" | "low" | "medium" | "high">) => {
  state.filter = action.payload;
},
```

এবং `selectTasks` selector টাকে filter অনুযায়ী কাজ করার মতো করে আপডেট করা হলো:

```ts
export const selectTasks = (state: RootState) => {
  const filter = state.todo.filter;

  if (filter === "all") {
    return state.todo.tasks;
  }

  return state.todo.tasks.filter((task) => task.priority === filter);
};
```

> ✏️ **একটু ছোট করে দেওয়া হয়েছে:** আগে `if / else if / else` দিয়ে প্রতিটা priority এর জন্য আলাদা আলাদা করে লেখা ছিল, যেখানে প্রতিটা শাখাতেই মূলত একই কাজ হচ্ছিল — শুধু `filter` এর মান পাল্টাচ্ছিল। যেহেতু `filter` এর মান আর `task.priority` এর মান একই রকম (`"high" | "medium" | "low"`), তাই `task.priority === filter` দিয়ে একটা লাইনেই কাজটা সেরে ফেলা যায় — কোড ছোট ও রক্ষণাবেক্ষণ (maintain) করা সহজ হয়।

`Task.tsx` এ Tabs দিয়ে filter সিলেক্ট করার UI:

```tsx
import { AddTaskModal } from "@/components/module/tasks/AddTaskModal";
import TaskCard from "@/components/module/tasks/TaskCard";
import { Tabs, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { selectTasks, updateFilter } from "@/redux/features/task/taskSlice";
import { useAppDispatch, useAppSelector } from "@/redux/hook";

const Task = () => {
  const tasks = useAppSelector(selectTasks);
  const dispatch = useAppDispatch();

  return (
    <div className="mx-auto max-w-7xl px-5 mt-20">
      <div className="flex justify-between items-center">
        <h1 className="text-2xl">Tasks</h1>
        <Tabs defaultValue="all" className="w-100">
          <TabsList>
            <TabsTrigger onClick={() => dispatch(updateFilter("all"))} value="all">
              All
            </TabsTrigger>
            <TabsTrigger onClick={() => dispatch(updateFilter("low"))} value="low">
              Low
            </TabsTrigger>
            <TabsTrigger onClick={() => dispatch(updateFilter("medium"))} value="medium">
              Medium
            </TabsTrigger>
            <TabsTrigger onClick={() => dispatch(updateFilter("high"))} value="high">
              High
            </TabsTrigger>
          </TabsList>
        </Tabs>
        <AddTaskModal />
      </div>

      <div className="space-y-5 mt-5">
        {tasks.map((task) => (
          <TaskCard key={task.id} task={task} />
        ))}
      </div>
    </div>
  );
};

export default Task;
```

---

## ১১. নিজে চেষ্টা করার জন্য (Practice)

- **Edit / Update Task:** বিদ্যমান একটা task এর তথ্য পরিবর্তনের ফিচার — `updateTask` নামে একটা reducer বানিয়ে `PayloadAction<{ id: string; data: DraftTask }>` টাইপ দিয়ে চেষ্টা করা যেতে পারে।
- **User যোগ করা:** ঠিক task এর মতোই একইভাবে একটা `user` slice বানিয়ে, প্রতিটা task এর সাথে একজন user (assignee) যুক্ত করার ফিচার (যেমন `ITask` এ `assignedTo: string` যোগ করে) — নিজে করে দেখার জন্য রাখা হলো।

---
Full source code: [redux-todos](https://github.com/arifucoder/redux-todos)
--
## সংক্ষেপে — পুরো ফ্লো

1. **Slice বানানো** → state আর initial data ঠিক করা।
2. **Store এ যুক্ত করা** → DevTools দিয়ে যাচাই করা।
3. **UI আগে static ভাবে বানানো**, পরে dynamic করা।
4. **Add** → `nanoid()` দিয়ে id বানিয়ে নতুন task push করা।
5. **Toggle Complete** → নির্দিষ্ট task খুঁজে `isCompleted` পাল্টানো।
6. **Delete** → `filter()` দিয়ে বাদ দেওয়া।
7. **Filter** → selector এর ভেতরেই filter এর লজিক রাখা, যাতে component পরিষ্কার থাকে।
