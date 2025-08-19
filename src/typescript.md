## 📘 TypeScript Deep-Dive (Advanced Types, Config & Best Practices)

### Why TypeScript?

- Adds **static typing** on top of JavaScript → catches bugs at compile time.
- Supports ESNext features (decorators, metadata).
- Better tooling (autocomplete, refactoring, IntelliSense).

---

### Type System Concepts

| Concept            | What it means / syntax example                         |
| ------------------ | ------------------------------------------------------ |
| Union types        | `type ID = string                                      | number`  |
| Intersection types | `type A = B & C`                                       |
| Literal types      | `let dir: "left"                                       | "right"` |
| Tuple              | `[string, number]`                                     |
| Enum               | `enum Role { ADMIN, USER }`                            |
| Generics           | `function wrap<T>(arg: T): T {}`                       |
| Utility types      | `Partial<T>`, `Required<T>`, `Pick<T,K>`, `Omit<T,K>`  |
| Conditional types  | `T extends U ? X : Y`                                  |
| Mapped types       | `{ [P in keyof T]: string }`                           |
| keyof operator     | Gives property names as a union: `keyof SomeInterface` |

---

### `unknown` vs `any`

| Type      | Requires type-checking | Safe? | Use case                |
| --------- | ---------------------- | ----- | ----------------------- |
| `any`     | No                     | ❌     | Legacy JS, avoid        |
| `unknown` | Yes — must narrow      | ✅     | Preferable safe default |

Example:
```ts
let val: unknown;
if (typeof val === "string") {
  val.toUpperCase();
}
```

---

### Key `tsconfig.json` Flags

| Flag                     | Description                                   |
| ------------------------ | --------------------------------------------- |
| `strict`                 | Enables all strict type-checks                |
| `skipLibCheck`           | Skips typing for node_modules → faster builds |
| `esModuleInterop`        | Allows default import for CommonJS modules    |
| `baseUrl`, `paths`       | Enable absolute path imports (e.g. `@app/*`)  |
| `experimentalDecorators` | Required for decorators                       |
| `emitDecoratorMetadata`  | Emits design-time types (NestJS, Inversify)   |

---

### Decorators

- Runtime annotations over classes/methods/properties.
- Require `"experimentalDecorators": true`.

```ts
function Log(): MethodDecorator {
  return (_, __, descriptor) => {
    const orig = descriptor.value;
    descriptor.value = function (...args: any[]) {
      console.log("Called with", args);
      return orig.apply(this, args);
    };
  };
}
```

---

### Compile-time vs Runtime

- **Types exist only at compile-time.**
- No generics at runtime → erased.
- Use `reflect-metadata` if you want runtime type info.

---

### Best Practices

- Prefer `interface` for contracts, `type` for unions/intersections.
- Enable `"strict": true`
- Always define return types on public functions.
- Avoid `any` and overuse of `!` (non-null assertion).
- Use namespaces only if you are not using ES modules.

---

### Frequently Asked Interview Questions

**Q: What is the difference between `interface` and `type` in TypeScript?**  
A: Both can describe the shape of an object. However, `interface` is *extendable* and supports declaration merging — multiple `interface` declarations with the same name get merged, useful in extending third-party types. `type` is more flexible (can represent union, intersection, mapped, conditional types), but does not merge automatically. Use `interface` when defining object contracts and `type` for more complex type compositions.

---

**Q: Explain `unknown` vs `any`.**  
A: `any` disables type checking — you can perform any operation on an `any` value. `unknown` is a safer counterpart: you can assign anything to it, but must perform **type narrowing** before using the value, enforcing type safety. Rule of thumb: use `unknown` as default escape hatch, not `any`.

---

**Q: What is a mapped type? Give an example.**  
A: Mapped types iterate over the keys of another type to construct a new type.

```ts
type Readonly<T> = { [P in keyof T]: T[P] }
```

In this example, `Readonly<T>` creates a type where all fields are the same as `T`, but (usually) marked `readonly`.

---

**Q: How do conditional types work?**  
A: They allow types to be chosen based on a condition, similar to ternary operations:

```ts
type IsString<T> = T extends string ? "yes" : "no"
```

So `IsString<"a">` becomes `"yes"` while `IsString<number>` becomes `"no"` — enabling powerful type-level logic.

---

**Q: What is `keyof` and how is it used?**  
A: `keyof` takes a type and returns a union of its keys.

```ts
interface User { id: number; name: string; }
type K = keyof User // "id" | "name"
```

Often used with generics to enforce valid property names when accessing object keys.

---

**Q: Write a generic function in TypeScript.**  
A: Example:

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

This function takes a value of any type `T` and returns the same type. Generics allow for strong typing while keeping functions flexible.

---

