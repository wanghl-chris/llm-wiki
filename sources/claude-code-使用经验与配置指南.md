---
title: Claude Code 使用经验与配置指南
source_type: article
created: 2026-05-09
source_hash: caba16e8
keywords: ["claude-code", "模型配置", "开发工具", "第三方模型接入", "JSON5"]
---

# 摘要
本文为 Claude Code 开发辅助工具的官方及社区使用经验汇总，系统梳理了4种默认调用模型的切换方案，明确了核心配置文件的语法规则，同时提供了多场景适配的配置策略，可帮助开发者灵活调配模型资源，根据不同任务需求匹配最优模型，提升日常开发效率。

# 核心观点
- Claude Code 提供4种模型切换路径，配置优先级遵循「环境变量 `ANTHROPIC_MODEL` > 细分模型指定变量 > 交互界面设置」的规则，同时支持全局/项目级双层配置生效
- 核心配置文件 `settings.json` 原生支持 JSON5 语法，允许添加注释、省略键名引号、保留尾随逗号，大幅降低配置的维护成本
- 支持接入第三方大模型服务商或本地部署的大模型服务，可根据任务复杂度匹配不同能力等级的模型，平衡使用成本与响应速度
- 所有配置修改完成后，可通过内置的 `/status` 命令快速验证模型、网关地址等参数是否生效，避免配置失效问题

# 详细内容
## 1. 默认模型切换方法
### 1.1 修改配置文件（最推荐）
通过修改 `settings.json` 配置文件切换模型，支持全局或单项目分层生效：
- 全局生效配置路径：`~/.claude/settings.json`（适用于 macOS/Linux 系统）
- 项目内生效配置路径：在项目根目录创建 `.claude/settings.json`

**配置优先级规则**：环境变量 `ANTHROPIC_MODEL` > 具体模型变量（如 `ANTHROPIC_DEFAULT_SONNET_MODEL`）

**完整配置模板**：
```json
{
  "env": {
    "ANTHROPIC_MODEL": "your-model-name",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "your-main-model",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "your-standard-model",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "your-fast-model",
    "ANTHROPIC_BASE_URL": "https://api.provider.com/v1",
    "ANTHROPIC_AUTH_TOKEN": "your-api-key"
  }
}
```
配置完成后输入 `/status` 即可确认模型是否切换成功。

### 1.2 环境变量配置（适合临时切换）
如需临时修改模型或网关地址，可通过导出环境变量的方式实现，无需修改永久配置：
```bash
export ANTHROPIC_BASE_URL="https://api.siliconflow.cn/"
export ANTHROPIC_MODEL="moonshotai/Kimi-K2-Instruct-0905"
export ANTHROPIC_API_KEY="YOUR_API_KEY"
claude
```

### 1.3 交互界面切换（仅限官方模型）
在 Claude Code 交互界面中，可通过内置命令快速切换 Anthropic 官方模型：
```
/model          # 打开交互式模型选择器
/model sonnet   # 直接切换到 Sonnet 4.6 模型
/model haiku    # 直接切换到 Haiku 4.5 模型
```

### 1.4 图形化配置工具（Desktop 端专属）
Claude Code 桌面端用户可通过开发者模式配置第三方推理服务：
1. 点击菜单栏 `Help → Troubleshooting → Enable Developer Mode`，重启工具生效
2. 点击菜单栏 `Developer → Configure Third-Party Inference`
3. 按提示填入推理网关地址与 API Key 即可完成配置

## 2. 配置文件语法特性
**核心结论**：Claude Code 的 `settings.json` 原生支持 JSON5 语法，因此可正常使用注释与非标准 JSON 特性，具体支持能力包括：
- 使用 `//` 添加单行注释
- 使用 `/* */` 添加多行注释
- 配置项键名可以不加双引号
- 配置项末尾的尾随逗号不会触发语法报错
该特性为 Claude Code 配置的核心优化点，官方示例中也大量使用注释说明配置项作用。

## 3. 实用使用建议
1. **多配置管理**：可使用开源工具 `cc-switch`，通过图形化界面管理多套模型配置，无需手动修改配置文件
2. **多模型适配**：支持接入智谱 GLM、七牛云等第三方大模型服务商，也可对接本地部署的 LM Studio、Ollama 等本地大模型服务
3. **配置验证**：所有配置修改完成后，务必使用 `/status` 命令验证参数生效状态，避免配置错误导致的调用失败
4. **分层配置策略**：可针对不同任务类型配置对应模型：复杂推理/架构设计任务使用 Opus 等高能力模型，日常开发/代码调试使用 Sonnet 等标准模型，简单格式化/信息查询使用 Haiku 等轻量模型，平衡成本与效率

# 关键概念
- **Claude Code**：Anthropic 推出的面向开发场景的AI辅助工具，提供命令行与桌面端两种形态，可调用大模型完成代码编写、调试、项目重构、文档生成等全流程开发任务
- **settings.json**：Claude Code 的核心配置文件，支持「全局配置（用户目录下）」与「项目级配置（项目根目录下）」两层结构，用于定义推理网关地址、API 密钥、默认调用模型等核心参数
- **JSON5 语法**：JSON 格式的非官方扩展标准，在完全兼容原生 JSON 语法的基础上，新增了单行/多行注释、无引号键名、尾随逗号等特性，更适合人工编写和维护配置文件
- **分层配置策略**：本文推荐的 Claude Code 配置思路，即针对不同复杂度的开发任务，匹配对应能力等级、响应速度、调用成本的大模型，实现资源的最优分配

# 引用与来源
1. DeepSeek 对话分享：Claude Code 配置相关问题解答
2. Z.AI 开发者文档
3. 百度云文档
4. 阿里云开发者社区