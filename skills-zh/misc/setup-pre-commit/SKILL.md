---
name: setup-pre-commit
description: 在当前仓库配置带 lint-staged（Prettier）的 Husky pre-commit 钩子，包含类型检查和测试。当用户想添加 pre-commit 钩子、配置 Husky、设置 lint-staged，或增加提交时的格式化/类型检查/测试时使用。
---

# 配置 Pre-Commit 钩子

## 这会配置什么

- **Husky** pre-commit 钩子
- **lint-staged**，对所有暂存文件运行 Prettier
- **Prettier** 配置（如果缺失）
- pre-commit 钩子里的 **typecheck** 和 **test** 脚本

## 步骤

### 1. 识别包管理器

检查 `package-lock.json`（npm）、`pnpm-lock.yaml`（pnpm）、`yarn.lock`（yarn）、`bun.lockb`（bun）。用存在的那一个。判断不清就默认 npm。

### 2. 安装依赖

作为 devDependencies 安装：

```
husky lint-staged prettier
```

### 3. 初始化 Husky

```bash
npx husky init
```

这会创建 `.husky/` 目录，并在 package.json 里加入 `prepare: "husky"`。

### 4. 创建 `.husky/pre-commit`

写入这个文件（Husky v9+ 不需要 shebang）：

```
npx lint-staged
npm run typecheck
npm run test
```

**按需调整**：把 `npm` 换成识别出的包管理器。如果仓库的 package.json 里没有 `typecheck` 或 `test` 脚本，就删掉对应的那几行，并告知用户。

### 5. 创建 `.lintstagedrc`

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

### 6. 创建 `.prettierrc`（如果缺失）

只在没有 Prettier 配置时创建。使用这些默认值：

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": false,
  "trailingComma": "es5",
  "semi": true,
  "arrowParens": "always"
}
```

### 7. 验证

- [ ] `.husky/pre-commit` 存在且可执行
- [ ] `.lintstagedrc` 存在
- [ ] package.json 里的 `prepare` 脚本是 `"husky"`
- [ ] `prettier` 配置存在
- [ ] 运行 `npx lint-staged` 确认它可用

### 8. 提交

暂存所有改动和新建的文件，用这条信息提交：`Add pre-commit hooks (husky + lint-staged + prettier)`

这次提交会走一遍新的 pre-commit 钩子，正好是一次验证一切可用的冒烟测试。

## 备注

- Husky v9+ 的钩子文件不需要 shebang
- `prettier --ignore-unknown` 会跳过 Prettier 无法解析的文件（图片等）
- pre-commit 先跑 lint-staged（快，只处理暂存文件），再跑完整的类型检查和测试
