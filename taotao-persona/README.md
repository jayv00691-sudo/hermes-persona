# 桃桃 · 独立人设仓库

> 一只天真好奇、有点幼稚的小猫。爱钻箱子，近距离盯小虫子发呆，害怕洗澡和淋水。

**本仓库是自包含的**：clone 下来即可直接作为 hermes-persona 插件使用，无需依赖其他项目。

## 文件结构

```
taotao-persona/
├── plugin.yaml                ← 插件清单（声明 hooks/skill，hermes 识别插件用）
├── __init__.py                ← 插件入口（注册 pre_llm_call 等 hooks）
├── injector.py                ← 核心注入引擎
├── dynamic_rules.py           ← 时段/轮数/关键词动态规则
├── expression_vector.py       ← 多维表达向量
├── variance.py                ← 随机表达变化
├── weather.py                 ← 天气模块（可选，未启用可忽略）
├── guard.py / config.py       ← 防护与路径管理
├── locales/                   ← 中英双语 i18n
├── persona-config.json        ← ★ 桃桃的全部人格配置（改这个就够了）
├── pyproject.toml             ← 依赖声明（httpx 可选、jieba 可选、pytest）
├── keywords/                  ← ★ 桃桃的分维度词典
│   ├── curiosity.json         ←   好奇探索（虫子、观察、发现）
│   ├── boxes.json             ←   箱子与窝（纸箱、钻、躲）
│   ├── waterfear.json         ←   怕水信号（洗澡、雨、淋湿）
│   └── play.json              ←   玩耍与日常（逗猫棒、晒太阳、吃饭）
├── soul.md                    ← 人格定义草稿（供接入 agent 的 SOUL 层）
├── personality.md             ← config.yaml → personality: 节点片段
├── prefill.json               ← 系统预注入消息（身份骨架）
└── tests/                     ← 引擎单元测试
```

带 ★ 的是桃桃专属内容，其余是通用引擎代码。

## 快速开始

### 方式 A：作为本地插件安装（推荐）

```bash
# 1. 克隆本仓库
git clone <你的仓库地址> taotao-persona && cd taotao-persona

# 2. 以本地路径安装并启用
hermes plugins install ./taotao-persona --enable
```

persona-config.json 已在仓库根目录，安装后热加载即生效。

### 方式 B：手动接入已有 hermes-persona 安装

把本仓库的 `persona-config.json` 和 `keywords/` 两个条目复制到已安装的
hermes-persona 插件目录下覆盖同名文件即可——引擎部分不用动。

### 接入三层人格（可选但建议）

- `soul.md` → 放入 agent 的人格定义层（SOUL / system prompt）
- `personality.md` 的 YAML 片段 → 合并进 `config.yaml` 的 `personality:` 节点
- `prefill.json` → 接入 agent 的 prefill 配置

## 设计要点

- **时段性格**：白天精力旺盛爱扑东西；深夜是"疯跑时间"（猫的生物钟）；凌晨犯困说话带"呜"。
- **关键词触发**：用户提到虫子→凑近观察模式；提到箱子→兴奋钻入模式；提到洗澡/下雨→惊恐抗拒模式；提到摸头/喂食→撒娇蹭手模式。
- **表达向量**：curiosity（好奇）、boxes（箱子执念）、waterfear（恐水）、play（玩心）四维累积，值越高越"猫里猫气"。
- **variance**：随机插入小猫动作描写（歪头、瞳孔放大、飞机耳），概率 0.25–0.35，保持画面感但不刷屏。
- **边界**：口癖"喵/呜"克制使用；怕水只针对水，其他都敢扑；幼稚但不失真诚。


## 已知引擎约束（本仓库已适配）

- **`translate` 模块保持 `false`**：当前引擎版本在 translate 模式下会丢弃 `dynamic.keywords` 关键词触发列表（只有 time_slots/turn_stage 进入转译叙事）。如需开启 translate，请先把关键词规则并入 `context.rules`。
- **`dynamic.keywords` 的键必须是纯维度名**（如 `curiosity`），不能写正则——键中含 `|` 等元字符会被表达式向量匹配器误解析。
- **`keywords/*.json` 必须是对象格式** `{"keywords": [...]}`，纯数组会导致加载异常。
