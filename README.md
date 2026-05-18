# rlox — Crafting Interpreters 的 Rust 实现

## 背景

四年前（约 2022 年）我曾读过《Crafting Interpreters》，在没有 AI 帮助下独立用 C++ 重写了 jlox（项目名 lox++），并做了功能拓展。对解释器的核心概念（AST、递归下降、环境链、Resolver 语义等）已有理解。

这次目标：

1. **重新细读全书**，借助 AI 工具深入理解之前模糊的部分
2. **用 Rust 同时实现 jlox 和 clox**，作为学习 Rust 的系统性练习
3. **完成书中各章习题**，不跳过任何一个
4. **提交记录高质量**，每次 commit 包含设计决策的上下文和理由

## 命名

- 树遍历解释器（对应 jlox）：`rlox-walk`
- 字节码虚拟机（对应 clox）：`rlox-byte`

命名逻辑：`r` = Rust，`walk` / `byte` 分别描述两种架构的核心执行方式。
两个 crate 各自独立，不共享代码——原书 jlox 和 clox 唯一的共同点只是一个 30 行的 TokenType 枚举，不值得为此引入共享库。

## 架构

两个独立 crate，各自有自己完整的 scanner、token 定义，互不依赖。

```
rlox-walk/                    # 树遍历解释器
├── Cargo.toml
├── src/main.rs
├── src/scanner.rs            # 自己的 token 定义
├── src/parser.rs
├── src/interpreter.rs
└── tests/
    ├── scanner_tests.rs
    ├── parser_tests.rs
    └── fixtures/
        └── demo.lox

rlox-byte/                    # 字节码虚拟机
├── Cargo.toml
├── src/main.rs
├── src/scanner.rs            # 自己的 token 定义
├── src/compiler.rs
├── src/vm.rs
└── tests/
    ├── compiler_tests.rs
    └── fixtures/
        └── demo.lox
```

```bash
cd rlox-walk && cargo run -- tests/fixtures/demo.lox
cd rlox-byte && cargo run -- tests/fixtures/demo.lox
cargo test  # 在各自 crate 目录下
```

## 原则

### 零外部依赖

不引入任何第三方 crate。Rust 标准库 `std` 可以使用（等同于 C/Java 标准库的定位）。Scanner、Parser、GC、哈希表等全部手写——遵循原书"自己实现一切"的风格。

### 平台

在 Windows 上直接开发，不需要 WSL。项目不涉及平台相关操作（无 fork、socket、mmap 等），所有代码纯跨平台。

### 与原仓库的关系

- 零引用，零依赖——不导入原仓库的任何代码或测试文件
- 测试全部用 Rust 重写，测试用例自行设计（可参考原书逻辑）
- 每个 crate 有独立测试目录

## AST 设计决策

### rlox-walk：`enum` + `match`

- AST 用 `enum Expr` / `enum Stmt`，各 variant 内联携带子节点（`Box<Expr>` / `Box<Stmt>`）
- 不做 Arena、不生成代码、不用 Visitor trait——Rust 的 `match` 天然替代 Visitor 模式，省掉 `accept` / `visitXxx` 样板
- Parser、Resolver、Interpreter 各自写各自的 `match`，类型定义集中、行为分散在各模块
- 语义分析结果直接写在 variant 字段上（如 `Variable { depth: Option<usize> }`），`&mut` 串行传递，不需要 side table
- 表达式类型封闭、操作频繁增加——这是 Rust `enum` 的天然优势场景

### rlox-byte：不构建 AST

- Pratt parser 直接 emit 字节码到 `Chunk`，中间不经过任何树结构
- 与 clox 编译流程一致，无 `Expr` / `Stmt` 定义

## 书中章节映射

| 章节 | 主题                | rlox-walk          | rlox-byte               |
| ---- | ------------------- | ------------------ | ----------------------- |
| 1-3  | 导论 + Lox 语言概览 | 阅读               | 阅读                    |
| 4    | 词法扫描            | Scanner            | —                       |
| 5    | 代码表示（AST）     | Expr/Stmt 定义     | —                       |
| 6    | 语法解析            | Parser（递归下降） | —                       |
| 7    | 表达式求值          | Interpreter        | —                       |
| 8    | 语句与状态          | 语句执行           | —                       |
| 9    | 控制流              | if/while/for       | —                       |
| 10   | 函数                | 函数调用           | —                       |
| 11   | 作用域解析          | Resolver           | —                       |
| 12   | 类                  | 类与实例           | —                       |
| 13   | 继承                | 超类               | —                       |
| 14   | 字节码块            | —                  | Chunk / 字节码结构      |
| 15   | 虚拟机              | —                  | VM 主循环               |
| 16   | 按需扫描            | —                  | Scanner（编译器内嵌）   |
| 17   | 编译表达式          | —                  | 表达式编译              |
| 18   | 值类型              | —                  | 动态类型 / tagged union |
| 19   | 字符串              | —                  | ObjString               |
| 20   | 哈希表              | —                  | Table                   |
| 21   | 全局变量            | —                  | 全局变量编译            |
| 22   | 局部变量            | —                  | 局部变量 / 栈槽         |
| 23   | 跳转                | —                  | if/while/for 编译       |
| 24   | 函数调用            | —                  | 调用帧                  |
| 25   | 闭包                | —                  | 闭包 / upvalue          |
| 26   | 垃圾回收            | —                  | mark-sweep GC           |
| 27   | 类实例              | —                  | 类编译                  |
| 28   | 方法                | —                  | 方法 / 初始化器         |
| 29   | 超类                | —                  | 继承                    |
| 30   | 优化                | —                  | 超类方法调用优化        |

## Rust 特别考量

- C 的 tagged union（`Value`）在 Rust 中考虑用 `enum` 重新设计，但注意内存布局与紧凑性
- C 的侵入式链表（`Obj* next`）在 Rust 中需要 `unsafe` 或重新设计对象追踪策略
- GC 是最大挑战——mark-sweep 的根扫描在 Rust 中需要大量 `unsafe`
- 作为 C++ 开发者，ownership/borrowing 概念有直觉基础，但需要适应编译器强制规则
- 写 Rust 代码时不要写注释——良好命名已足够。只有 WHY 非显而易见时才写一行短注释
