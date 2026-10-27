# worklog

从 git 历史生成周报草稿。**多仓库合并、按目录自动分组、过滤噪声提交。**

零第三方依赖，一个 Python 文件，不用连任何云服务。

```
$ worklog week --repo . --last-week
  my-project: 12 笔有效 / 18 笔总计
[OK] 已生成: 周报_20261025.md
  共 12 笔有效提交，过滤 6 笔噪声
```

## 为什么做这个

写周报最费时间的不是写，是**回忆这周干了什么**。而 `git log` 已经记下来了——但直接贴出来又不能用，因为：

1. **噪声淹没有效内容。** 一次改动里混着 `Merge origin/master`、`自动发布`、`chore(release): v1.2.3`，这些占了提交的一半，但一个字的活儿都不算。
2. **一次工作跨多个仓库。** 前端一个仓库、后端一个、脚本又一个，周报却必须合成一份。
3. **没有结构。** 领导想看的不是 commit 列表，是「这周在几个模块上做了什么」。

worklog 解决的就是这三件事。它不替你写"下周计划"，那个只有你知道。

## 安装

零依赖。把 `worklog.py` 放到任意位置即可：

```bat
python worklog.py week --repo . --last-week
```

要求 Python 3.10+（用了 `X | None` 的类型注解）。想要 docx 输出再装 `python-docx`，不装也能用。

## 快速开始

```bat
:: 本周（周一到今天）
python worklog.py week --repo .

:: 上一整周
python worklog.py week --repo . --last-week

:: 指定周期 + 指定输出
python worklog.py week --repo . --since 2026-10-19 --until 2026-10-25 --out 周报.md

:: 多个仓库合并成一份
python worklog.py week --repo D:\proj-web --repo D:\proj-api --last-week

:: 开发记录（按模块分节，适合挂到任务系统）
python worklog.py record --repo . --module 用户中心 --since 2026-10-01
```

## 输出长什么样

```markdown
# 工作周报（2026-10-01 ~ 10-09）

## 概览

- 有效提交 **3** 笔（已过滤噪声 2 笔）
- 覆盖 **1** 个模块分组

## my-project

### UI2（3 笔）

- `5c290a2f` 2026-10-08 删除标准制度管理单位fid字段
- `cb95ac7f` 2026-10-08 删除部分开发配置权限
- `d672c4c7` 2026-10-09 fix(IT资产): 清单 1/2/3 落地

## 工时（待填）

| 模块 | 工时 |
|---|---|
| UI2 | |

## 下周计划（待填）
```

## 配置

在仓库根目录放一个 `worklog.json`（也可放在任意上级目录，会向上查找）：

```json
{
  "title": "工作周报",
  "person": {"name": "", "role": ""},
  "repos": [
    {"path": ".", "name": "前端"},
    {"path": "D:\\proj-api", "name": "后端"}
  ],
  "modules": {
    "src/components/": "组件库",
    "src/pages/": "业务页面",
    "server/": "服务端"
  },
  "ignore_commits": ["^Merge ", "^自动发布$", "^chore\\(release\\)"]
}
```

- **`modules`**：路径前缀 → 给人的模块名。没匹配上的提交按**顶层目录**归组。
- **`ignore_commits`**：正则，匹配到的提交不进周报。会叠加内置默认规则（`Merge branch`、`自动发布`、纯版本号等）。
- **`repos`**：不传 `--repo` 时用这里的列表。

## 内置的噪声规则

```
^Merge (branch|remote|pull request)
^Merge 
^自动发布
^chore\(release\)
^v?\d+\.\d+\.\d+$
^pull commit$
^pull$
^update$
^更新$
^wip$
^\d+$
```

都可以在配置里追加。如果你的仓库有别的机器提交（比如 `[skip ci]`、`jenkins`），加一条就行。

## docx 输出

```bat
python worklog.py week --repo . --format docx
```

需要 `pip install python-docx`。没装会自动退回 Markdown 并提示，不会报错退出。

## 常见问题

**生成的周报里没有内容？**
先确认 `--since/--until` 覆盖到了提交。git 的 `--since` 用的是**本地时间**，跨时区可能差一天。

**某个模块名不对？**
检查 `modules` 里配的路径前缀，要按 `/` 分隔（脚本内部会把 `\` 统一成 `/`）。没配的按顶层目录归组。

**能生成日报吗？**
`--since` 和 `--until` 传同一天就是日报。

## License

MIT
