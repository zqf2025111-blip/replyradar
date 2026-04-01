# ReplyRadar — Product Requirements Document (PRD)

**Version**: 1.0  
**Date**: 2026-04-01  
**Status**: Active Development

---

## 1. 产品概述

### 核心定位
ReplyRadar 是一个"AI社交购买意图监控"工具。它帮助 SaaS 创始人、独立开发者和自由职业者，实时监控 Reddit / Hacker News / Twitter(X) 上用户发出的购买信号，并通过 AI 生成个性化、非垃圾式的回复建议，让用户可以在竞争对手之前抢先接触潜在客户。

### 核心差异化（vs 竞品）
| 竞品 | 缺陷 | ReplyRadar 的答案 |
|------|------|------------------|
| F5Bot | 关键词匹配，噪音极大，无AI | 语义级意图识别，只推送真线索 |
| ReplyGuy | 全自动发帖，封号风险极高 | Human-in-the-loop，AI草稿+人工发布 |
| Trigify | $69-$549/月，太贵 | $19-$79/月，专注个人创始人 |
| Brand24/Mention | 企业级复杂，无购买意图识别 | 轻量极简，专注"找客户"单一场景 |
| Gummysearch | 已倒闭（2025年底） | **直接填补市场真空** |

### 商业模式
- BYOK（用户自带 OpenAI API Key）→ 零边际成本，利润率~90%
- 订阅制：$19 / $39 / $79 per month
- 目标：500个付费用户 = $10k-20k MRR

---

## 2. 目标用户

### Primary（主要）
- **SaaS 独立创始人 / Indie Hackers**：产品已上线，需要冷启动获客
- **自由职业者**：设计师、开发者、文案，需要找项目
- **小型 B2B SaaS 团队**：1-5人，无专职销售

### Secondary（次要）
- 数字营销代理公司
- 增长黑客 / Growth Hacker

### 用户画像（主要）
- 英语用户，活跃在 Reddit / HN / Twitter
- 有技术背景，理解 BYOK 的意义
- 月收入 < $5k，每个工具的预算敏感

---

## 3. 功能规格

### P0 — MVP（第1-2周）

#### 3.1 关键词管理
- 用户可以添加多个监控关键词（如 "looking for email tool", "alternative to Mailchimp"）
- 支持设置监控平台（Reddit / HN）
- 支持开启/关闭单个关键词

#### 3.2 Reddit 监控引擎
- 使用 Reddit 官方 API（snoowrap JS SDK）
- 每小时自动扫描一次
- 扫描范围：帖子标题 + 内容
- 去重：通过 external_id 防止重复推送

#### 3.3 HN 监控引擎
- 使用 Algolia HN Search API（完全免费）
- 每小时扫描 Ask HN / Show HN

#### 3.4 AI 意图过滤（核心）
- 调用用户自己的 OpenAI API（BYOK）
- 使用 gpt-4o-mini（成本极低：$0.15/1M tokens）
- 分类标签：
  - `seeking_recommendation`：正在找工具推荐 ✅
  - `frustrated_with_competitor`：对竞品不满 ✅
  - `asking_for_help`：技术问题求助 ✅
  - `general_discussion`：普通讨论 ❌（过滤）
  - `news_or_spam`：新闻/垃圾 ❌（过滤）
- 意图强度评分（0-100）

#### 3.5 AI 回复生成
- 对通过意图过滤的帖子，生成 3 条不同风格的回复
  - 风格1：友好建议（不带产品链接）
  - 风格2：专家解答（结尾提及产品）
  - 风格3：简短有力（适合快速互动）
- 用户可以编辑生成的回复后手动发布
- 一键跳转到原帖

#### 3.6 BYOK 设置
- 用户在设置页面输入 OpenAI API Key
- 前端不明文显示（掩码处理）
- 后端 AES-256 加密存储

#### 3.7 仪表盘（Dashboard）
- 今日新发现线索数
- 本周发现 vs 上周对比
- 最新待处理的意图列表
- 快速过滤：全部 / 未处理 / 已回复 / 已忽略

### P1 — 提升留存（第3-4周）

#### 3.8 邮件提醒
- 发现高分意图时（>70分）发送邮件
- 使用 Resend（免费 3000封/月）

#### 3.9 产品知识库（Knowledge Base）
- 用户填写自己的产品描述、核心功能、对比竞品
- AI 生成回复时会引用这些信息，使回复更个性化

#### 3.10 状态追踪
- 标记意图为：待处理 / 已回复 / 已忽略 / 已转化
- 简单统计：本月回复次数、转化次数

### P2 — 未来规划

- Twitter/X 监控（需 RapidAPI 付费额度）
- 多平台：LinkedIn、IndieHackers、Quora
- Chrome 插件：在浏览器侧边栏显示回复建议
- 团队协作：多用户共享一个工作区

---

## 4. 技术架构

### 技术栈
```
前端:      Next.js 14 (App Router) + Tailwind CSS + shadcn/ui
后端:      Next.js Route Handlers + Vercel Edge Functions
数据库:    Supabase (PostgreSQL + Auth)
任务调度:  Vercel Cron Jobs (每小时触发扫描)
Reddit:    snoowrap (官方 Reddit API，免费)
HN:        Algolia HN Search API (完全免费)
AI:        OpenAI API (用户自带 Key，gpt-4o-mini)
邮件:      Resend (免费 3000封/月)
支付:      Stripe Checkout
部署:      Vercel (Hobby Plan，免费)
```

### 系统流程
```
Vercel Cron (每小时)
  → 读取所有活跃关键词
  → 并行查询 Reddit API + Algolia HN API
  → 去重过滤（检查 mentions 表）
  → AI 意图分类（gpt-4o-mini，用用户自己的Key）
  → 保存到 mentions 表（含意图分数）
  → 高分意图触发邮件提醒（Resend）
  
用户操作
  → 登录 Dashboard，查看意图列表
  → 点击"生成回复" → 调用 OpenAI 生成3条建议
  → 用户编辑后，点击"去Reddit回复"（跳转原帖）
  → 标记状态
```

### 数据库 Schema

```sql
-- 用户配置
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users,
  openai_api_key_encrypted TEXT,
  product_name TEXT,
  product_description TEXT,
  product_url TEXT,
  subscription_tier TEXT DEFAULT 'free', -- free, starter, pro, agency
  stripe_customer_id TEXT,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 监控关键词
CREATE TABLE keywords (
  id SERIAL PRIMARY KEY,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  phrase TEXT NOT NULL,
  platforms TEXT[] DEFAULT '{reddit,hn}',
  is_active BOOLEAN DEFAULT true,
  last_scanned_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 发现的意图帖子
CREATE TABLE mentions (
  id SERIAL PRIMARY KEY,
  keyword_id INTEGER REFERENCES keywords(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  external_id TEXT NOT NULL,
  platform TEXT NOT NULL, -- reddit, hn, twitter
  title TEXT,
  content TEXT,
  url TEXT NOT NULL,
  author TEXT,
  subreddit TEXT,
  intent_label TEXT, -- seeking_recommendation, frustrated_with_competitor, etc.
  intent_score INTEGER DEFAULT 0, -- 0-100
  status TEXT DEFAULT 'pending', -- pending, replied, ignored, converted
  posted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(external_id, platform)
);

-- AI 生成的回复
CREATE TABLE ai_replies (
  id SERIAL PRIMARY KEY,
  mention_id INTEGER REFERENCES mentions(id) ON DELETE CASCADE,
  style TEXT, -- friendly, expert, concise
  suggested_text TEXT NOT NULL,
  edited_text TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Row Level Security
ALTER TABLE keywords ENABLE ROW LEVEL SECURITY;
ALTER TABLE mentions ENABLE ROW LEVEL SECURITY;
ALTER TABLE ai_replies ENABLE ROW LEVEL SECURITY;
```

---

## 5. 定价方案

| 套餐 | 价格 | 关键词数 | 扫描频率 | 平台 | 邮件提醒 |
|------|------|---------|---------|------|---------|
| **Free** | $0 | 2个 | 每24小时 | Reddit | 否 |
| **Starter** | $19/月 | 5个 | 每小时 | Reddit + HN | 是 |
| **Pro** | $39/月 | 20个 | 每30分钟 | Reddit + HN + Twitter | 是 |
| **Agency** | $79/月 | 100个 | 实时 | 全部 + 优先支持 | 是 |

**Free 套餐策略**：提供免费版让用户体验"真的有效"，再转化付费。

---

## 6. 获客策略（冷启动）

### 第1周：内容种草
- 在 Reddit r/SaaS、r/indiehackers、r/entrepreneur 发真诚的帖子
- 标题策略："I built a tool to find leads on Reddit without spamming - here's how"
- HN Show HN 帖

### 第2周：直接找用户
- 用产品本身监控关键词，找到正在抱怨类似痛点的人
- 发送个性化的 DM（非垃圾式）

### 持续：SEO
- 目标关键词："reddit marketing tool", "find leads on reddit", "social intent monitoring"
- 每周 AI 自动生成 2-3 篇 SEO 博客文章

---

## 7. 里程碑

| 时间 | 目标 |
|------|------|
| 第1-2周 | Landing Page 上线 + 收集 Email 等待列表 |
| 第3-4周 | MVP 上线（Reddit + HN 监控 + AI 回复） |
| 第5-6周 | 获得前 10 个付费用户 |
| 第2-3个月 | 50 付费用户 = ~$1.5k MRR |
| 第4-6个月 | 250 付费用户 = ~$7k MRR |
| 第6-9个月 | 400 付费用户 = ~$12k MRR |

---

*文档由 AI 代理自动生成，版本 1.0*
