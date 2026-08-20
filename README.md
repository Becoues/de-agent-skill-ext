# de-agent-skill-ext

给 AI 编码助手（Claude Code 等）用的团队协作规范 skill，由数据工程（DE）维护。

## 安装

```bash
npx -y skills add Becoues/de-agent-skill-ext -s '*' -a claude-code -y -g
```

`-g` 装到用户级（`~/.claude/skills/`），所有仓库都生效，只需装一次；想只对当前仓生效就去掉 `-g`。

显式写 `-a claude-code` 是有必要的：不加它，CLI 在识别不到 agent 的目录里会装到 `agent/skills/` 和 `.agents/skills/`，Claude Code 读不到。

装完再配一个 hook（让判定不依赖模型自觉，且不占基础上下文）：见 [`data-change-notify/references/hook-setup.md`](data-change-notify/references/hook-setup.md)，或直接对 Claude Code 说「按 data-change-notify 的 hook-setup 配好 hook」。

## 包含的 skill

| skill | 作用 |
|-|-|
| [`data-change-notify`](data-change-notify/SKILL.md) | 改数据库 schema、字段读写逻辑、数据写入开关或回填历史数据时，判定是否需要周知数据工程，并生成周知文案。 |

## 为什么需要 data-change-notify

业务库的数据会被同步进数据仓库，再层层加工成看板、财务报表和对外材料。数据管线的故障**绝大多数是静默的**——它很少报错，更常见的是安静地算出一个看起来正常的错数字，然后进入财务报表。

有些改动数据团队能自动吸收（加字段），有些能发现但滞后一天（删字段、改名），有些**无论加多少监控都发现不了**（单位从分改成元、回填历史数据）。这个 skill 就是帮你在改动的当场把它归到对应的那一档，只在真有必要时才提醒你说一声。

判定原则和三层清单见 [`data-change-notify/SKILL.md`](data-change-notify/SKILL.md)。

## 这些 skill 不做什么

- 不阻断提交，不修改你的代码
- 不自动发送任何消息，文案交给你，发不发你决定
- 不联网、不上报、不读取业务数据

## 维护

数据工程团队维护。内容与实际不符时提 issue 或 PR。
