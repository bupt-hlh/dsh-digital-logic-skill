# dsh-digital-logic-skill

**数字逻辑课程 AI 助教技能** ｜ A study-assistant skill for a digital logic course

> 适用于 DeepSeek Harness（DSH）以及任何兼容 Agent Skills（`SKILL.md`）格式的 AI 工具。纯 Markdown，无代码依赖，开箱即用。
>
> Works with DeepSeek Harness and any Agent Skills–compatible tool. Markdown only, no build step.

---

## 简介

本技能把数字逻辑课程的完整知识体系整理为 8 章参考文档，让 AI 以「助教」身份工作：**先讲定义、再分步推导、最后规范地写 Verilog**，而不是直接丢答案。

覆盖范围：

- 进制与编码（二/八/十/十六进制、8421 BCD、格雷码、原/反/补码）
- 布尔代数、公式法化简、卡诺图
- 逻辑门、编码器、译码器、MUX、加法器、比较器
- 触发器、寄存器、同步/异步计数器
- Verilog HDL 与 Testbench
- ROM/RAM 与存储器扩展
- PAL/GAL/CPLD/FPGA 与 LUT 原理
- Mealy/Moore 状态机、状态化简、三段式写法

## 目录结构

```text
dsh-digital-logic-skill/
├── SKILL.md                  # 技能入口：角色、知识范围、回答规范、索引
├── README.md
├── LICENSE
└── references/
    ├── 01-number-system.md
    ├── 02-boolean-algebra.md
    ├── 03-combinational.md
    ├── 04-sequential.md
    ├── 05-verilog-hdl.md
    ├── 06-memory-devices.md
    ├── 07-programmable-logic.md
    └── 08-fsm-design.md
```

## 章节索引

| 文件 | 核心内容 | 适用场景 |
| :--- | :--- | :--- |
| `references/01-number-system.md` | 进制转换、BCD码、格雷码、补码运算 | 进制转换计算、编码问题 |
| `references/02-boolean-algebra.md` | 布尔定理、公式法化简、卡诺图 | 逻辑函数化简、真值表 |
| `references/03-combinational.md` | 编码器、译码器、MUX、加法器、比较器 | 组合电路分析与设计 |
| `references/04-sequential.md` | 触发器、寄存器、计数器 | 时序电路分析、状态转换 |
| `references/05-verilog-hdl.md` | 模块结构、三种建模方式、Testbench | Verilog 编程、仿真验证 |
| `references/06-memory-devices.md` | ROM/RAM 结构、存储器扩展 | 存储容量设计 |
| `references/07-programmable-logic.md` | PAL/GAL/CPLD/FPGA 对比 | 可编程器件选型 |
| `references/08-fsm-design.md` | Mealy/Moore、状态化简、三段式写法 | 状态机设计、序列检测 |

## 安装

### DeepSeek Harness（DSH）

把整个 `dsh-digital-logic-skill` 文件夹放入任一技能根目录：

| 作用域 | 路径 |
| :--- | :--- |
| 用户级（所有项目可用） | `~/.dsh/skills/`（Windows：`C:\Users\<用户名>\.dsh\skills\`） |
| 项目级（仅当前项目） | `<项目根>/.dsh/skills/` |
| 项目级（兼容 agents 约定） | `<项目根>/.agents/skills/` |

```bash
# 示例：用户级安装
git clone https://github.com/<your-name>/dsh-digital-logic-skill.git
mkdir -p ~/.dsh/skills
cp -r dsh-digital-logic-skill ~/.dsh/skills/
```

安装后 DSH 会自动发现新技能（无需重启；目录监听会刷新技能目录），在对话中提问或显式指定技能名即可使用。

### 其他 Agent Skills 工具

把本文件夹放入该工具的技能目录（通常为 `~/.<tool>/skills/` 或项目内的技能目录），保持**目录名与 `SKILL.md` 中的 `name` 一致**（`dsh-digital-logic-skill`），具体以工具文档为准。

### 验证安装

提一个问题，例如：

```text
用卡诺图化简 F(A,B,C,D) = Σm(0,1,2,3,6,7,8,9,11,14)，并逐步说明圈组。
```

如果回答先给推导、并引用 `references/02-boolean-algebra.md` 的圈组原则，说明技能已生效。

## 使用示例

| 你可以这样问 | 会用到 |
| :--- | :--- |
| `(1101.101)₂` 转十进制并说明位权 | 01 进制与编码 |
| 用公式法化简 `AB + A'C + BC` | 02 布尔代数 |
| 用 8 选 1 MUX 实现 `F = Σm(1,2,4,7)` | 03 组合逻辑 |
| 设计 4 位同步二进制计数器并推导驱动方程 | 04 时序逻辑 |
| 给 D 触发器写一个自检 Testbench | 05 Verilog HDL |
| 用 1K×4 芯片扩展成 2K×8 存储器 | 06 存储器 |
| 4 输入 LUT 如何实现任意逻辑函数 | 07 可编程逻辑 |
| 设计 1011 序列检测器（重叠检测） | 08 状态机 |

## 设计原则

- **分步推导**：计算题展示完整过程（除基取余、卡诺图圈组、激励表推导），不直接给结果。
- **代码规范**：Verilog 示例区分 `wire`/`reg`，时序用非阻塞赋值，FSM 采用三段式，Testbench 带自检。
- **可核对**：例题与答案中的数值均逐项验算；每章末尾附自测题与解析。
- **只依赖 Markdown**：无脚本、无构建步骤，任何文本编辑器即可修改。

## 贡献

欢迎提交 Issue 或 Pull Request：

1. 保持中文表述与现有章节编号连续；
2. 新增例题需给出完整推导步骤；
3. 若调整文件结构，请同步更新 `SKILL.md` 与 README 的索引表。

## 许可证

[MIT](LICENSE)。可自由用于学习、教学与二次开发，请保留版权声明。

## 免责声明

教学内容用于课程学习与复习；具体芯片型号、引脚与电气参数请以厂商数据手册为准。

---

## English overview

A Markdown-only Agent Skill that turns an AI assistant into a digital-logic teaching assistant. It ships eight chapter references (number systems, Boolean algebra and K-maps, combinational and sequential logic, Verilog HDL, memories, programmable logic, and finite state machines) plus a `SKILL.md` that defines the tutoring behavior: step-by-step derivations, IEEE-style Verilog, and guided problem solving.

**Install (DeepSeek Harness):** copy this folder into `~/.dsh/skills/` (user scope) or `<project>/.dsh/skills/` (project scope). Keep the folder name identical to the `name` field in `SKILL.md`.

**Use:** ask a digital-logic question, e.g. *"Simplify F(A,B,C,D)=Σm(0,1,2,3,6,7,8,9,11,14) with a Karnaugh map and explain each group."*

Licensed under MIT.
