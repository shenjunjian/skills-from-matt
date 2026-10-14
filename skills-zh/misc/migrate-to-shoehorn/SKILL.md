---
name: migrate-to-shoehorn
description: 把测试文件从 `as` 类型断言迁移到 @total-typescript/shoehorn。当用户提到 shoehorn、想替换测试里的 `as`，或需要部分测试数据时使用。
---

# 迁移到 Shoehorn

## 为什么用 shoehorn？

`shoehorn`（类型断言辅助库）让你在测试里传入部分数据，同时让 TypeScript 满意。它用类型安全的写法替代 `as` 断言。

**只用于测试代码。** 绝不在生产代码里使用 shoehorn。

`as` 在测试里的问题：

- 习惯了不用它
- 必须手工指定目标类型
- 对于故意写错的数据，要用双重 as（`as unknown as Type`）

## 安装

```bash
npm i @total-typescript/shoehorn
```

## 迁移模式

### 大对象、只需少量属性

改前：

```ts
type Request = {
  body: { id: string };
  headers: Record<string, string>;
  cookies: Record<string, string>;
  // ...20 more properties
};

it("gets user by id", () => {
  // Only care about body.id but must fake entire Request
  getUser({
    body: { id: "123" },
    headers: {},
    cookies: {},
    // ...fake all 20 properties
  });
});
```

改后：

```ts
import { fromPartial } from "@total-typescript/shoehorn";

it("gets user by id", () => {
  getUser(
    fromPartial({
      body: { id: "123" },
    }),
  );
});
```

### `as Type` → `fromPartial()`

改前：

```ts
getUser({ body: { id: "123" } } as Request);
```

改后：

```ts
import { fromPartial } from "@total-typescript/shoehorn";

getUser(fromPartial({ body: { id: "123" } }));
```

### `as unknown as Type` → `fromAny()`

改前：

```ts
getUser({ body: { id: 123 } } as unknown as Request); // wrong type on purpose
```

改后：

```ts
import { fromAny } from "@total-typescript/shoehorn";

getUser(fromAny({ body: { id: 123 } }));
```

## 各自什么时候用

| 函数            | 适用场景                                       |
| --------------- | ---------------------------------------------- |
| `fromPartial()` | 传入部分数据，仍能通过类型检查                 |
| `fromAny()`     | 传入故意写错的数据（保留自动补全）             |
| `fromExact()`   | 强制完整对象（之后可换回 fromPartial）         |

## 工作流

1. **收集需求** - 问用户：
   - 哪些测试文件里的 `as` 断言造成了问题？
   - 是不是在处理大对象，而其中只有部分属性有意义？
   - 是否需要为测试错误路径传入故意写错的数据？

2. **安装并迁移**：
   - [ ] 安装：`npm i @total-typescript/shoehorn`
   - [ ] 找出含 `as` 断言的测试文件：`grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
   - [ ] 用 `fromPartial()` 替换 `as Type`
   - [ ] 用 `fromAny()` 替换 `as unknown as Type`
   - [ ] 从 `@total-typescript/shoehorn` 添加 import
   - [ ] 运行类型检查验证
