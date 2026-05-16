# Insightclaw MVP 实现方案

> **给执行者的说明：** 实现本计划时，需要按任务逐步执行。每个步骤使用复选框（`- [ ]`）追踪进度。

**目标：** 构建 Insightclaw MVP，包含五个页面：首页、安装教程、模型推荐、动态 OpenRouter 模型排行、关于页。

**架构：** 使用 Next.js App Router、TypeScript 和 Tailwind CSS。编辑性内容放在类型化数据文件中，大部分页面静态渲染，模型排行通过服务端数据模块拉取 OpenRouter 数据，并提供本地备用数据。

**技术栈：** Next.js、React、TypeScript、Tailwind CSS、Vitest、Testing Library、Playwright、OpenRouter API。

---

## 文件结构

创建或修改以下文件：

```text
package.json
next.config.ts
tsconfig.json
postcss.config.mjs
tailwind.config.ts
vitest.config.ts
playwright.config.ts
app/layout.tsx
app/page.tsx
app/install/page.tsx
app/llms/page.tsx
app/models/page.tsx
app/about/page.tsx
app/globals.css
components/SiteHeader.tsx
components/SiteFooter.tsx
components/PageHero.tsx
components/CardLink.tsx
components/InstallGuide.tsx
components/RecommendationGrid.tsx
components/ModelsExplorer.tsx
data/featuredLinks.ts
data/installGuides.ts
data/llmRecommendations.ts
data/fallbackModels.ts
lib/models.ts
lib/format.ts
tests/models.test.ts
tests/format.test.ts
tests/e2e/navigation.spec.ts
```

职责划分：

- `app/*`：页面路由组合和页面 metadata。
- `components/*`：可复用 UI 模块。
- `data/*`：MVP 的结构化内容数据。
- `lib/models.ts`：OpenRouter 拉取、数据标准化、备用数据逻辑。
- `lib/format.ts`：价格和数字格式化工具。
- `tests/*`：单元测试和路由冒烟测试。

## 任务 1：初始化 Next.js 项目

**文件：**
- 创建：`package.json`
- 创建：`next.config.ts`
- 创建：`tsconfig.json`
- 创建：`postcss.config.mjs`
- 创建：`tailwind.config.ts`
- 创建：`app/globals.css`
- 创建：`app/layout.tsx`

- [ ] **步骤 1：初始化 package 信息**

运行：

```powershell
npm init -y
```

预期：生成 `package.json`。

- [ ] **步骤 2：安装依赖**

运行：

```powershell
npm install next react react-dom
npm install -D typescript @types/node @types/react @types/react-dom tailwindcss postcss autoprefixer vitest @testing-library/react @testing-library/jest-dom jsdom playwright
```

预期：依赖安装完成，并生成 `package-lock.json`。

- [ ] **步骤 3：替换 `package.json` scripts**

使用下面的 scripts：

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "test": "vitest run",
    "test:watch": "vitest",
    "e2e": "playwright test",
    "lint": "next lint"
  }
}
```

- [ ] **步骤 4：创建框架配置**

创建 `next.config.ts`：

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  experimental: {},
};

export default nextConfig;
```

创建 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

创建 `postcss.config.mjs`：

```js
const config = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

export default config;
```

创建 `tailwind.config.ts`：

```ts
import type { Config } from "tailwindcss";

const config: Config = {
  content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: {
        ink: "#172033",
        muted: "#667085",
        line: "#e6eaf2",
        paper: "#f8fafc",
        brand: "#2563eb",
        accent: "#14b8a6",
      },
      boxShadow: {
        soft: "0 18px 50px rgba(23, 32, 51, 0.08)",
      },
    },
  },
  plugins: [],
};

export default config;
```

- [ ] **步骤 5：创建全局样式和根布局**

创建 `app/globals.css`：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  color-scheme: light;
}

html {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  background: #f8fafc;
  color: #172033;
}

::selection {
  background: rgba(37, 99, 235, 0.16);
}
```

创建 `app/layout.tsx`：

```tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: {
    default: "Insightclaw",
    template: "%s | Insightclaw",
  },
  description: "AI Agent 工具与模型精选指南。",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN">
      <body>{children}</body>
    </html>
  );
}
```

- [ ] **步骤 6：验证应用能启动构建**

运行：

```powershell
npm run build
```

预期：构建成功；如果只是因为 `app/page.tsx` 还不存在而失败，继续执行任务 2。

## 任务 2：公共布局组件

**文件：**
- 创建：`components/SiteHeader.tsx`
- 创建：`components/SiteFooter.tsx`
- 创建：`components/PageHero.tsx`
- 创建：`components/CardLink.tsx`
- 修改：`app/layout.tsx`

- [ ] **步骤 1：创建顶部导航**

创建 `components/SiteHeader.tsx`：

```tsx
import Link from "next/link";

const navItems = [
  { href: "/", label: "首页" },
  { href: "/install", label: "安装" },
  { href: "/llms", label: "模型推荐" },
  { href: "/models", label: "模型排行" },
  { href: "/about", label: "关于" },
];

export function SiteHeader() {
  return (
    <header className="sticky top-0 z-40 border-b border-line bg-white/90 backdrop-blur">
      <div className="mx-auto flex max-w-6xl items-center justify-between px-5 py-4">
        <Link href="/" className="text-lg font-semibold tracking-tight text-ink">
          Insight<span className="text-brand">claw</span>
        </Link>
        <nav className="hidden items-center gap-5 text-sm text-muted md:flex">
          {navItems.map((item) => (
            <Link key={item.href} href={item.href} className="hover:text-brand">
              {item.label}
            </Link>
          ))}
        </nav>
        <Link
          href="/install"
          className="rounded-full bg-ink px-4 py-2 text-sm font-medium text-white hover:bg-brand"
        >
          开始安装
        </Link>
      </div>
    </header>
  );
}
```

- [ ] **步骤 2：创建页脚**

创建 `components/SiteFooter.tsx`：

```tsx
export function SiteFooter() {
  return (
    <footer className="border-t border-line bg-white">
      <div className="mx-auto flex max-w-6xl flex-col gap-3 px-5 py-8 text-sm text-muted md:flex-row md:items-center md:justify-between">
        <p>Insightclaw · 个人精选的 AI Agent 工具指南</p>
        <p>推荐基于易用性、稳定性、性价比和真实工作流价值。</p>
      </div>
    </footer>
  );
}
```

- [ ] **步骤 3：创建页面 Hero 组件**

创建 `components/PageHero.tsx`：

```tsx
type PageHeroProps = {
  eyebrow?: string;
  title: string;
  description: string;
};

export function PageHero({ eyebrow, title, description }: PageHeroProps) {
  return (
    <section className="mx-auto max-w-6xl px-5 pb-10 pt-14">
      {eyebrow ? (
        <p className="mb-3 text-sm font-medium uppercase tracking-wide text-brand">{eyebrow}</p>
      ) : null}
      <h1 className="max-w-3xl text-4xl font-semibold tracking-tight text-ink md:text-5xl">
        {title}
      </h1>
      <p className="mt-5 max-w-2xl text-lg leading-8 text-muted">{description}</p>
    </section>
  );
}
```

- [ ] **步骤 4：创建入口卡片组件**

创建 `components/CardLink.tsx`：

```tsx
import Link from "next/link";

type CardLinkProps = {
  href: string;
  title: string;
  description: string;
  label: string;
};

export function CardLink({ href, title, description, label }: CardLinkProps) {
  return (
    <Link
      href={href}
      className="rounded-2xl border border-line bg-white p-6 shadow-soft transition hover:-translate-y-0.5 hover:border-brand"
    >
      <h3 className="text-lg font-semibold text-ink">{title}</h3>
      <p className="mt-3 min-h-20 text-sm leading-6 text-muted">{description}</p>
      <span className="mt-5 inline-flex text-sm font-medium text-brand">{label}</span>
    </Link>
  );
}
```

- [ ] **步骤 5：把导航和页脚接入根布局**

修改 `app/layout.tsx`：

```tsx
import type { Metadata } from "next";
import { SiteFooter } from "@/components/SiteFooter";
import { SiteHeader } from "@/components/SiteHeader";
import "./globals.css";

export const metadata: Metadata = {
  title: {
    default: "Insightclaw",
    template: "%s | Insightclaw",
  },
  description: "AI Agent 工具与模型精选指南。",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN">
      <body>
        <SiteHeader />
        <main>{children}</main>
        <SiteFooter />
      </body>
    </html>
  );
}
```

## 任务 3：结构化内容数据

**文件：**
- 创建：`data/featuredLinks.ts`
- 创建：`data/installGuides.ts`
- 创建：`data/llmRecommendations.ts`
- 创建：`data/fallbackModels.ts`

- [ ] **步骤 1：创建首页精选入口数据**

创建 `data/featuredLinks.ts`：

```ts
export const featuredLinks = [
  {
    href: "/install",
    title: "安装 OpenClaw 与 Claude Code",
    description: "用两条路线帮你搭好 Agent 工具环境：一个面向 Skills 生态，一个面向本地代码工作流。",
    label: "查看安装路线",
  },
  {
    href: "/llms",
    title: "按场景选择模型",
    description: "从编程、中文问答、长上下文、低成本和强推理几个角度快速选模型。",
    label: "查看推荐",
  },
  {
    href: "/models",
    title: "查看实时模型排行",
    description: "接入 OpenRouter 数据，按热度、价格和上下文长度筛选可用模型。",
    label: "打开榜单",
  },
] as const;
```

- [ ] **步骤 2：创建安装教程数据**

创建 `data/installGuides.ts`：

```ts
export type InstallStep = {
  title: string;
  body: string;
  command?: string;
};

export type InstallGuide = {
  id: "openclaw" | "claude-code";
  name: string;
  summary: string;
  bestFor: string;
  steps: InstallStep[];
  issues: string[];
};

export const installGuides: InstallGuide[] = [
  {
    id: "openclaw",
    name: "OpenClaw",
    summary: "适合想体验 Skills、MCP 和 Agent 工具生态的用户。",
    bestFor: "工具探索、Web3/数据类 Skills、可扩展 Agent 工作流",
    steps: [
      {
        title: "准备 Node.js",
        body: "优先安装 Node.js 22 LTS。安装后重新打开终端，确认 node 和 npm 可以使用。",
        command: "node -v && npm -v",
      },
      {
        title: "运行安装脚本",
        body: "根据官方安装脚本完成 OpenClaw 初始化。Windows 用户使用 PowerShell，macOS/Linux 用户使用 shell。",
        command: "irm https://openclaw.ai/install.ps1 | iex",
      },
      {
        title: "首次启动",
        body: "启动后按提示完成模型、密钥和工作目录配置。",
        command: "openclaw",
      },
    ],
    issues: ["命令不可用时，先确认 Node.js 已加入 PATH。", "安装脚本失败时，换用稳定网络后重新执行。"],
  },
  {
    id: "claude-code",
    name: "Claude Code",
    summary: "适合在本地项目里让 AI 读代码、改代码、运行任务的用户。",
    bestFor: "代码理解、功能实现、重构、测试修复",
    steps: [
      {
        title: "安装 Claude Code",
        body: "按 Anthropic 官方文档完成安装，并确认命令行工具可用。",
        command: "claude --version",
      },
      {
        title: "完成登录或鉴权",
        body: "打开终端中的登录流程，按提示授权账号或配置 API Key。",
        command: "claude login",
      },
      {
        title: "进入项目目录",
        body: "在你要处理的代码项目根目录启动 Claude Code，让它读取项目上下文。",
        command: "cd your-project && claude",
      },
    ],
    issues: ["鉴权失败时，检查账号权限和网络。", "项目过大时，先让模型阅读关键目录和 README。"],
  },
];
```

- [ ] **步骤 3：创建模型推荐数据**

创建 `data/llmRecommendations.ts`：

```ts
export type Recommendation = {
  title: string;
  models: string[];
  audience: string;
  why: string;
  caveat: string;
  scenarios: string[];
};

export const useCaseRecommendations: Recommendation[] = [
  {
    title: "编程与代码修改",
    models: ["Claude Sonnet", "GPT-5.4", "Gemini Pro"],
    audience: "需要读项目、写代码、修测试的开发者。",
    why: "这类模型在长链路推理、代码理解和指令跟随上更稳。",
    caveat: "强模型成本更高，日常小改可以切到更便宜的模型。",
    scenarios: ["功能实现", "代码审查", "测试修复"],
  },
  {
    title: "中文问答与内容整理",
    models: ["Qwen", "DeepSeek", "GPT-5.4 Mini"],
    audience: "需要中文解释、总结和资料整理的用户。",
    why: "中文表达自然，成本通常更容易控制。",
    caveat: "涉及最新事实时仍要配合搜索或来源核验。",
    scenarios: ["中文教程", "资料总结", "问答助手"],
  },
  {
    title: "低成本日常使用",
    models: ["Gemini Flash", "GPT-5.4 Mini", "DeepSeek Chat"],
    audience: "高频使用、预算敏感的新手和个人用户。",
    why: "响应快、价格低，适合大量轻任务。",
    caveat: "复杂任务需要升级到更强模型。",
    scenarios: ["快速问答", "草稿生成", "简单脚本"],
  },
];

export const toolRecommendations: Recommendation[] = [
  {
    title: "OpenClaw 推荐组合",
    models: ["Claude Sonnet", "Qwen", "Gemini Flash"],
    audience: "想用 Agent Skills 跑工具、查数据、串工作流的用户。",
    why: "强模型负责规划和执行，轻模型负责便宜的日常调用。",
    caveat: "涉及交易或安全判断时，保留人工确认。",
    scenarios: ["Skills 调用", "MCP 工具链", "数据分析"],
  },
  {
    title: "Claude Code 推荐组合",
    models: ["Claude Sonnet", "Claude Opus", "GPT-5.4"],
    audience: "把 Agent 用在真实代码库里的开发者。",
    why: "代码上下文理解和多步修改能力更关键。",
    caveat: "大型重构前要先让模型生成计划，并分批执行。",
    scenarios: ["项目理解", "功能开发", "重构计划"],
  },
];
```

- [ ] **步骤 4：创建备用模型数据**

创建 `data/fallbackModels.ts`：

```ts
export type ModelInfo = {
  id: string;
  name: string;
  provider: string;
  contextLength: number;
  promptPrice: number;
  completionPrice: number;
  isFree: boolean;
  tags: string[];
};

export const fallbackModels: ModelInfo[] = [
  {
    id: "anthropic/claude-sonnet",
    name: "Claude Sonnet",
    provider: "Anthropic",
    contextLength: 200000,
    promptPrice: 3,
    completionPrice: 15,
    isFree: false,
    tags: ["coding", "reasoning"],
  },
  {
    id: "openai/gpt-5.4",
    name: "GPT-5.4",
    provider: "OpenAI",
    contextLength: 128000,
    promptPrice: 5,
    completionPrice: 15,
    isFree: false,
    tags: ["general", "coding"],
  },
  {
    id: "google/gemini-flash",
    name: "Gemini Flash",
    provider: "Google",
    contextLength: 1000000,
    promptPrice: 0.3,
    completionPrice: 2.5,
    isFree: false,
    tags: ["fast", "long-context"],
  },
];
```

## 任务 4：格式化工具和模型数据逻辑

**文件：**
- 创建：`lib/format.ts`
- 创建：`lib/models.ts`
- 创建：`tests/format.test.ts`
- 创建：`tests/models.test.ts`
- 创建：`vitest.config.ts`

- [ ] **步骤 1：创建 Vitest 配置**

创建 `vitest.config.ts`：

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "jsdom",
    globals: true,
  },
});
```

- [ ] **步骤 2：编写格式化测试**

创建 `tests/format.test.ts`：

```ts
import { describe, expect, it } from "vitest";
import { formatContext, formatUsdPerMillion } from "@/lib/format";

describe("format helpers", () => {
  it("formats token context", () => {
    expect(formatContext(128000)).toBe("128K");
    expect(formatContext(1000000)).toBe("1M");
  });

  it("formats prices per million tokens", () => {
    expect(formatUsdPerMillion(0)).toBe("免费");
    expect(formatUsdPerMillion(3)).toBe("$3.00/M");
  });
});
```

- [ ] **步骤 3：实现格式化工具**

创建 `lib/format.ts`：

```ts
export function formatContext(tokens: number) {
  if (tokens >= 1_000_000) return `${Math.round(tokens / 1_000_000)}M`;
  if (tokens >= 1_000) return `${Math.round(tokens / 1_000)}K`;
  return `${tokens}`;
}

export function formatUsdPerMillion(price: number) {
  if (!price) return "免费";
  return `$${price.toFixed(2)}/M`;
}
```

- [ ] **步骤 4：编写模型标准化测试**

创建 `tests/models.test.ts`：

```ts
import { describe, expect, it } from "vitest";
import { normalizeOpenRouterModel } from "@/lib/models";

describe("normalizeOpenRouterModel", () => {
  it("normalizes OpenRouter model data", () => {
    const model = normalizeOpenRouterModel({
      id: "provider/example",
      name: "Example Model",
      context_length: 128000,
      pricing: { prompt: "0.000003", completion: "0.000015" },
      architecture: { input_modalities: ["text", "image"] },
    });

    expect(model).toEqual({
      id: "provider/example",
      name: "Example Model",
      provider: "provider",
      contextLength: 128000,
      promptPrice: 3,
      completionPrice: 15,
      isFree: false,
      tags: ["image"],
    });
  });
});
```

- [ ] **步骤 5：实现 OpenRouter 模型逻辑**

创建 `lib/models.ts`：

```ts
import { fallbackModels, type ModelInfo } from "@/data/fallbackModels";

type OpenRouterModel = {
  id: string;
  name?: string;
  context_length?: number;
  pricing?: {
    prompt?: string;
    completion?: string;
  };
  architecture?: {
    input_modalities?: string[];
  };
};

export function normalizeOpenRouterModel(model: OpenRouterModel): ModelInfo {
  const promptPrice = Number(model.pricing?.prompt ?? 0) * 1_000_000;
  const completionPrice = Number(model.pricing?.completion ?? 0) * 1_000_000;
  const modalities = model.architecture?.input_modalities ?? [];

  return {
    id: model.id,
    name: model.name ?? model.id,
    provider: model.id.split("/")[0] ?? "unknown",
    contextLength: model.context_length ?? 0,
    promptPrice,
    completionPrice,
    isFree: promptPrice === 0 && completionPrice === 0,
    tags: modalities.includes("image") ? ["image"] : [],
  };
}

export async function getModels(): Promise<{ models: ModelInfo[]; source: "live" | "fallback" }> {
  try {
    const response = await fetch("https://openrouter.ai/api/v1/models", {
      next: { revalidate: 3600 },
      headers: {
        "User-Agent": "Insightclaw MVP",
      },
    });

    if (!response.ok) {
      return { models: fallbackModels, source: "fallback" };
    }

    const payload = (await response.json()) as { data?: OpenRouterModel[] };
    const models = (payload.data ?? []).map(normalizeOpenRouterModel);

    return {
      models: models.length > 0 ? models : fallbackModels,
      source: models.length > 0 ? "live" : "fallback",
    };
  } catch {
    return { models: fallbackModels, source: "fallback" };
  }
}
```

- [ ] **步骤 6：运行单元测试**

运行：

```powershell
npm test
```

预期：两个测试文件全部通过。

## 任务 5：构建静态页面

**文件：**
- 创建：`app/page.tsx`
- 创建：`app/install/page.tsx`
- 创建：`app/llms/page.tsx`
- 创建：`app/about/page.tsx`
- 创建：`components/InstallGuide.tsx`
- 创建：`components/RecommendationGrid.tsx`

- [ ] **步骤 1：创建首页**

创建 `app/page.tsx`：

```tsx
import { CardLink } from "@/components/CardLink";
import { PageHero } from "@/components/PageHero";
import { featuredLinks } from "@/data/featuredLinks";

const paths = [
  { title: "新手路线", body: "先看安装页，完成 OpenClaw 或 Claude Code 的第一轮启动。" },
  { title: "开发者路线", body: "先安装 Claude Code，再用模型推荐页选择适合代码任务的模型。" },
  { title: "进阶路线", body: "从模型排行开始，对比价格、上下文长度和能力标签。" },
];

export default function HomePage() {
  return (
    <>
      <PageHero
        eyebrow="AI Agent Guide"
        title="用一条清晰路线开始你的 Agent 工具箱。"
        description="Insightclaw 精选 OpenClaw、Claude Code 和常用模型，帮你少踩坑、更快进入真实工作流。"
      />
      <section className="mx-auto grid max-w-6xl gap-5 px-5 pb-12 md:grid-cols-3">
        {featuredLinks.map((link) => (
          <CardLink key={link.href} {...link} />
        ))}
      </section>
      <section className="mx-auto max-w-6xl px-5 pb-20">
        <h2 className="text-2xl font-semibold text-ink">推荐路径</h2>
        <div className="mt-5 grid gap-4 md:grid-cols-3">
          {paths.map((path) => (
            <div key={path.title} className="rounded-2xl border border-line bg-white p-5">
              <h3 className="font-semibold text-ink">{path.title}</h3>
              <p className="mt-2 text-sm leading-6 text-muted">{path.body}</p>
            </div>
          ))}
        </div>
      </section>
    </>
  );
}
```

- [ ] **步骤 2：创建安装教程组件**

创建 `components/InstallGuide.tsx`：

```tsx
import type { InstallGuide as InstallGuideType } from "@/data/installGuides";

export function InstallGuide({ guide }: { guide: InstallGuideType }) {
  return (
    <article className="rounded-2xl border border-line bg-white p-6 shadow-soft">
      <div>
        <h2 className="text-2xl font-semibold text-ink">{guide.name}</h2>
        <p className="mt-2 text-muted">{guide.summary}</p>
        <p className="mt-3 text-sm font-medium text-brand">适合：{guide.bestFor}</p>
      </div>
      <ol className="mt-6 space-y-4">
        {guide.steps.map((step, index) => (
          <li key={step.title} className="rounded-xl bg-paper p-4">
            <p className="text-sm font-semibold text-ink">
              {index + 1}. {step.title}
            </p>
            <p className="mt-2 text-sm leading-6 text-muted">{step.body}</p>
            {step.command ? (
              <pre className="mt-3 overflow-x-auto rounded-lg bg-ink p-3 text-sm text-white">
                <code>{step.command}</code>
              </pre>
            ) : null}
          </li>
        ))}
      </ol>
      <div className="mt-6 border-t border-line pt-5">
        <h3 className="text-sm font-semibold text-ink">常见问题</h3>
        <ul className="mt-3 space-y-2 text-sm text-muted">
          {guide.issues.map((issue) => (
            <li key={issue}>· {issue}</li>
          ))}
        </ul>
      </div>
    </article>
  );
}
```

- [ ] **步骤 3：创建安装页**

创建 `app/install/page.tsx`：

```tsx
import type { Metadata } from "next";
import { InstallGuide } from "@/components/InstallGuide";
import { PageHero } from "@/components/PageHero";
import { installGuides } from "@/data/installGuides";

export const metadata: Metadata = {
  title: "安装教程",
};

export default function InstallPage() {
  return (
    <>
      <PageHero
        eyebrow="Install"
        title="安装 OpenClaw 和 Claude Code。"
        description="两条路线分别面向 Agent Skills 生态和本地代码工作流。按步骤完成后，你就能开始真实使用。"
      />
      <section className="mx-auto grid max-w-6xl gap-6 px-5 pb-20 lg:grid-cols-2">
        {installGuides.map((guide) => (
          <InstallGuide key={guide.id} guide={guide} />
        ))}
      </section>
    </>
  );
}
```

- [ ] **步骤 4：创建推荐卡片网格组件**

创建 `components/RecommendationGrid.tsx`：

```tsx
import type { Recommendation } from "@/data/llmRecommendations";

export function RecommendationGrid({ items }: { items: Recommendation[] }) {
  return (
    <div className="grid gap-5 md:grid-cols-2 lg:grid-cols-3">
      {items.map((item) => (
        <article key={item.title} className="rounded-2xl border border-line bg-white p-5 shadow-soft">
          <h3 className="text-lg font-semibold text-ink">{item.title}</h3>
          <p className="mt-3 text-sm text-brand">{item.models.join(" / ")}</p>
          <p className="mt-4 text-sm leading-6 text-muted">{item.audience}</p>
          <p className="mt-3 text-sm leading-6 text-muted">{item.why}</p>
          <p className="mt-3 text-sm leading-6 text-muted">注意：{item.caveat}</p>
          <div className="mt-4 flex flex-wrap gap-2">
            {item.scenarios.map((scenario) => (
              <span key={scenario} className="rounded-full bg-paper px-3 py-1 text-xs text-muted">
                {scenario}
              </span>
            ))}
          </div>
        </article>
      ))}
    </div>
  );
}
```

- [ ] **步骤 5：创建模型推荐页**

创建 `app/llms/page.tsx`：

```tsx
import type { Metadata } from "next";
import { PageHero } from "@/components/PageHero";
import { RecommendationGrid } from "@/components/RecommendationGrid";
import { toolRecommendations, useCaseRecommendations } from "@/data/llmRecommendations";

export const metadata: Metadata = {
  title: "模型推荐",
};

export default function LlmsPage() {
  return (
    <>
      <PageHero
        eyebrow="LLM Picks"
        title="按任务和工具选择模型。"
        description="先按用途判断你需要什么能力，再看 OpenClaw 和 Claude Code 中更适合的组合。"
      />
      <section className="mx-auto max-w-6xl px-5 pb-12">
        <h2 className="mb-5 text-2xl font-semibold text-ink">按用途推荐</h2>
        <RecommendationGrid items={useCaseRecommendations} />
      </section>
      <section className="mx-auto max-w-6xl px-5 pb-20">
        <h2 className="mb-5 text-2xl font-semibold text-ink">按工具推荐</h2>
        <RecommendationGrid items={toolRecommendations} />
      </section>
    </>
  );
}
```

- [ ] **步骤 6：创建关于页**

创建 `app/about/page.tsx`：

```tsx
import type { Metadata } from "next";
import { PageHero } from "@/components/PageHero";

export const metadata: Metadata = {
  title: "关于",
};

export default function AboutPage() {
  return (
    <>
      <PageHero
        eyebrow="About"
        title="Insightclaw 是个人精选，不是广告目录。"
        description="这里关注的是工具能不能帮你完成真实工作，而不是把所有热门名词都堆在一起。"
      />
      <section className="mx-auto max-w-3xl px-5 pb-20 text-base leading-8 text-muted">
        <p>
          Insightclaw 会优先推荐容易上手、运行稳定、价格合理，并且适合真实 Agent 工作流的工具和模型。
        </p>
        <p className="mt-5">
          第一版聚焦 OpenClaw、Claude Code 和模型选择。后续可以扩展 Agent 工具导航、教程文章和可复用配置模板。
        </p>
      </section>
    </>
  );
}
```

## 任务 6：动态模型排行页

**文件：**
- 创建：`components/ModelsExplorer.tsx`
- 创建：`app/models/page.tsx`

- [ ] **步骤 1：创建客户端模型浏览组件**

创建 `components/ModelsExplorer.tsx`：

```tsx
"use client";

import { useMemo, useState } from "react";
import type { ModelInfo } from "@/data/fallbackModels";
import { formatContext, formatUsdPerMillion } from "@/lib/format";

type SortKey = "rank" | "price" | "context";
type FilterKey = "all" | "free" | "paid";

export function ModelsExplorer({ models, source }: { models: ModelInfo[]; source: "live" | "fallback" }) {
  const [query, setQuery] = useState("");
  const [filter, setFilter] = useState<FilterKey>("all");
  const [sort, setSort] = useState<SortKey>("rank");

  const visibleModels = useMemo(() => {
    return models
      .filter((model) => {
        const matchesQuery =
          model.name.toLowerCase().includes(query.toLowerCase()) ||
          model.id.toLowerCase().includes(query.toLowerCase());
        const matchesFilter =
          filter === "all" || (filter === "free" && model.isFree) || (filter === "paid" && !model.isFree);
        return matchesQuery && matchesFilter;
      })
      .sort((a, b) => {
        if (sort === "price") return a.promptPrice + a.completionPrice - (b.promptPrice + b.completionPrice);
        if (sort === "context") return b.contextLength - a.contextLength;
        return 0;
      });
  }, [filter, models, query, sort]);

  return (
    <section className="mx-auto max-w-6xl px-5 pb-20">
      {source === "fallback" ? (
        <div className="mb-5 rounded-xl border border-amber-200 bg-amber-50 px-4 py-3 text-sm text-amber-900">
          实时排行暂时不可用，当前展示备用模型数据。
        </div>
      ) : null}
      <div className="mb-5 grid gap-3 md:grid-cols-[1fr_auto_auto]">
        <input
          value={query}
          onChange={(event) => setQuery(event.target.value)}
          placeholder="搜索模型名称或 ID"
          className="rounded-xl border border-line bg-white px-4 py-3 text-sm outline-none focus:border-brand"
        />
        <select
          value={filter}
          onChange={(event) => setFilter(event.target.value as FilterKey)}
          className="rounded-xl border border-line bg-white px-4 py-3 text-sm"
        >
          <option value="all">全部</option>
          <option value="free">免费</option>
          <option value="paid">付费</option>
        </select>
        <select
          value={sort}
          onChange={(event) => setSort(event.target.value as SortKey)}
          className="rounded-xl border border-line bg-white px-4 py-3 text-sm"
        >
          <option value="rank">默认排行</option>
          <option value="price">价格优先</option>
          <option value="context">上下文优先</option>
        </select>
      </div>
      <div className="overflow-hidden rounded-2xl border border-line bg-white shadow-soft">
        <table className="w-full min-w-[760px] text-left text-sm">
          <thead className="bg-paper text-muted">
            <tr>
              <th className="px-4 py-3">模型</th>
              <th className="px-4 py-3">供应商</th>
              <th className="px-4 py-3">上下文</th>
              <th className="px-4 py-3">输入</th>
              <th className="px-4 py-3">输出</th>
              <th className="px-4 py-3">标签</th>
            </tr>
          </thead>
          <tbody>
            {visibleModels.map((model) => (
              <tr key={model.id} className="border-t border-line">
                <td className="px-4 py-4">
                  <p className="font-medium text-ink">{model.name}</p>
                  <p className="mt-1 font-mono text-xs text-muted">{model.id}</p>
                </td>
                <td className="px-4 py-4 text-muted">{model.provider}</td>
                <td className="px-4 py-4 text-muted">{formatContext(model.contextLength)}</td>
                <td className="px-4 py-4 text-muted">{formatUsdPerMillion(model.promptPrice)}</td>
                <td className="px-4 py-4 text-muted">{formatUsdPerMillion(model.completionPrice)}</td>
                <td className="px-4 py-4 text-muted">{model.tags.length ? model.tags.join(", ") : "text"}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </section>
  );
}
```

- [ ] **步骤 2：创建模型排行页**

创建 `app/models/page.tsx`：

```tsx
import type { Metadata } from "next";
import { ModelsExplorer } from "@/components/ModelsExplorer";
import { PageHero } from "@/components/PageHero";
import { getModels } from "@/lib/models";

export const metadata: Metadata = {
  title: "模型排行",
};

export default async function ModelsPage() {
  const { models, source } = await getModels();

  return (
    <>
      <PageHero
        eyebrow="Model Ranking"
        title="查看可用大模型排行。"
        description="基于 OpenRouter 数据展示模型，并支持按名称、价格、免费状态和上下文长度筛选。"
      />
      <ModelsExplorer models={models} source={source} />
    </>
  );
}
```

## 任务 7：端到端检查

**文件：**
- 创建：`playwright.config.ts`
- 创建：`tests/e2e/navigation.spec.ts`

- [ ] **步骤 1：创建 Playwright 配置**

创建 `playwright.config.ts`：

```ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  webServer: {
    command: "npm run dev",
    url: "http://127.0.0.1:3000",
    reuseExistingServer: true,
  },
  use: {
    baseURL: "http://127.0.0.1:3000",
    trace: "on-first-retry",
  },
  projects: [
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "mobile", use: { ...devices["Pixel 5"] } },
  ],
});
```

- [ ] **步骤 2：创建路由冒烟测试**

创建 `tests/e2e/navigation.spec.ts`：

```ts
import { expect, test } from "@playwright/test";

const routes = [
  { path: "/", text: "Insightclaw" },
  { path: "/install", text: "OpenClaw" },
  { path: "/llms", text: "按用途推荐" },
  { path: "/models", text: "搜索模型名称或 ID" },
  { path: "/about", text: "个人精选" },
];

for (const route of routes) {
  test(`renders ${route.path}`, async ({ page }) => {
    await page.goto(route.path);
    await expect(page.getByText(route.text).first()).toBeVisible();
  });
}
```

- [ ] **步骤 3：安装 Playwright 浏览器**

运行：

```powershell
npx playwright install chromium
```

预期：本地端到端测试所需的 Chromium 安装完成。

- [ ] **步骤 4：运行全部检查**

运行：

```powershell
npm test
npm run build
npm run e2e
```

预期：单元测试通过，生产构建成功，桌面端和移动端路由冒烟测试通过。

## 任务 8：本地运行和最终打磨

**文件：**
- 按需修改：`app/*`、`components/*`、`data/*`、`app/globals.css`

- [ ] **步骤 1：启动开发服务器**

运行：

```powershell
npm run dev
```

预期：应用可通过 `http://localhost:3000` 访问。

- [ ] **步骤 2：视觉检查**

打开以下页面：

```text
http://localhost:3000/
http://localhost:3000/install
http://localhost:3000/llms
http://localhost:3000/models
http://localhost:3000/about
```

预期：

- 桌面端和移动端文字都清晰可读。
- 卡片内容不溢出。
- 表格在小屏幕上可以横向滚动。
- 导航链接能到达对应页面。
- 视觉风格保持干净知识库风。

- [ ] **步骤 3：如果当前目录已初始化 Git，则提交**

如果当前文件夹里有 `.git` 目录，运行：

```powershell
git add .
git commit -m "feat: build insightclaw mvp"
```

预期：生成一个包含 MVP 实现的提交。

## 自检

规格覆盖：

- 五个 MVP 路由由任务 5 和任务 6 覆盖。
- 结构化数据文件由任务 3 覆盖。
- OpenRouter 实时数据和备用数据逻辑由任务 4 和任务 6 覆盖。
- 干净知识库视觉方向由任务 2、任务 5、任务 6 和任务 8 覆盖。
- 错误处理由 `getModels()` 的备用数据逻辑和页面状态提示覆盖。
- 验证流程由单元测试、构建、Playwright 路由检查和人工视觉检查覆盖。

占位检查：

- 计划中没有 TBD 标记。
- 每个步骤都包含明确文件、命令、预期结果和代码片段。

类型一致性：

- `ModelInfo`、`InstallGuide`、`Recommendation` 类型在数据文件中定义，并被组件复用。
- `getModels()` 返回的 `source` 值与 `ModelsExplorer` 消费的值一致。
- `ModelsExplorer` 使用的格式化工具已经定义并有测试覆盖。

