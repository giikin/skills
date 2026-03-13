# 初始化 Mock 地址

当用户要求“开启一个新的 mock 地址”“接入 Apifox mock 地址”“把某个模块切到本地 mock”时，优先按下面约定处理。

## 默认约定

1. 不修改项目根目录的 `.env`
2. 只在 `.env.development` 中新增或覆盖对应的 `VITE_BASE_*_API`
3. 在 `vite.config.ts` 的 `server.proxy` 中新增代理配置
4. 新代理必须写在 `...giimeDevProxyFn(env),` 的下面

## 推荐配置

本地 Apifox Mock 根地址默认使用：

```txt
http://127.0.0.1:4523/m1/7051948-6772053-default
```

## 修改步骤

### 1. 修改 `.env.development`

不要改 `.env`，直接在 `.env.development` 增加对应变量，例如：

```env
VITE_BASE_AUTH_AI_API=/auth-ai-mock
```

如果该变量已经存在，则直接覆盖为新的 mock 前缀。

### 2. 修改 `vite.config.ts`

在 `server.proxy` 中，确保新增代理写在 `...giimeDevProxyFn(env),` 后面：

```ts
const proxy = {
  ...giimeDevProxyFn(env),
  '/auth-ai-mock': {
    target: 'http://127.0.0.1:4523/m1/7051948-6772053-default',
    changeOrigin: true,
    rewrite: p => p.replace(/^\/auth-ai-mock/, ''),
    bypass,
  },
};
```

## 注意事项

1. 前缀命名应体现业务模块，例如 `authAi` 使用 `/auth-ai-mock`
2. `target` 填 mock 根地址，不要拼具体接口路径
3. `rewrite` 只去掉本地代理前缀，保留真实接口路径
4. 修改完 `.env.development` 或 `vite.config.ts` 后，需要重启本地 dev 服务
