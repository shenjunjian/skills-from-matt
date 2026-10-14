# 什么时候该 mock

只在**系统边界**上 mock：

- 外部 API（支付、邮件等）
- 数据库（视情况 - 优先用测试数据库）
- 时间/随机数
- 文件系统（视情况）

不要 mock：

- 你自己的类/模块
- 内部协作者
- 任何由你控制的东西

## 为可 mock 性做设计

在系统边界上，把接口设计成容易 mock 的样子：

**1. 使用依赖注入**

把外部依赖传进来，而不是在内部创建它们：

```typescript
// Easy to mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**2. 优先用 SDK 风格的接口，而不是通用的 fetcher**

为每一个外部操作写一个专门的函数，而不是一个带条件逻辑的通用函数：

```typescript
// GOOD: Each function is independently mockable
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD: Mocking requires conditional logic inside the mock
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

SDK 这种做法带来的好处：
- 每个 mock 只返回一种特定的形状
- 测试准备代码里没有条件逻辑
- 更容易看出一个测试打到了哪些端点
- 每个端点各自有类型安全
