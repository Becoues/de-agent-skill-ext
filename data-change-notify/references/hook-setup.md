# 配一个 hook，让判定不依赖模型的自觉

## 为什么需要

skill 的触发是**概率性的**——模型读了 description 自己决定要不要调用。改一行 migration 就漏掉判定完全可能。

hook 是**确定性的**：由 Claude Code 本体执行，与模型判断无关。而且 hook **不占基础上下文**（它不在 prompt 里，只有真触发时那一行输出才进入对话）。

两者配合：hook 负责「一定会被提醒」，skill 负责「提醒之后怎么判」。

## 安装

把下面这段合并进 `settings.json`。二选一：

- **只对当前仓生效** → `<repo>/.claude/settings.json`
- **对所有仓生效**（推荐，一次配好） → `~/.claude/settings.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "p=$(grep -oE '\"file_path\"[^,]*' | head -1); case \"$p\" in *migrat*|*alembic*|*schema*|*entity*|*.sql*|*nacos*|*ddl*) echo '数据变更提醒：此改动可能影响数据仓库。请用 data-change-notify 判定是否需周知数据工程；不确定就按需要周知处理。';; esac"
          }
        ]
      }
    ]
  }
}
```

已经有 `hooks` 配置的，把 `PostToolUse` 数组里的那一项追加进去，不要整段覆盖。

嫌手动改麻烦，可以直接对 Claude Code 说：**「按 data-change-notify 的 hook-setup 配好 hook」**，让它替你合并。

## 验证

配完后随便编辑一个路径里带 `migration` 的文件，应当看到那行提醒。没看到就检查 JSON 是否合法、以及是否放在了生效的那个 settings 文件里。

## 调这套匹配模式

默认模式是**故意收紧的**，只盯这几类路径：数据库迁移目录、`.sql` 文件、schema / entity 定义、DDL、开关配置。

调整时记住一条：**被无视的提示比没有提示更糟**。太松会每次改代码都刷提醒，两周后所有人都会条件反射跳过它，等于白配。

建议做法：从紧开始跑一两周，看实际漏掉了什么，再针对性放宽——而不是一上来就把所有可能相关的路径都写进去。

已知的坑：不要把 `model` 加进模式。很多业务代码库里 `model` 是高频词（业务模型、算法模型、3D 模型），匹配它基本等于每次都触发。改成更具体的写法，比如 `*entity*`、`*/schema/*`、`*_table.go`。

## 这个 hook 不做什么

- **不阻断**任何操作。它只 echo 一行字，退出码始终为 0。
- **不读文件内容**，只看 `file_path` 字段。
- **不联网、不上报**。
