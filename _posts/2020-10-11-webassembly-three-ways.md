---
layout: post
title: "WebAssembly Three Ways"
description: "Go vs Rust vs AssemblyScript"
categories: post
tags: WebAssembly Rust Golang AS js programming
---

[WebAssembly](https://webassembly.org/) is byte code for the web. It is an open standard that has been implemented by all of the major browsers (FireFox, Chrome, Safari, Edge). It has also been implemented as a compilation target for a number of programing languages. Which may have you asking, if I want to build something in WebAssembly what language should I use?

Now if you are starting with an existing project or library, the answer is easy. Just keep going with whatever language it's written in. But if you are starting something new, you have a number of options. Maybe too many options?

I decided to try a few new (to me) programing languages that compile to wasm and see if I can compare them.

## The Test

In order to best compare these different languages with different tooling I decided that they "apples to apples" comparison would be to implement the same program in each language.

The program in this case will be the Fermat Prime Check algorithm. It's something that is not too complex, but is also not trivial. This is something that I should be able to implement mostly the same regardless of language. Any differences in implementation or performance should be specific to the language and/or it's wasm tool chain.

I want to assess how easy it is to go from "zero to hero" in each language. These languages are going to be new to me. So while there may be some very clever or obscure way of doing things, that may provide incredible performance gains, that wouldn't provide a fair comparison. And someone new to the language would not know these tricks.

In addition, I will try to add some hard numbers on top of the subjective measurements, by doing some size and speed comparisons. Size comparison is simply the size on disk. And the speed test was performed using [Benchmark.js](https://benchmarkjs.com/) and the same set of 9 prime numbers for each.
```
[5, 13, 277, 2999, 10151, 154877, 5297879, 15485339, 694847533]
```

## The Contenders

For this I am starting with comparing [AssemblyScript](https://www.assemblyscript.org/), [Go](https://golang.org/), and [Rust](https://www.rust-lang.org/). These are three languages that I have not used before trying to work with WebAssembly. 

The language I probably have the most of an edge with would be AssemblyScript. It is a variant of TypeScript, which is basically a type layer on JavaScript, a language I am very familiar with. In the case of Go and Rust I went through some tutorials before writing the Prime checker for this test. In fact, I wrote an [earlier post when I did the Rust implementation](/2020/06/28/bigger-than-a-breadbox-learning-rust-and-webassembly.html).

## AssemblyScript

[My Code](https://github.com/ianmcodes/as-primecheck) |
[Live Demo](https://www.ianmccall.codes/as-primecheck/)

### Learning the Language

For me AssemblyScript was the easiest to learn since I already know JavaScript. The important thing to remember is what types AssemblyScript supports. They support signed and unsigned 32 and 64 bit integers, 32 and 64 bit floats, a 128 bit vector, and not much else. AssemblyScript does provide smaller integer types, bool, and some special types, but those map to 32 bit integers.

Note that there was no mention of char, string, or any other types. AssemblyScript does support these types, but WebAssembly does not. That is important to know for this and all other languages that compile to WebAssembly. WebAssembly only supports very simple integer and float types. So if you want to pass anything else you need to put it into the "memory" and pass a pointer. Fortunately most languages that compile to Wasm provide a loader that will handle those thing for us.

For the Fermat prime checker, we don't need anything more complex that a 64 bit integer or float. But it's something you will want to keep in mind if you want to pass strings or more complex objects to Wasm modules.

To export functions to JS you use the same `export` keyword you use to export functions from any JS module. It's important to keep in mind that, while you are essentially writing code is JavaScript, you do not automatically have access to all JS or DOM functions. If you want your module to have access to something like `console.log` you need to declare it in your module and pass it in through the WebAssembly `env`. You can find more information about imports and exports in AS here (https://www.assemblyscript.org/exports-and-imports.html).

### Tool Chain

The AssemblyScript tools are easily installed using `npm`. Installing the `assemblyscript` module gives you access to `asinit` and `as` commands, which you can run using `npx`. The `asinit` command which scaffolds an AssemblyScript project. After that you can use the `asc` command to compile your project. The compiler supports a number of optimization options and can generate binary, wasm text format, and JS equivalent all at the same time.

To load and run your compiled code on a page you will need to use the `@assemblyscript/loader` module. Because of that, I decided to leverage "browserify" to be able to build a script to run in the browser.

## Rust

[My Code](https://github.com/ianmcodes/rust-primecheck) |
[Live Demo](https://www.ianmccall.codes/rust-primecheck/)

### Learning the language

In a previous post I talked about [learning Rust](/2020/06/28/bigger-than-a-breadbox-learning-rust-and-webassembly.html#learning-rust). The short version is that the way it works is very different from other languages I've used. The hardest thing to get used to is how extremely strict Rust is about variables. For somebody that is used to playing fast and loose with variables in JS and other web programing languages, the way Rust handles variables requires some getting used to. But the good thing is that the build tools are really good at pointing out what you are doing wrong and how to fix it.

### Tool Chain

Besides Rust and Cargo, to build for WebAssembly you will need to install `wasm-pack` from here (https://rustwasm.github.io/wasm-pack/installer/). Once you have that and you setup your `Cargo.toml` file, it's as simple as running `wasm-pack build --release -t web`. The build process creates your binary and a JS loader so you can more easily load your wasm binary on your page.

## Go

[My Code](https://github.com/ianmcodes/go-primecheck) |
[Live Demo](https://www.ianmccall.codes/go-primecheck/)

### Learning the language

To learn Go (or GoLang) I started with the [Learn Go](https://www.codecademy.com/learn/learn-go) tutorial on Codecademy. Go has a C like syntax, sans semicolon (`;`). All in all, I didn't find it too hard to learn, especially compared to Rust. Variables can be explicitly typed or implied. It works the way you would expect a modern compiled language to work.

Interfacing with JavaScript is really where Go is very different from other languages that compile to Wasm. 

_A note on tinygo_: If you have looked into using Go to build Wasm modules you have probably come across "tinygo". I did try it out, and it did result in a much smaller binary, but I couldn't get it to run correctly. It's important to note that Wasm support between Go and tinygo differ in several ways. I will post a follow up if I get it to work properly.

### Tool Chain

The standard install of GoLang has everything you need to target WebAssembly. So to create your binary you just need to run `GOOS=js GOARCH=wasm go build -o main.wasm`.

To load the binary on a page you will need to copy `wasm_exec.js` from `{GoHome}/misc/wasm/` to your project. Then you can load the script on your page and use the `Go()` class it provides to run you wasm binary.

## Performance

### Size

| Language | Binary Size |
|----------|-------------|
| AssemblyScript | 3.2K |
| Rust | 33K |
| Go | 1.4M |

As you can see AssemblyScript is the clear winner size wise. But the really surprizing result here is how huge the Go binary is! The go compiler doesn't provide any optimization options it's self. In fact the guidance from the Go team for [reducing the size of Wasm files](https://github.com/golang/go/wiki/WebAssembly#reducing-the-size-of-wasm-files) is to just use gzip or brotli compression. Or to switch to using TinyGo.

### Benchmark

| Language | Average Execution Time (ms) |
|----------------|------|----|
| AssemblyScript | 0.03146 (+/-0.85%) |
| Rust | 0.09850 (+/-3.41%) |
| Go | 0.27862 (+/-1.78%) |

Again, a win for AssemblyScript, but much closer this time. Rust was only an average of 0.06704ms slower than AS. Though it's also important to point out that AS was much more consistent from run to run than Rust or Go.

## Conclusion?
From this, you maybe convinced that AssemblyScript is the way to go, but I think it's a little premature to make that conclusion. This test is limited to just passing in numbers, not complex objects that would require using the `ArrayBuffer` memory and passing pointers. And even then, the AssemblyScript version only works because Chrome added support for passing `BigInt` into wasm in Chrome 85. The loaders for Rust and Go handle that complication for you. So, depending on the complexity of what you are trying to do, you may find yourself better off accepting the size overhead that comes with Rust or Go.
