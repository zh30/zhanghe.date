---
title: 用 Rust 给 JavaScript 加速：WebAssembly 实战
date: 2025-03-05
---

JavaScript 嘛，大家都知道，是门脚本语言，离 CPU 直接运行的机器码还是有点距离的。但是，JavaScript 有个秘密武器，叫做 WebAssembly (Wasm)，可以让它执行接近机器码的东西！Wasm 是一种底层、可移植的二进制格式，能在浏览器里跑出接近原生应用的速度！

Wasm 特别适合像 C、C++、Rust 这样的语言编译后跑在浏览器里，让那些高性能应用，比如 [Google Earth](https://medium.com/google-earth/google-earth-comes-to-more-browsers-thanks-to-webassembly-1877d95810d6) 和 [Photoshop](https://web.dev/articles/ps-on-the-web)，直接在浏览器里溜起来。而且 Wasm 还超级安全，因为它有严格的沙盒机制，所以特别适合金融、医疗这种对安全性要求高的平台。现在 [Deno 2.1 已经完美支持 Wasm](https://deno.com/blog/v2.1#first-class-wasm-support) 啦，用 Wasm 模块比以前更简单了！

这篇文章就来教大家怎么创建一个简单的 Wasm 模块，并且通过它在 JavaScript 里调用 Rust 代码。

- [搞一个 Wasm 模块](https://deno.com/blog/intro-to-wasm#building-a-wasm-module)
- [让 JavaScript 通过 Wasm 调用 Rust](https://deno.com/blog/intro-to-wasm#call-rust-from-javascript-via-wasm)
- [下一步搞啥？](https://deno.com/blog/intro-to-wasm#whats-next)

## 咱们先搞个 Wasm 模块

先来创建一个简单的 Wasm 模块，然后把它导入到 Deno 里。

咱们从一个小小的 `add` 函数开始，用 [WebAssembly 文本格式](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format)写。新建一个文件，命名为 `add.wat`，然后把下面的代码放进去：

```wat
(module
(func (export "add") (param $a i32) (param $b i32) (result i32)
local.get $a
local.get $b
i32.add
)
)
```

用 [wat2wasm](https://github.com/webassembly/wabt) 把这段代码编译成 `add.wasm`：

```bash
wat2wasm add.wat -o add.wasm
```

好啦，现在把 `add.wasm` 导入到 Deno 里试试：

```javascript
import { add } from "./add.wasm"

console.log(add(1, 2))
```

跑一下，结果是：

```
3
```

Deno 导入 Wasm 的时候，会识别它的导出项，并且会[做类型检查](https://docs.deno.com/runtime/reference/wasm/#type-checking)哦！想了解更多关于 Wasm 导入的知识，可以[参考文档](https://docs.deno.com/runtime/reference/wasm/)。

这个例子很简单吧？不过，大多数时候我们都会用 Rust、C++ 或者 Go 来编译 Wasm，而不是直接手写 `wat`。

## 让 JavaScript 通过 Wasm 调用 Rust

接下来，我们来用 [wasmbuild](https://github.com/denoland/wasmbuild) 把 Rust 函数导入到 JavaScript 里。这个 CLI 工具可以生成一些胶水代码，方便你在 JavaScript 里调用 Rust 代码，它背后用了 [wasm-bindgen](https://rustwasm.github.io/docs/wasm-bindgen/introduction.html)。

首先，确认你已经安装了 Deno 和 Rust (`deno -v`, `rustup -v`, 和 `cargo -v`)。新建一个目录，然后创建一个 `deno.json` 文件：

```json
{
  "tasks": {
    "wasmbuild": "deno run -A jsr:@deno/wasmbuild@0.19.0"
  }
}
```

用 `new` 参数运行一下这个 task：

```bash
$ deno task wasmbuild new
Task wasmbuild deno run \-A jsr:@deno/wasmbuild@0.19.0 "new"
Creating rs\_lib...
To get started run:
deno task wasmbuild
deno run mod.js
```

这会在 `rs_lib` 目录下生成一个 Rust crate，里面包含了一些示例函数和测试：

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
a + b
}

#[wasm_bindgen]
pub struct Greeter {
name: String,
}

#[wasm_bindgen]
impl Greeter {
#[wasm_bindgen(constructor)]
pub fn new(name: String) -> Self {
Self { name }
}

pub fn greet(&self) -> String {
format!("Hello {}!", self.name)
}
}

#[cfg(test)]
mod tests {
use super::*;

#[test]
fn it_adds() {
let result = add(1, 2);
assert_eq!(result, 3);
}

#[test]
fn it_greets() {
let greeter = Greeter::new("world".into());
assert_eq!(greeter.greet(), "Hello world!");
}
}
```

这个文件定义了一个叫做 `add` 的函数，它接收两个有符号整数，返回一个有符号整数。还有一个叫做 `Greeter` 的结构体，里面也有两个函数。通过 `#[wasm_bindgen]` 这个属性，可以把这些东西导出，供 JavaScript 使用。

你可以在这里写你自己的 Rust 代码，不过，我们这里就用这些生成的代码作为例子。

接下来，运行 `wasmbuild` task 来构建这个项目：

```bash
deno task wasmbuild
```

这会生成一些文件：

- `lib/rs_lib.internal.js`
- `lib/rs_lib.js`
- `lib/rs_lib.d.ts`
- `lib/rs_lib.wasm`
- `mod.js`

我们可以用 [Wasm Code Explorer](https://wasdk.github.io/wasmcodeexplorer/) 把生成的 wasm 二进制文件 `lib/rs_lib.wasm` 可视化一下：

![Rust 函数可视化](https://deno.com/blog/intro-to-wasm/rust-add-greet-wasm.webp)

现在来导入它。最后一个文件 `mod.js` 其实包含了怎么在 JavaScript 里导入 Rust 函数的例子：

```javascript
import { add, Greeter } from "./lib/rs_lib.js"

console.log(add(1, 1))

const greeter = new Greeter("world")
console.log(greeter.greet())
```

这会导入 `add` 和 `Greeter`，这些函数原本是用 Rust 写的，但是被转换成了 JavaScript 可以用的形式。你可以运行 `deno mod.js` 试试：

```bash
$ deno mod.js
2
Hello world!
```

成功啦！

_想了解更多关于 Rust 和 JavaScript 结合使用的知识？看看 [用 Rust 撸一个 JavaScript 运行时](https://deno.com/blog/roll-your-own-javascript-runtime)吧。_
