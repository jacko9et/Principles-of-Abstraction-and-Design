# 抽象与设计的基本原则<br>Principles of Abstraction and Design

一套关于抽象与设计的原创文档体系：先以「抽象与设计的基本原则」确立立场与公理，再以「Origin Language 核心抽象设计」将其应用到编程语言设计，最后以实验性设计文档展开具体方向。

A set of original design documents on abstraction and design: it starts with *Principles of Abstraction and Design* to establish the stance and axioms, then applies them to programming language design in *Origin Language Core Abstract Design*, and finally unfolds concrete directions in the experimental design documents.

**Keywords:** abstraction · design principles · programming language design · computational ontology · axiomatic system · observation layers · value · operation · composition · environment · language theory · Chinese · English

## 文档结构 / Document Structure

三层结构，建议按顺序阅读。<br>Three layers, recommended to read in order.

| 层级 / Layer | 中文 / 简体中文原文 | English / 英文版本 |
| --- | --- | --- |
| 先导 / Foundation | [抽象与设计的基本原则](抽象与设计的基本原则.md) | [Principles of Abstraction and Design](English/Principles%20of%20Abstraction%20and%20Design.md) |
| 核心应用 / Core Application | [Origin Language 核心抽象设计](Origin%20Language%20核心设计文档/Origin%20Language%20核心抽象设计.md) | [Origin Language Core Abstract Design](English/Origin%20Language%20Core%20Design%20Docs/Origin%20Language%20Core%20Abstract%20Design.md) |
| 实现者指南 / Implementer's Guide | [Origin Language 实现者指南](Origin%20Language%20核心设计文档/Origin%20Language%20实现者指南.md) | [Origin Language Implementer's Guide](English/Origin%20Language%20Core%20Design%20Docs/Origin%20Language%20Implementer's%20Guide.md) |
| 实验性设计 / Experimental Designs | [实验性设计目录](Origin%20Language%20核心设计文档/实验性设计/README.md) | [Experimental Designs Directory](English/Origin%20Language%20Core%20Design%20Docs/Experimental%20Designs/README.md) |

### 实验性设计 / Experimental Designs

| 中文 / 简体中文原文 | English / 英文版本 |
| --- | --- |
| [Origin Language 语法设计](Origin%20Language%20核心设计文档/实验性设计/Origin%20Language%20语法设计.md) | [Origin Language Syntax Design](English/Origin%20Language%20Core%20Design%20Docs/Experimental%20Designs/Origin%20Language%20Syntax%20Design.md) |
| [Origin Language 语言结构设计](Origin%20Language%20核心设计文档/实验性设计/Origin%20Language%20语言结构设计.md) | [Origin Language Language Structure Design](English/Origin%20Language%20Core%20Design%20Docs/Experimental%20Designs/Origin%20Language%20Language%20Structure%20Design.md) |
| [Origin Language 语义设计](Origin%20Language%20核心设计文档/实验性设计/Origin%20Language%20语义设计.md) | [Origin Language Semantics Design](English/Origin%20Language%20Core%20Design%20Docs/Experimental%20Designs/Origin%20Language%20Semantics%20Design.md) |
| [Origin Language 类型系统设计](Origin%20Language%20核心设计文档/实验性设计/Origin%20Language%20类型系统设计.md) | [Origin Language Type System Design](English/Origin%20Language%20Core%20Design%20Docs/Experimental%20Designs/Origin%20Language%20Type%20System%20Design.md) |
| [Origin Language 工程层设计](Origin%20Language%20核心设计文档/实验性设计/Origin%20Language%20工程层设计.md) | [Origin Language Engineering Layer Design](English/Origin%20Language%20Core%20Design%20Docs/Experimental%20Designs/Origin%20Language%20Engineering%20Layer%20Design.md) |
| [Origin Language 综合示例集](Origin%20Language%20核心设计文档/实验性设计/Origin%20Language%20综合示例集.md) | [Origin Language Comprehensive Examples](English/Origin%20Language%20Core%20Design%20Docs/Experimental%20Designs/Origin%20Language%20Comprehensive%20Examples.md) |

## 核心概念 / Core Concepts

- **四元**：值、操作、组合、环境——最小闭合系统中的四个必要位置。<br>
  **The Fourfold**: Value, Operation, Composition, Environment—the four necessary positions in a minimally closed system.

- **删除实验**：判断一个概念是否属于核心的方法——删除它，体系是否仍然闭合。<br>
  **Deletion Experiment**: The method for judging whether a concept belongs to the core—delete it, and see whether the system still closes.

- **判定五问**：新概念进入体系前的审查流程。<br>
  **Five Review Questions**: The review process before a new concept enters the system.

- **结构保持，环境生长**：结构不随环境变化，新能力从环境与结构的关系中生长出来。<br>
  **Structure Preserved, Environment Grows**: The structure does not change with the environment; new capabilities grow from the relation between environments and structures.

## 阅读建议 / Reading Guide

1. 先读「抽象与设计的基本原则」，理解立场、公理与判定方法。<br>
   Read *Principles of Abstraction and Design* first to understand the stance, axioms, and methods of judgment.

2. 再读「Origin Language 核心抽象设计」，看公理如何应用于语言设计。<br>
   Then read *Origin Language Core Abstract Design* to see how the axioms apply to language design.

3. 需要动手实现时，读「Origin Language 实现者指南」（含基于 Lua 的最小核心示例）。<br>
   When implementing, read *Origin Language Implementer's Guide* (includes a minimal core example in Lua).

4. 最后按兴趣翻阅「实验性设计」下的各方向文档。<br>
   Finally, browse the experimental design documents as your interest leads.

> 注意：实验性设计目录下的文档是探索性内容，与先导文档和核心应用文档的层级不同。<br>
> Note: Documents under Experimental Designs are exploratory content, on a different level from the foundational and core application documents.
