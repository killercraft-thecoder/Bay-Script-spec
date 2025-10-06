# 🌀 Bay Script

Bay Script is a brutally minimal, dynamically executed scripting language designed for raw power and tiny interpreters. It’s not trying to be easy—it’s trying to be possible.

## ✨ What Is Bay Script?

Bay Script is a low-level, expression-driven language with:
- Explicit control flow (`goto`, `call`, `while`)
- Linked variable buffers via `cstr`
- Dynamic module execution (`module.run`)
- Basic I/O (`input`, `print`)
- No closures, no classes, no fluff

It’s designed to be:
- **Tiny**: You can write an interpreter in under 500 lines.
- **Flexible**: Execution model is up to you—AST walker, bytecode VM, JIT, or even Excel.
- **Powerful**: You can build dynamic loaders, plugin systems, and even OS shells.

## 📜 Spec

The full language specification is in [`spec.md`](./spec.md).  
It defines the required behavior, built-in modules, and language rules.  
Execution model and implementation details are intentionally left flexible.
