# Origin Language 综合示例集

> 定位：用定案语法编写的验收样本——覆盖全部特性的完整程序。每个示例附「走查记录」：覆盖点与走查发现的问题。
>
> 语法基线：`int x = 1;`、`Person { ... }`、`B + A { ... }`、`Box<T> { ... }`、构造器、`f(args)`、中缀操作、`[1, 2, 3]`、`a[n]` / `a.length`、`s.at(n)` / `s.length`、`if / while / for / try-catch / return`、`mode` / `expect` / `load`、`main`。

## 示例 1：组合与可替换（接口 + 可替换）

```
mode(strong);

Animal {
  String name;
  String speak();
}

Dog + Animal {
  String speak() { return "dog " + this.name; }
}

Cat + Animal {
  String speak() { return "cat " + this.name; }
}

run(Animal a) {
  print(a.speak());
}

main(String[] args) {
  run(Dog("wang"));
  run(Cat("miao"));
}
```

**走查记录**：
- `Animal` 含无体方法 `speak();` → 接口方法；强类型下不可实例构造（未构造它）✓；
- `Dog + Animal` = 组合复用 + 名义声明实现 Animal（speak 有实现）→ 可替换资格 ✓；
- `run(Dog("wang"))`：Dog 用于 Animal 位置 → 静态判定（组合声明 + 无体方法全部实现）✓；运行时 `a.speak()` 经实例类型值找到 Dog 实现 ✓；
- `Dog("wang")`：Dog 无显式构造器 → 字段顺序构造，参数按声明顺序对应字段——Dog 未声明自己的字段，提升自 Animal 的 `name` 即其字段，参数 `"wang"` 对应 `name` ✓；
- `"dog " + this.name`：String + String 拼接 ✓。

## 示例 2：泛型 + 数组 + 分支循环 + 失败捕获

```
mode(strong);

Box<T> {
  T value;
  T get() { return this.value; }
}

int sum(int[] a) {
  int s = 0;
  for (int x : a) {
    s = s + x;
  }
  return s;
}

main(String[] args) {
  int[] nums = [1, 2, 3, 4];
  print(sum(nums));

  Box(int) b = Box(int)(42);
  print(b.get());

  Box(String) s = Box(String)("hi");
  print(s.get());

  try {
    print(nums[10]);
  } catch (e) {
    print("out of bounds");
  }

  int i = 0;
  while (i < nums.length) {
    print(nums[i]);
    i = i + 1;
  }
}
```

**走查记录**：
- `Box(int)(42)`：类型实例化 + 实例构造（参数是类型值 → 类型构造；普通值 → 实例构造）✓；
- 增强 for + while + 元素访问 `nums[i]` + `nums.length` ✓；
- `nums[10]` 越界 → 诊断值 → try/catch 结构捕获（块内结构分号结尾）✓；
- `print(nums)` 会整体输出数组（print 对任何值成立，呈现由环境决定）✓。

## 示例 3：多组合 + 构造器重载 + 模式区域

```
mode(strong);

A {
  int id;
  int show() { return this.id; }
}

B {
  String name;
  String label() { return this.name; }
}

C + A + B {
  int id;
  String name;
  C(int id, String name) {
    this.id = id;
    this.name = name;
  }
  C(int id) {
    this.id = id;
    this.name = "";
  }
}

C c1 = C(1, "one");
C c2 = C(2);
print(c1.show());
print(c2.label());

mode(weak) {
  print(c1);
}

main(String[] args) {
  print("done");
}
```

**走查记录**：
- `C + A + B` 多组合：成员提升（show / label 可用）✓；
- 覆盖同类型字段（id、name）——值直接共用，构造器参数只收子结构字段 ✓；
- 构造器重载按参数声明匹配（`C(1, "one")` / `C(2)`）✓；显式构造器存在时字段顺序构造不再适用 ✓；
- `c1.show()`：提升方法体 `this.id` 在合并成员表中解析到 C 覆盖的 id（int）✓ 类型一致；
- `mode(weak) { ... }` 区域级声明（工程强类型下的弱类型区域）✓；`print(c1)` 弱类型呈现 ✓。

## 示例 4：多文件工程（load 载入 + main 入口 + 两种执行方式）

工程由三个文件组成，入口文件由载入环境指定（《工程层设计》「入口文件」）：

`app.o`（入口文件）：
```
mode(strong);
load("math.o");
load("greet.o");

main(String[] args) {
  int n = int(args[0]);
  String name = args[1];
  print(add(n, 10));
  print(greet(name));
}
```

`math.o`：
```
int add(int a, int b) {
  return a + b;
}
```

`greet.o`：
```
String greet(String name) {
  return "hello " + name;
}
```

**走查记录**：
- `mode(strong);` = 入口文件顶层第一个值的工程级声明，整个工程强类型（《类型系统设计》「工程级声明」）✓；
- `load("math.o")` 单参数形态：相对链接相对该 load 所在文件（app.o）的所在位置解析 → 同目录 ✓；不传策略 → 按优先级链落到脚本内部声明 / 工程级声明（math.o / greet.o 无内部声明 → 按工程级声明 strong 解释）——整个工程按强类型模式解释，与「工程级声明」「其余文件不重复声明」一致；按工程级声明解释不构成穿透，main 中直接调用 add / greet 按强类型检查成立 ✓；
- 被载入文件顶层依次解释：`add` / `greet` 绑定进入全局绑定表（《工程层设计》「名称空间」）→ main 中直接调用 ✓；
- 幂等：同一次运行中同一文件只载入解释一次——main 内再次 `load("math.o")` 不重复解释（无重复发生的作用，《工程层设计》「load 载入语义」）✓；
- 编译执行：顶层解释完成（载入 + 声明）后自动应用 `main(命令行参数序列)`——`args[0]` / `args[1]` 是启动方传入的 String 数组元素，`int(args[0])` 从 String 解析为 int（《语义设计》「转换：类型值可应用」）✓；
- 解释执行：不自动应用，main 是普通名称——执行它 = 显式 `main(["5", "Jack"])`（《工程层设计》「两种执行方式」）✓；
- 位置约束：被载入文件含 `mode(P);` → 诊断（非入口文件）——本示例被载入文件无 ✓；被载入文件中的 main（若有）= 普通名称 ✓；
- `load(link, weak)` 传入策略形态与 expect 边界见《类型系统设计》「传入式调用」/「expect 断言操作」（本示例未用）。

