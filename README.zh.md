# google-maps-skill

[English](README.md) | 中文

AI 驱动的 Google Maps 对话技能 — 让出境旅行规划像聊天一样简单。

## 为什么做这个？

Google Maps 是全球使用最广泛的地图。虽然在少数国家（中国、朝鲜）受限于当地政策，但出境旅行离不开它。

**痛点**：出国玩的时候，Google Maps 显示的都是当地文字——韩文看起来像回形针撒在屏幕上，日文地名在 Google Maps 和中文社区（小红书、马蜂窝等）之间翻译不一致，经常对不上号。每次都要反复比对，浪费大量时间。

**解法**：把这个 skill 装进 Claude Code，就相当于在真实的 Google Maps 数据上加了一层 AI 对话。用中文直接问，AI 自动调用地图 API、翻译地名、对照差异，一步到位。

## 功能

| 命令 | 说明 |
|---|---|
| `geocode` | 地址 → 坐标（"东京塔在哪？"） |
| `reverse-geocode` | 坐标 → 地址（"这个坐标是什么地方？"） |
| `directions` | 路线规划，支持驾车/步行/骑行/公交 4 种模式 |
| `places-search` | 文字搜索地点（"新宿附近的拉面店"） |
| `places-nearby` | 在指定位置周边发现地点（"500米内的咖啡店"） |
| `place-detail` | 查询地点详情 |
| `elevation` | 查询任意位置的海拔 |
| `timezone` | 查询任意位置的时区 |

## 配置

### 前提条件

- [Bun](https://bun.sh/) 运行时
- [Google Cloud CLI](https://cloud.google.com/sdk/docs/install) (`gcloud`)

### 开通 GCP API（4 条命令搞定）

不用在 Google Cloud Console 网页上点来点去。用 `gcloud` CLI 直接搞定，还可以让 AI 帮你执行。

**第 1 步：登录并创建项目**

```bash
gcloud auth login
gcloud projects create my-maps-skill --name="my-maps-skill"
```

**第 2 步：关联计费账号**

```bash
# 查看你的 billing account ID
gcloud billing accounts list

# 关联
gcloud billing projects link my-maps-skill --billing-account=YOUR_ACCOUNT_ID
```

**第 3 步：一次性启用 5 个 API**

```bash
gcloud services enable \
  geocoding-backend.googleapis.com \
  routes.googleapis.com \
  places.googleapis.com \
  elevation-backend.googleapis.com \
  timezone-backend.googleapis.com \
  --project=my-maps-skill
```

**第 4 步：创建 API Key**

```bash
gcloud services api-keys create --display-name="maps-skill-key" --project=my-maps-skill
```

保存输出中的 `keyString`。

### 设置环境变量

```bash
export GOOGLE_MAPS_API_KEY="your_key_here"
```

加入 `~/.zshrc` 或 `~/.bashrc` 以持久化。

> **关于费用**：Google 每月提供 $200 免费额度，覆盖所有 Maps API。手动测试几十次完全免费，但需要绑定信用卡才能开通。建议在 Console 中设置用量上限防止意外扣费。

## 安装

### 方式一：Claude Code Marketplace（推荐）

```
/plugin marketplace add deusyu/google-maps-skill
/plugin install gmaps@google-maps-skill
```

### 方式二：npx

```bash
npx skills add deusyu/google-maps-skill
```

### 方式三：手动安装

```bash
git clone https://github.com/deusyu/google-maps-skill.git
cp -r google-maps-skill/skills/gmaps ~/.claude/skills/
```

## 使用

安装后，在 Claude Code 中直接用自然语言对话：

```
鹿儿岛核心区有哪些酒店？
```

```
从福冈到鹿儿岛怎么走，有几种方式？
```

```
Tokyo Tower 的经纬度是多少？
```

```
Find restaurants near Shibuya Station
```

```
曼谷是什么时区？
```

Claude 会自动读取地图数据、翻译地名、整理结果——不需要记任何命令。

## 设计模式

本项目采用**声明式命令定义 + 命令即模块**的架构：

- 每个命令是一个独立文件，导出一个 `CommandDef` 声明式配置对象
- 通用引擎读取配置自动执行，无需为每个命令写执行逻辑
- 通用校验器根据 flag 定义自动验证输入参数
- 支持两种认证模式（query param / header）和两种 HTTP 方法（GET / POST）

**新增一个命令只需：新建 1 个文件 + 注册表加 1 行。**

```
commands/
├── geocode.ts          # 每个文件就是一个命令定义
├── directions.ts       # POST + header auth + fieldMask
├── place-detail.ts     # 动态 URL (buildUrl)
├── ...
└── index.ts            # 注册表：Map<string, CommandDef>
```

对比此前的 [amap-skill](https://github.com/deusyu/rainman-skills)（高德地图）：

| | amap-skill | google-maps-skill |
|---|---|---|
| 命令定义 | 300 行 switch | 每个命令一个文件，声明式 |
| 参数校验 | 490 行 switch | 通用 schema 驱动 ~55 行 |
| HTTP | 仅 GET | GET + POST |
| 认证 | 仅 query param | query 或 header，按命令配置 |
| 路线命令 | 8 个（4 模式 x 2 变体） | 1 个 `directions --mode` |
| 新增命令 | 改 4+ 个文件 | 新建 1 个文件 + 注册加 1 行 |

## License

[MIT](./LICENSE)
