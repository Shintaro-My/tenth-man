# tenth-man

一个面向 [Codex](https://developers.openai.com/codex) 和 [Claude Code](https://claude.com/claude-code) 的多模型审议插件（包含一个技能和若干席位代理）。适用于设计选择、方针审查等不应只依赖单个模型回答的决策。由不同模型担任的席位进行投票，另有一个固定席位始终提出反对意见，从而降低议事会在共享同一盲点的情况下达成一致的可能性。

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (Brasil)](README.pt-BR.md) | [Deutsch](README.de.md) | [Français](README.fr.md)

投票成员会评估每一项提案。另设的**第十人**（tenth man）无论自己怎么看，都必须反对每一项提案，并说明什么样的证据可以推翻每一条反对意见。主会话担任主席，主持三轮讨论，处理重大疑虑，并撰写会议纪要，记录达成了哪些一致、理由是什么，以及哪些问题仍未解决。

该技能总是先给出议事会计划，并等待你的批准。运行议事会不会自动把结论应用到你的项目中。

## 工作方式

1. 主席提出议题、模型、规则、成本和输出文件夹。
2. 第 1 轮：成员投票；第十人对每个议题提出质疑。
3. 第 2 轮：成员重新审视重大疑虑，并回应最重要的反对意见。
4. 第 3 轮：成员对会议纪要草案投票；第十人对照证据核查结果。
5. 主席写出 `.tenth-man/<date>-<slug>/minutes.md`。

票分为**赞成**、**有条件赞成**、**反对**和**重大疑虑**。达成一致需要投票成员过半数，且没有遗留的重大疑虑。第十人没有投票权。

## 在 Codex 中安装

### 作为插件

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

如果使用本地检出的副本，改为添加它的路径：

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

输入 `$tenth-man:council` 并在后面写上问题即可调用插件技能，也可以从 `/skills` 中选择：

```text
$tenth-man:council 是否应该把会话状态从 Redis 迁移到签名 Cookie？
```

### 作为独立技能

将 `skills/council` 复制到 Codex 的技能目录：

```sh
cp -r skills/council ~/.agents/skills/
```

以这种方式安装时，用 `$council` 调用。

如果只在一个仓库中使用，则改为复制到 `<repo>/.agents/skills/`。本仓库的 `.codex/agents/` 下还包含不绑定角色的 Codex 席位配置，Codex 会在本仓库的检出目录中自动发现这些配置。如需在所有仓库中使用这些配置：

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

即使不复制，该技能也能正常工作：它可以用同样明确指定的模型启动子代理，并在各轮之间延续它们的对话线程。以插件形式打包时，不会自动安装自定义代理的 TOML 文件。

## 在 Claude Code 中安装

### 作为插件

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

测试本地检出的副本：

```sh
claude --plugin-dir /path/to/tenth-man
```

调用方式：

```text
/tenth-man:council 是否应该把会话状态从 Redis 迁移到签名 Cookie？
```

### 手动安装

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

手动安装后的命令是 `/council`，席位名称不带 `tenth-man:` 前缀。

## 模型预设

问题较小时，主席会建议 `light`，其他情况建议 `standard`。批准计划时，你可以选择预设，也可以更改任意席位。

第十人使用与推理能力最强的成员相同的模型。这个排名依据实测结果，而不是版本号；下表中的“第十人”列反映的是 2026 年 10 月时的排名。新模型的排名未知时，主席会询问你。如需使用其他模型，请在批准计划、同意开始时说明。

### Codex

| 预设 | 投票成员 | 第十人 | 每轮运行次数 |
|---|---|---|---|
| `light` | GPT-6 Astra、GPT-6.1 Sol、GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra、GPT-6.1 Sol、GPT-6.1 Terra、GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol、GPT-6.1 Terra、GPT-6 Sol | GPT-6.1 Sol | 4 |

这些默认配置只使用当前 Codex 宿主和账号提供的模型。主席会报告不可用的模型，绝不会在不告知的情况下换用其他模型。如果宿主的并发上限小于议事会的席位数，各席位会基于同一份未改动的任务说明分批运行；所有席位完成之前不会计票。

对应的 Codex 自定义代理是 `seat_astra`、`seat_sol`、`seat_terra`、`seat_previous_sol`，以及可选的低成本 `seat_luna`。它们的 TOML 定义固定了模型和推理强度，并让议事会的工作保持只读。

### Claude Code

| 预设 | 投票成员 | 第十人 | 每轮运行次数 |
|---|---|---|---|
| `light` | Fable、Opus、Sonnet | Fable | 4 |
| `standard` | Fable、Opus、Sonnet、Opus 5、Opus 4.8 | Fable | 6 |
| `no-fable` | Opus、Sonnet、Opus 5 | Opus | 4 |

即使 Fable 排名更高，每个预设也都保留一个 Opus 席位，因为 Opus 有时能发现 Fable 遗漏的问题。

Claude 插件在 `agents/` 中包含五个不绑定角色的席位代理。每次运行的角色（成员或第十人）由任务说明指定。

### 添加 Claude 模型

可以通过代理定义中的 `model:` 字段选择较早的 Claude 模型。复制一个席位文件，修改 `name`、`description` 和 `model`，然后保存到 `~/.claude/agents/`（用于所有项目）或 `.claude/agents/`（用于单个项目）：

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

正文保持与随附席位相同的、不绑定角色的内容。之后主席就可以把 `seat-opus-4-7` 指定为成员或第十人。

### 使用你自己的 Claude 代理

任何现有的 Claude 代理定义都可以担任席位。主席会在提示词开头说明这次任务是担任议事会席位，而不是该代理平时的工作，然后分配角色和共同的任务说明。对于固定在较早模型上的审查代理，这很有用。

### 固定的 Claude 模型停止提供时

如果固定的模型不再提供服务，该席位将无法启动。主席会移除该席位并告知你，绝不会在不告知的情况下换用其他模型。请删除或更新席位文件，使其与你的账号可用的模型一致。

## 成本与要求

每个席位每轮对应一次完整的子代理运行。三轮下来，Codex 的 `standard` 预设共运行 15 次，Claude Code 的 `standard` 预设共运行 18 次。

- Codex：已启用子代理的当前版本，以及所选模型的访问权限。
- Claude Code：Agent 和 SendMessage 工具、后台代理完成通知，以及所选模型的访问权限。

目前不支持 ChatGPT 对话，因为它无法为每个席位使用不同的模型运行子代理。

## 为什么叫“第十人”？

该技能的灵感来自 2013 年电影《僵尸世界大战》（*World War Z*）中虚构的以色列规则“第十人”。当九个人根据相同的信息得出相同的结论时，第十个人要假定他们是错的，并调查相反的可能性。这条规则是虚构的，但现实中存在类似的做法：

- 以色列军事情报部门在 1973 年赎罪日战争前误判了预警信号，此后设立了一个专门质疑自身评估的职能，通常称为 *ipcha mistabra*，这是阿拉米语，意为“很可能正好相反”。
- 《塔木德》（Sanhedrin 17a）记载，在死刑案件中，如果所有法官都投票定罪，被告就会被释放；全体一致的结果意味着没有人认真为被告辩护。
- 在天主教的封圣审理中，“魔鬼代言人”负责反对候选人，并检验有关候选人生平及神迹的说法中的薄弱之处。这个职位于 1587 年正式设立，1983 年被大幅削减。

## 许可证

MIT。参见 [LICENSE](LICENSE)。
