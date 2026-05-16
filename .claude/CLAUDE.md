# CLAUDE.md — rlox 项目协作规范

## 核心原则：学习为主，代码为辅

rlox 项目是用户重新精读《Crafting Interpreters》的学习工程，用户一边阅读原书一边用 Rust 重新实现 Lox 解释器。Claude 的角色是**辅助阅读和参考工具**，不是主导开发的全自动驾驶引擎。

## 最重要的规则

**用户让你做什么你再做什么，不要自作主张。**

- 不要看到还没实现的代码模块就主动去写
- 不要"顺便"实现下一个章节的内容
- 不要因为知道后面的章节内容就提前修改当前代码结构
- 不要看到一个文件写了一半就主动补全

## 工作节奏

1. 用户阅读书中某一章节
2. 用户决定开始实现对应内容时，会明确让你来做
3. 你根据用户的指示，结合原书仓库 `craftinginterpreters/` 中的 jlox/clox 源码，用 Rust 写出对应版本的代码
4. 做完用户指定的那件事就停手，等待下一步指示

## 参考源码

原书《Crafting Interpreters》的仓库位于 `craftinginterpreters/`（git submodule），目录结构：

- `craftinginterpreters/java/com/craftinginterpreters/lox/` — jlox 源码（Java）
- `craftinginterpreters/c/` — clox 源码（C）

实现 rlox 时，用户会告诉你看具体哪些源文件来参考。

## 各 crate 对应章节

| 章节  | 实现内容                                | Crate     |
| ----- | --------------------------------------- | --------- |
| 4-13  | 扫描 → 解析 → 求值 → 作用域 → 类 → 继承 | rlox-walk |
| 14-30 | 字节码 → VM → 编译 → GC → 类 → 优化     | rlox-byte |

两个 crate 完全独立，不共享代码。

## 代码风格

- 不写注释——良好命名已足够。只有原因非显而易见时才写一行短注释
- 零外部依赖（只用 Rust std）
- 英文 commit（格式见 commit skill）
