# 命题者说 Skill + 高三物理二轮命题分析套件

本仓库包含：

- `skill/mingti-zheshuo/`：题库 Agent 的“命题者说”Skill。覆盖数学、物理、化学、生物学、思想政治、英语的命题视角分析与高考式出卷，内置 SVG 试题配图工具。
- `physics-second-round/`：原创高三物理二轮模拟卷（力学＋电磁学，100 分/90 分钟）、学生/教师卷与逐题命题分析报告。
- `distillation-audit/`：从逐题报告中提取“物理二轮命题分析”能力的候选、三重验证、覆盖审计与边界记录。

## 安装 Skill

将 `skill/mingti-zheshuo/` 复制到题库 Agent 的 skills 目录，例如：

```bash
cp -R skill/mingti-zheshuo ~/.tikuagent/agent/skills/
```

重新打开会话后，即可用“按命题者说分析这道题”“按命题者说出一套高考题”等方式调用。

## 重要边界

- 模拟卷及报告均为原创命题初稿，**未经真实试测**，使用前须由教师逐题审校。
- 报告中没有实测作答数据；“学生表现/误区”均为可能路径，不是正确率、选项率或心理事实。
- `distillation-audit/` 当前完成到 cangjie 蒸馏的阶段 1.5（候选与三重验证）；能力尚未编译回 `mingti-zheshuo`。后续编译应保留其中的审校、数据边界与教学适配限制。
- 本公开包不含《命题者说》原始 PDF、OCR 全文或其他受版权保护的原书内容。

## 目录

```text
.
├── skill/mingti-zheshuo/
├── physics-second-round/
└── distillation-audit/
```
