## 严重问题 (必须修复)

**硬编码敏感配置与回退地址**  
当前代码在获取 `BACKEND_URL` 和 `BACKEND_API_TOKEN` 时，直接以**明文形式**将生产/内网地址以及内部令牌设置为默认值。

```java
// 第 52–54 行上下文
String backendUrl = getEnvOrDefault("BACKEND_URL", "http://dean.nat200.top");
String backendToken = getEnvOrDefault("BACKEND_API_TOKEN", "openai-code-review-internal-token-2026");
```

**风险说明**  
1. **令牌泄露**：`openai-code-review-internal-token-2026` 硬编码在源码中，一旦代码仓库被公开或内部越权访问，攻击者可直接使用该令牌调用后端服务，导致数据泄露或内部操作被冒用。  
2. **意外环境连通**：默认 URL `http://dean.nat200.top` 可能指向真实服务。如果环境变量未正确设置，本地开发或测试环境会直连该地址，轻则造成脏数据，重则影响线上服务。  
3. **配置漂移**：此后端地址和令牌属于部署态配置，不应与代码耦合。每次更换地址都需要改代码、重新构建，违背 12‑Factor App 配置分离原则。

### ✅ 建议修复方案

**移除默认值中的真实凭据与地址**，强制从环境变量读取；若缺失则在启动时提前失败或使用明确的开发占位符，并输出警告。

```java
// 强制要求环境变量，缺失时启动失败
String backendUrl = System.getenv("BACKEND_URL");
String backendToken = System.getenv("BACKEND_API_TOKEN");
if (backendUrl == null || backendUrl.isBlank()) {
    throw new IllegalStateException("BACKEND_URL 环境变量未设置");
}
if (backendToken == null || backendToken.isBlank()) {
    throw new IllegalStateException("BACKEND_API_TOKEN 环境变量未设置");
}
```

如果确实需要在本地开发阶段提供便利，可以使用一个**明显无效**的占位符，并打印醒目的提示：

```java
String backendUrl = getEnvOrDefault("BACKEND_URL", "http://localhost:8080"); // 仅本地开发占位
String backendToken = getEnvOrDefault("BACKEND_API_TOKEN", "dev-placeholder-token-change-me");
if ("dev-placeholder-token-change-me".equals(backendToken)) {
    log.warn("正在使用不安全的开发令牌，请在生产环境通过 BACKEND_API_TOKEN 环境变量设置真实令牌！");
}
```

> ⚠️ **切勿将任何形式的真实令牌保存到 Git 历史中**。如果该令牌曾经提交过，需要立即在服务端作废并轮换新令牌，同时清理 Git 历史。

---

### 🟢 亮点
- 使用 `getEnvOrDefault` 工具方法统一处理环境变量读取，体现了对配置外部化的初步意识，值得肯定。只需收紧默认值策略即可从“半安全”提升到“安全”。