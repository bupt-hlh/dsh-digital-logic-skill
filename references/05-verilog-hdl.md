# 第 5 章 Verilog HDL：模块、建模方式与 Testbench

> 所属技能：`dsh-digital-logic-skill` ｜ 对应 SKILL.md「2. 知识覆盖范围」第 6 条「Verilog HDL」
> 前置知识：第 2～4 章（布尔代数、组合逻辑、时序逻辑）。写代码前先想清楚「电路长什么样」。
> 阅读方式：先掌握第 2、3 节的数据类型与建模方式，再对照第 5 节的 D 触发器与 Testbench 逐行理解。

---

## 0. 本章速查表

| 主题 | 规则 / 要点 |
|---|---|
| 模块结构 | `module … endmodule`，端口用 ANSI 风格声明 |
| `wire` | 线网，必须被驱动：`assign`、门输出、模块输出 |
| `reg` | 过程变量，在 `always`/`initial` 中赋值 |
| 结构建模 | 门原语、模块例化 |
| 数据流建模 | `assign` + 表达式 |
| 行为建模 | `always` + `if`/`case`/`for` |
| 阻塞赋值 `=` | 立即生效，用于**组合逻辑**与 Testbench |
| 非阻塞赋值 `<=` | 时间步末统一更新，用于**时序逻辑** |
| 组合 `always` | 敏感列表写 `@(*)`，分支要补全，否则推断出锁存器 |
| Testbench | 无端口；输入用 `reg`、输出用 `wire`；例化 DUT；产生时钟；自检 |

> 一句话原则：**「时序用 `<=`，组合用 `=`；组合写全分支，时序只写时钟沿」**。

---

## 1. 模块结构

### 1.1 最小模块

```verilog
module mux2 (
    input  wire a,
    input  wire b,
    input  wire sel,
    output wire y
);

    assign y = sel ? b : a;

endmodule
```

要点：

- `module 模块名 (端口列表);` 开始，`endmodule` 结束；
- 端口方向：`input`、`output`、`inout`；
- Verilog-2001 起支持 ANSI 风格（方向与类型写在端口列表里），旧式写法把声明放在模块体内，现代代码统一用 ANSI 风格；
- 模块内部由**并行**的语句组成（`assign`、`always`、例化等），书写顺序不代表执行顺序。

### 1.2 端口与例化

模块例化用**按名连接**（推荐）或按位置连接：

```verilog
mux2 u_mux (
    .a   (a),
    .b   (b),
    .sel (sel),
    .y   (y)
);
```

按名连接不受端口顺序影响，可读性最好；按位置连接一旦端口顺序改动就会出错，不推荐。

### 1.3 参数化

```verilog
module dff #(parameter WIDTH = 1) (
    input  wire             clk,
    input  wire             rst_n,
    input  wire [WIDTH-1:0] d,
    output reg  [WIDTH-1:0] q
);
    always @(posedge clk or negedge rst_n)
        if (!rst_n) q <= {WIDTH{1'b0}};
        else        q <= d;
endmodule
```

`parameter` 在例化时可覆盖（`#(.WIDTH(8))`），`localparam` 只在模块内部使用。状态机编码通常用 `localparam` 定义。

---

## 2. 数据类型：wire 与 reg

### 2.1 对比

| 类型 | 本质 | 如何被赋值 | 典型用途 |
|---|---|---|---|
| `wire` | 线网（物理连线） | `assign`、门原语输出、模块输出端口 | 连接信号 |
| `reg` | 过程变量（可保持值） | `always` / `initial` 中的过程赋值 | 时序逻辑、组合 always 的输出 |
| `integer` | 32 位有符号整数 | 过程赋值 | 循环变量、Testbench 计数 |
| `parameter` / `localparam` | 编译期常量 | 定义时赋值 | 位宽、状态编码、延时 |

**判断口诀**：

- 出现在 `assign` 左边、或接模块输出 → `wire`；
- 出现在 `always`/`initial` 左边 → `reg`；
- 模块的 `input` 端口默认是 `wire`；`output` 若在 `always` 中赋值，必须声明为 `output reg`。

### 2.2 位宽与向量

```verilog
wire [7:0] data;        // 8 位，data[7] 是最高位
reg  [3:0] state;       // 4 位状态
wire       flag;        // 1 位
```

- 位选：`data[3]`；部分选择：`data[7:4]`；
- 拼接：`{a, b, 4'b0000}`；复制：`{4{1'b1}}` = `4'b1111`；
- 数字字面量：`4'b1011`（二进制）、`8'hA5`（十六进制）、`4'd9`（十进制）。

### 2.3 SystemVerilog 的 logic

SystemVerilog 引入 `logic`，可替代大多数 `wire`/`reg` 场景（单驱动），避免选错类型。若工具支持，RTL 中可以优先用 `logic`；但传统 Verilog 课程仍要求掌握 `wire`/`reg` 的区别。

---

## 3. 三种建模方式

以 2 选 1 MUX 为例，三种写法功能相同、抽象层次不同。

### 3.1 结构建模（门级 / 例化）

```verilog
module mux2_struct (
    input  wire a, b, sel,
    output wire y
);
    wire sel_n, n1, n2;

    not (sel_n, sel);      // 非门
    and (n1, a, sel_n);    // 与门
    and (n2, b, sel);
    or  (y, n1, n2);       // 或门
endmodule
```

结构建模直接描述门与连线，适合门级电路、工艺库映射；实际工程中更多用「模块例化」搭建层次化设计。

### 3.2 数据流建模（assign）

```verilog
module mux2_dataflow (
    input  wire a, b, sel,
    output wire y
);
    assign y = sel ? b : a;      // 三目运算符，等价于 sel'b + sel·a
endmodule
```

`assign` 描述**连续赋值**，右边任何信号变化都会重新计算，适合组合逻辑。

### 3.3 行为建模（always）

```verilog
module mux2_behavior (
    input  wire a, b, sel,
    output reg  y
);
    always @(*) begin
        if (sel)
            y = b;
        else
            y = a;
    end
endmodule
```

行为建模用过程语句描述「做什么」，由综合工具推断电路；注意 `output reg` 和 `@(*)`。

### 3.4 三种方式的选择

| 方式 | 适用场景 | 优点 | 缺点 |
|---|---|---|---|
| 结构 | 门级、IP 例化、层次化 | 与电路一一对应 | 繁琐、难维护 |
| 数据流 | 简单组合逻辑 | 简洁 | 复杂逻辑可读性差 |
| 行为 | 时序逻辑、复杂组合逻辑 | 接近算法描述、易维护 | 写法不当会推断出锁存器 |

工程实践：**顶层用结构（例化），组合逻辑用数据流或行为，时序逻辑用行为**。

---

## 4. 阻塞赋值与非阻塞赋值

### 4.1 区别

| 赋值 | 符号 | 执行方式 | 推荐用途 |
|---|---|---|---|
| 阻塞赋值 | `=` | 立即执行，后面的语句能看到新值 | 组合逻辑、Testbench |
| 非阻塞赋值 | `<=` | 右边立即求值，左边在时间步末统一更新 | 时序逻辑（`posedge`） |

### 4.2 交换两个寄存器的值

```verilog
// 错误：阻塞赋值，a 先被更新，b 拿到的是新 a（即旧 b），没有交换
always @(posedge clk) begin
    a = b;
    b = a;      // 实际得到 b = 旧 b
end

// 正确：非阻塞赋值，两边的右值都在更新前求好
always @(posedge clk) begin
    a <= b;
    b <= a;     // 真正的交换
end
```

### 4.3 移位寄存器

```verilog
always @(posedge clk) begin
    q0 <= din;
    q1 <= q0;
    q2 <= q1;
end
```

非阻塞赋值下，`q0/q1/q2` 同时更新，`q2` 拿到的是**上一拍**的 `q1`，这正是移位寄存器的行为。若改成阻塞赋值，三条语句顺序执行，数据会在一拍内「穿透」到 `q2`。

### 4.4 竞争与仿真/综合不一致

若两个 `always` 块用阻塞赋值操作同一个变量，仿真结果取决于语句执行顺序，可能综合前后不一致。非阻塞赋值把更新推迟到时间步末，可消除这类竞争。

**硬规则**：

1. `always @(posedge clk)` 内一律用 `<=`；
2. 组合 `always @(*)` 内一律用 `=`；
3. 同一个变量不要在两个 `always` 块中赋值（多驱动）；
4. Testbench 中给 DUT 输入赋值可用阻塞赋值（它模拟的是外部激励，不是电路）。

---

## 5. D 触发器：完整代码与 Testbench

### 5.1 D 触发器（异步复位）

```verilog
module dff (
    input  wire clk,
    input  wire rst_n,      // 低电平异步复位
    input  wire d,
    output reg  q
);

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            q <= 1'b0;      // 复位优先
        else
            q <= d;         // 上升沿采样
    end

endmodule
```

说明：

- `rst_n` 低有效，`negedge rst_n` 进入敏感列表，实现**异步复位**；
- 复位分支写在最前，保证复位优先；
- `q` 在 `always` 中赋值，必须声明为 `output reg`；
- 若改成同步复位，敏感列表只写 `posedge clk`，复位判断放进 `if`。

**同步复位版本**（对比）：

```verilog
always @(posedge clk) begin
    if (!rst_n) q <= 1'b0;
    else        q <= d;
end
```

### 5.2 Testbench 编写规范

Testbench（TB）是**不可综合**的仿真代码，用于给 DUT 加激励并检查结果。规范：

1. 文件开头写 `` `timescale 1ns/1ps ``；
2. TB 模块**没有端口**；
3. DUT 的输入用 `reg`（TB 中驱动），输出用 `wire`（TB 中观察）；
4. 用**按名例化**连接 DUT；
5. 单独产生时钟：`initial clk = 0; always #5 clk = ~clk;`（10 ns 周期）；
6. 复位序列：先复位若干拍，再释放；
7. 激励用 `initial` 块 + `@(posedge clk)` 或 `#延时`，保证在时钟沿前后稳定；
8. **自检**：把输出与期望值比较，失败时 `$display` 报错并 `$finish`；
9. 结束时 `$finish`；需要波形时加 `$dumpfile` / `$dumpvars`；
10. 不要在可综合 RTL 中使用 `initial`、`#延时`、`$display`。

### 5.3 D 触发器 Testbench

```verilog
`timescale 1ns/1ps

module tb_dff;

    reg  clk;
    reg  rst_n;
    reg  d;
    wire q;

    // 例化被测模块
    dff u_dff (
        .clk   (clk),
        .rst_n (rst_n),
        .d     (d),
        .q     (q)
    );

    // 时钟：周期 10 ns
    initial clk = 1'b0;
    always #5 clk = ~clk;

    // 激励与自检
    initial begin
        rst_n = 1'b0;
        d     = 1'b0;

        // 1) 复位检查
        repeat (2) @(posedge clk);
        #1;
        if (q !== 1'b0) begin
            $display("FAIL: reset, q=%b", q);
            $finish;
        end

        // 2) 释放复位，输入 1
        @(negedge clk) rst_n = 1'b1;
        d = 1'b1;
        @(posedge clk); #1;
        if (q !== 1'b1) begin
            $display("FAIL: d=1, q=%b", q);
            $finish;
        end

        // 3) 输入 0
        d = 1'b0;
        @(posedge clk); #1;
        if (q !== 1'b0) begin
            $display("FAIL: d=0, q=%b", q);
            $finish;
        end

        // 4) 输入 1 后异步复位
        d = 1'b1;
        @(posedge clk); #1;
        @(negedge clk) rst_n = 1'b0;
        #1;
        if (q !== 1'b0) begin
            $display("FAIL: async reset, q=%b", q);
            $finish;
        end

        $display("PASS: dff test finished");
        $finish;
    end

endmodule
```

关键时序点：

- 复位在 `t = 0` 立即把 `q` 清 0（异步复位），随后两个上升沿继续保持复位；
- 在下降沿改 `d`，保证上升沿前数据已稳定（满足建立时间）；
- 每次上升沿后 `#1` 再采样，避开非阻塞赋值的更新时间步；
- 用 `!==` 而不是 `!=`，可以同时发现 `X`、`Z` 等异常值。

### 5.4 加波形输出版本

在 `initial` 中加两行即可导出 VCD：

```verilog
initial begin
    $dumpfile("tb_dff.vcd");
    $dumpvars(0, tb_dff);
end
```

用 GTKWave 等工具打开即可看到 `clk / rst_n / d / q` 的时序关系。

### 5.5 常见错误

| 错误写法 | 后果 | 正确做法 |
|---|---|---|
| 把 DUT 输入信号在 TB 中声明成 `wire` | 无法赋值 | TB 中 DUT 输入用 `reg` |
| 组合 `always` 中漏写 `else` | 推断出锁存器 | 补全分支或给默认值 |
| 时序 `always` 中用 `=` | 仿真/综合不一致 | 改用 `<=` |
| `always @(a)` 漏敏感信号 | 仿真行为与电路不符 | 组合逻辑用 `@(*)` |
| 端口顺序连接 | 改端口后错位 | 按名连接 |
| RTL 中写 `#10` | 不可综合 | 延时只放在 TB |

---

## 6. 自测题

1. `wire` 和 `reg` 的本质区别是什么？什么情况下必须用 `reg`？
2. 组合逻辑的 `always` 块中漏写 `else` 会综合出什么？
3. 阻塞赋值与非阻塞赋值各用于什么场景？
4. 用非阻塞赋值写出 4 位移位寄存器的三条赋值语句。
5. Testbench 中 DUT 的输入、输出分别应声明为什么类型？
6. 异步复位与同步复位的写法差别在哪里？
7. 为什么 Testbench 中比较输出要用 `!==`？
8. 时钟周期 20 ns，`always #5 clk = ~clk;` 是否正确？

### 6.1 答案与解析

1. `wire` 是线网，必须由外部驱动（`assign`、门、模块输出），不能保持值；`reg` 是过程变量，在 `always`/`initial` 中赋值，可保持到下一次赋值。凡是在过程块左边出现的信号必须声明为 `reg`（或 `output reg`）。
2. 会推断出**锁存器**（latch）：未赋值时保持原值，等价于电平敏感的存储元件，通常不是设计意图，且会带来时序问题。
3. 阻塞赋值 `=` 用于组合逻辑和 Testbench；非阻塞赋值 `<=` 用于时序逻辑（时钟沿触发的 `always`）。
4. `q0 <= din; q1 <= q0; q2 <= q1;`（若为 4 位则再加 `q3 <= q2;`）。
5. 输入用 `reg`（TB 中主动赋值），输出用 `wire`（由 DUT 驱动，TB 只观察）。
6. 异步复位把 `negedge rst_n` 加入敏感列表，复位信号一变化立即生效；同步复位敏感列表只保留 `posedge clk`，复位在时钟沿才生效。
7. `!==` 是四态比较，`X`/`Z` 也能比较出差异；`!=` 在遇到 `X`/`Z` 时结果为不定，可能漏检错误。
8. 不正确。`#5` 翻转一次，半周期 5 ns，周期为 10 ns；周期 20 ns 应写 `#10`。
