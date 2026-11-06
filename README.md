# C# from the Ground Up

A self-paced workbook for learning C# properly: the language first, then object-oriented design, then working with data. You'll write everything in the console, so you're learning C# itself rather than fighting an engine at the same time.

But every stage also shows you **where that idea lives in Godot**, in both GDScript and Godot C#. If you already make games in GDScript, you'll find you know more C# than you think, and you'll finally see *why* Godot works the way it does.

This repo covers **Blocks A to C**. When you finish Block C you'll be able to read and write the kind of C# that sits underneath a real game, and the final project ends with porting your console game into Godot.

| File | Block | What you'll learn |
|------|-------|-------------------|
| [`block-a-fundamentals.md`](block-a-fundamentals.md) | A | Types, operators, branching, loops, arrays, methods, strings, debugging |
| [`block-b-object-oriented.md`](block-b-object-oriented.md) | B | Classes, inheritance, interfaces, polymorphism, structs, enums, records |
| [`block-c-collections-linq.md`](block-c-collections-linq.md) | C | Lists, dictionaries, generics, delegates, lambdas, LINQ, events |

---

## How this works

Each block is split into **stages** (A1, A2, and so on). Every stage has the same shape:

1. **The idea.** A short explanation with examples. Read it, then type the examples out and run them. Don't copy and paste. Your fingers learning the syntax is part of the point.
2. **Check yourself.** A few quick questions. Answer them in your head or on paper *before* opening the answers at the bottom of the stage.
3. **Exercises.** Problems for you to solve. Some say exactly what to use. Some deliberately don't, because recognising *which* tool a problem needs is the real skill, and nobody labels problems for you in the wild.
4. **Stretch** (optional). Harder variations if you're flying.
5. **🎮 In Godot.** The same idea in a Godot project: what it looks like in GDScript, what it looks like in Godot C#, and where you'll bump into it in a real game.

There are no solutions to the exercises in this repo. That's on purpose. If you're stuck, re-read the idea, run smaller experiments, use the debugger, or ask your teacher. Bring finished work for feedback rather than asking someone to write it for you.

### A running theme

Most examples and exercises are set in a small **turn-based battle game**: characters with health, attacks, healing, status effects like poison. By the end of Block C you'll have built most of the logic for one, entirely in the console. It's the same kind of code that powers real games; the graphics are the easy bit to bolt on later.

### Algorithms strand

Scattered through the blocks are **algorithm challenges** marked with 🧮. These are the classic searches, sorts and data structures (binary search, insertion sort, linked lists, and so on). Each one appears at the earliest point you have the tools to build it. If you're doing A-level Computer Science, these are the same algorithms you trace on paper, except here you write them for real.

---

## Setup

You need the **.NET SDK** (version 8 or newer). Check if you already have it:

```bash
dotnet --version
```

If that prints a version number, you're set. If not, install the SDK from Microsoft's .NET download page.

Then make a playground project to do all your exercises in:

```bash
dotnet new console -n Playground
cd Playground
dotnet run
```

You should see `Hello, World!`. Open the folder in VS Code (with the C# Dev Kit extension) or JetBrains Rider.

**Tip:** make a new folder or project per stage (`A1`, `A2`...) or keep one project and comment out old exercises. Whatever you pick, use Git and commit after each stage. Future you will want to look back at how far you've come.

### A note on autocomplete

Your editor will try to finish your code for you. For the **exercises**, consider turning AI completions off. You learn by recalling, and autocomplete recalls for you. For bigger projects, turn it back on.

---

## Progress tracker

Tick these off as you go.

**Block A: Fundamentals**
- [ ] A1 Program structure, types, value vs reference
- [ ] A2 Operators and expressions
- [ ] A3 Branching and boolean logic
- [ ] A4 Loops
- [ ] A5 Arrays
- [ ] A6 Methods and recursion
- [ ] A7 Strings and characters
- [ ] A8 Debugging and exceptions
- [ ] Block A checkpoint

**Block B: Object-oriented C#**
- [ ] B1 Classes, properties, constructors, encapsulation
- [ ] B2 Inheritance, interfaces, polymorphism
- [ ] B3 Structs, enums, records, tuples
- [ ] Block B checkpoint

**Block C: Collections, generics, LINQ**
- [ ] C1 Collections and generics
- [ ] C2 Delegates, lambdas and LINQ
- [ ] C3 Events
- [ ] Block C final project
