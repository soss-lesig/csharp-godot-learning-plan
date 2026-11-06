# Block A: Language Fundamentals

Pure C#, in the console. No engine, no classes of your own yet. This block gives you the building blocks everything else is made from: types, maths, decisions, repetition, collections of data, functions and error handling.

Every stage ends with a **🎮 In Godot** section showing where the idea turns up in a Godot project, in both GDScript and Godot C#. If you already use GDScript, these are your bridge: you know more C# than you think.

**Done when:** every stage's exercises run correctly, and you can do the Block A checkpoint without looking anything up.

---

## A1. Program structure, types, value vs reference

### The idea

A brand new console project contains one line:

```csharp
Console.WriteLine("Hello, World!");
```

That's using **top-level statements**: C# lets the first file skip the ceremony. The longer version, which you'll see in older code and in every other file you write, looks like this:

```csharp
namespace Playground;

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello, World!");
    }
}
```

`Main` is where the program starts. Code lives inside **methods**, methods live inside **classes**, classes live inside **namespaces**.

**C# is statically typed.** Every variable has a type, fixed when you declare it:

```csharp
int health = 100;          // whole numbers
double critChance = 0.15;  // decimal numbers
float speed = 300.0f;      // decimal, less precise (note the f). Godot uses float a lot
bool isAlive = true;       // true or false
char grade = 'A';          // a single character, single quotes
string name = "Ash";       // text, double quotes

var gold = 50;             // the compiler works out the type (int). Still fixed!
```

Try `health = "lots";` and the compiler refuses. That's a feature: a whole category of bugs caught before you ever run the program.

**Value types vs reference types.** This is the single most important idea in A1, and it'll come back in every block.

- **Value types** (`int`, `double`, `bool`, `char`, and structs) hold their data directly. Copying one copies the data.
- **Reference types** (classes, arrays, strings) hold a *reference* to data stored elsewhere. Copying one copies the reference, so both variables point at the **same** thing.

```csharp
int a = 5;
int b = a;   // b gets its own copy of 5
b = 99;
Console.WriteLine(a);  // 5. a is untouched

int[] scoresA = { 1, 2, 3 };
int[] scoresB = scoresA;  // both point at the SAME array
scoresB[0] = 99;
Console.WriteLine(scoresA[0]);  // 99. changed through the other variable!
```

Think of a value type as writing a number on a sticky note and handing someone a photocopy. A reference type is handing someone the address of a house: you both now visit the same house, and if they repaint the door, you see it too.

`string` is a reference type, but it's **immutable** (it can never be changed after creation), so in practice it behaves like a value. You'll see why in A7.

### Check yourself

1. `int x = 10; int y = x; y++;` What is `x`?
2. Two array variables point at the same array. You change an element through one. Does the other see the change?
3. Why does `int count = 3.5;` fail to compile?

### Exercises

**A1.1** Write a program that asks for the player's name and age, then prints `"Welcome, <name>! In 10 years you'll be <age+10>."` You'll need `Console.ReadLine()` and `int.Parse(...)`.

**A1.2** What happens if the user types `twelve` for their age? Run it and read the error. You'll fix this properly in A8; for now just note what you see.

**A1.3** Write a short program that *proves* the value vs reference difference to someone who doesn't believe you. Print before and after values for both an `int` and an array.

### 🎮 In Godot

The same types exist in GDScript, with optional type hints:

```gdscript
var health: int = 100
var speed: float = 300.0
var player_name: String = "Ash"
```

In Godot C#, a script is a class. It **must** be `partial` (Godot generates the other half behind the scenes):

```csharp
using Godot;

public partial class Player : CharacterBody2D
{
    private int _health = 100;
    private float _speed = 300.0f;
}
```

**Value vs reference matters a lot in Godot.** `Vector2` is a **struct**, a value type. Nodes are classes, reference types. This is why the following, which works fine in GDScript, refuses to compile in C#:

```csharp
Velocity.Y = -300.0f;  // error CS1612!
```

`Velocity` hands you a *copy* of the vector. Changing `.Y` on a copy would change nothing, so C# stops you. The fix is to copy, change, then assign back:

```csharp
Vector2 v = Velocity;
v.Y = -300.0f;
Velocity = v;
```

You'll properly understand *why* once you meet structs in B3. For now, just recognise the error when you see it.

<details>
<summary>Answers</summary>

1. `10`. `int` is a value type, so `y` got its own copy.
2. Yes. Arrays are reference types; both variables refer to the same array.
3. `3.5` is a `double`, and squeezing it into an `int` would lose the `.5`. C# won't silently lose data, so it demands an explicit cast: `(int)3.5`, which gives `3`.

</details>

---

## A2. Operators, expressions, string interpolation

### The idea

Arithmetic works as you'd expect, with one big trap:

```csharp
int a = 9 / 2;          // 4, not 4.5!
double b = 9 / 2;       // still 4! The division happens as ints FIRST
double c = 9 / 2.0;     // 4.5. one side is a double, so it's double division
double d = (double)9 / 2;  // 4.5. cast before dividing
```

**Integer division throws away the remainder.** The type of the *operands* decides the type of the division, not the variable you store it in.

The **remainder** operator `%` gives you what integer division threw away:

```csharp
Console.WriteLine(9 % 2);   // 1
Console.WriteLine(10 % 5);  // 0  (so "x % 2 == 0" means x is even)
```

Shorthand operators:

```csharp
health -= 10;   // health = health - 10
gold *= 2;      // gold = gold * 2
turn++;         // turn = turn + 1
```

**String interpolation** puts values straight into text. Put a `$` before the quotes and expressions in `{}`:

```csharp
string name = "Ash";
int dmg = 37;
double crit = 0.15;

Console.WriteLine($"{name} deals {dmg} damage!");
Console.WriteLine($"Crit chance: {crit:P0}");    // Crit chance: 15%
Console.WriteLine($"Average: {12.0 / 7:F2}");    // Average: 1.71
Console.WriteLine($"[{dmg,5}]");                 // [   37]  right-aligned in 5 spaces
```

### Check yourself

1. What does `double result = 7 / 2;` store?
2. What's `17 % 5`?
3. Give two different ways to make `7 / 2` produce `3.5`.

### Exercises

**A2.1** A hit does `baseDamage * multiplier` damage, where `baseDamage` is an `int` and `multiplier` is a `double`. Store the result as a whole number, rounded *down*. Print it in a sentence.

**A2.2** Given a number of seconds (e.g. `3725`), print it as hours, minutes and seconds: `1h 2m 5s`. Use only `/` and `%`.

**A2.3** Print a little combat summary table using alignment specifiers, so the numbers line up in neat columns:

```
Attacker   Damage   Crit
Ash            37    15%
Bramble       112     5%
```

### 🎮 In Godot

The integer division trap is identical in GDScript: `9 / 2` is `4`. Godot even warns you about it (`INTEGER_DIVISION`).

You'll see this pattern in nearly every movement script, in both languages:

```gdscript
velocity.x = direction * SPEED
```

```csharp
velocity.X = direction * Speed;
```

And `delta` (the time since the last frame) is exactly the kind of `double`/`float` maths this stage is about. Multiplying by `delta` turns "per frame" into "per second", so the game runs at the same speed on a fast PC and a slow one.

GDScript string formatting uses `%` instead of `$"..."`:

```gdscript
print("%s deals %d damage!" % [player_name, dmg])
```

<details>
<summary>Answers</summary>

1. `3`. Both operands are `int`, so integer division happens before the result is stored.
2. `2` (17 = 3 × 5 + 2).
3. `7 / 2.0`, `7.0 / 2`, `(double)7 / 2`. Any version where at least one operand is a `double`.

</details>

---

## A3. Branching, boolean logic, switch

### The idea

```csharp
if (health <= 0)
{
    Console.WriteLine("Defeated!");
}
else if (health < 20)
{
    Console.WriteLine("Low health!");
}
else
{
    Console.WriteLine("Fighting fit.");
}
```

Combine conditions with `&&` (and), `||` (or), `!` (not). C# **short-circuits**: in `a && b`, if `a` is false, `b` is never even checked. That's useful:

```csharp
if (target != null && target.Health > 0) { ... }  // safe: never touches Health on a null target
```

**switch** picks between many fixed options. The modern **switch expression** returns a value:

```csharp
string rank = score switch
{
    >= 90 => "S",
    >= 75 => "A",
    >= 50 => "B",
    _     => "C"    // _ means "anything else"
};
```

**Boundary discipline.** Most branching bugs live at the edges. Is it `<` or `<=`? What happens at *exactly* 0? At exactly the maximum? Every time you write a comparison, ask: *what happens at the boundary value itself?* Then test it.

**Negative remainder.** In C#, the sign of `%` follows the left-hand number:

```csharp
Console.WriteLine( 7 % 3);  //  1
Console.WriteLine(-7 % 3);  // -1   (not 2!)
```

This bites whenever you want something to **wrap around**. If you subtract from 0 and take `% count`, you get a negative number, which is a crash waiting to happen if you use it as an index. The safe "always positive" wrap is:

```csharp
int wrapped = ((value % count) + count) % count;
```

### Check yourself

1. In `if (a || b)`, when is `b` *not* evaluated?
2. What's `-1 % 4` in C#? What does `((-1 % 4) + 4) % 4` give?
3. A grade boundary says "50 or more is a pass". Write the condition. What value would you test to catch a `<` vs `<=` mistake?

### Exercises

**A3.1** Write a program that reads a temperature and prints "freezing", "cold", "mild" or "hot" using sensible boundaries you choose. Then list every boundary value and test each one.

**A3.2** A pause menu has 5 options, numbered 0 to 4, with the current selection stored as an int. Pressing **down** moves to the next option and pressing **up** moves to the previous one. Going down from the last option goes back to the first; going up from the first goes to the last. Write the code that updates the selection for each key press, and test both edges.

**A3.3** Rock, paper, scissors against a fixed choice. Use a `switch` expression to decide the winner.

**Stretch:** A3.2 again, but now some options are disabled (greyed out) and must be skipped.

### 🎮 In Godot

GDScript uses `and`, `or`, `not` instead of `&&`, `||`, `!`, and `match` instead of `switch`:

```gdscript
if jump and is_on_floor():
    velocity.y = JUMP_VELOCITY

match state:
    "idle": play_idle()
    "run": play_run()
    _: pass
```

GDScript's `%` *also* gives negative results for negative numbers. Godot provides helpers for the safe wrap so you don't have to remember the formula: `posmod(-1, 5)` and `wrapi(-1, 0, 5)` both give `4`. In C#, it's `Mathf.PosMod(-1, 5)`. (Understand the formula first, *then* use the helper.)

Menu cursors, inventory slots, weapon cycling, looping a carousel of level select screens: all exercise A3.2.

<details>
<summary>Answers</summary>

1. When `a` is true. The whole thing is already true, so C# doesn't bother checking `b`.
2. `-1 % 4` is `-1`. The wrapped version gives `3`.
3. `score >= 50`. Test exactly `50` (should pass) and `49` (should fail).

</details>

---

## A4. Loops

### The idea

```csharp
// for: when you know how many times
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Turn {i + 1}");
}

// while: repeat until a condition changes
int enemyHealth = 50;
while (enemyHealth > 0)
{
    enemyHealth -= 12;
}

// do-while: always runs at least once
string input;
do
{
    Console.Write("Type yes or no: ");
    input = Console.ReadLine();
} while (input != "yes" && input != "no");

// foreach: every item in a collection
string[] party = { "Ash", "Bramble", "Cinder" };
foreach (string member in party)
{
    Console.WriteLine(member);
}
```

`break` leaves a loop immediately. `continue` skips straight to the next iteration.

**Off-by-one errors** are the loop version of A3's boundary problem. `i < 5` runs 5 times (0 to 4). `i <= 5` runs 6 times. When in doubt, trace the first and last iteration by hand.

### Check yourself

1. How many times does `for (int i = 1; i <= 10; i += 3)` run? What values does `i` take?
2. When would you choose `do-while` over `while`?
3. What's the difference between `break` and `continue`?

### Exercises

**A4.1** FizzBuzz, game edition: print 1 to 30, but print "Crit!" for multiples of 3, "Block!" for multiples of 5 and "Combo!" for multiples of both.

**A4.2** Simulate a fight: a player (100 HP) and an enemy (80 HP) take turns hitting each other for fixed amounts. Print each turn and stop the moment someone hits 0. Make sure nobody attacks after they're defeated.

**A4.3** Ask for a number and print the sum of its digits (`4271` → `14`). No converting to a string allowed: use `%` and `/`.

**A4.4** Draw a right-angled triangle of `*` of a given height using nested loops.

### 🎮 In Godot

GDScript loops look like this:

```gdscript
for i in range(5):
    print(i)

for enemy in get_tree().get_nodes_in_group("enemies"):
    enemy.take_damage(10)
```

But here's the big one: **you rarely write the main loop in Godot.** The engine *is* the loop. `_process(delta)` and `_physics_process(delta)` are called every frame for you. If you write `while (true)` inside `_process`, the game freezes, because the frame can never finish.

So the A4.2 fight loop, in a game, wouldn't be a `while` loop at all. Each turn would happen in response to a button press or a timer. Recognising "this is a loop in a console app, but an event in a game" is a key mental shift.

<details>
<summary>Answers</summary>

1. 4 times: `i` is 1, 4, 7, 10.
2. When the body must run at least once, e.g. asking for input before you can check it.
3. `break` exits the loop entirely; `continue` skips the rest of the current iteration and carries on with the next one.

</details>

---

## A5. Arrays

### The idea

An array is a **fixed-size** sequence of values of one type:

```csharp
int[] scores = new int[5];          // five zeros
int[] damage = { 12, 40, 7, 33 };   // filled in directly

Console.WriteLine(damage[0]);       // 12. indexes start at 0
Console.WriteLine(damage.Length);   // 4
Console.WriteLine(damage[damage.Length - 1]);  // 33. the last element

damage[4] = 50;  // IndexOutOfRangeException! valid indexes are 0 to 3
```

Arrays are **reference types** (remember A1). Pass one to somewhere else and change it there, and you've changed the original.

A **2D array** is a grid, perfect for boards and tile maps:

```csharp
char[,] board = new char[3, 3];
board[1, 1] = 'X';
int rows = board.GetLength(0);
int cols = board.GetLength(1);
```

### Check yourself

1. An array has `Length` 8. What's the first valid index? The last?
2. Can you add a 6th item to an array created with `new int[5]`?
3. Why does changing an array inside a method affect the caller's array?

### Exercises

**A5.1** Given an array of enemy health values, print the highest value *and* which index it's at.

**A5.2** Reverse an array **in place** (without creating a second array).

**A5.3** Count how many values in an array are above the average.

**A5.4** Store a 5×5 tile map as `char[,]` (`#` for walls, `.` for floor) and print it as a grid.

### 🧮 Algorithm challenges

These are classics. Write each one yourself, from a blank file. For each, work out roughly how many steps it takes as the array gets bigger (its **Big-O**).

**A5.5 Linear search.** Given an array and a target, return the index of the target, or `-1` if it's not there. Test: target at the start, at the end, in the middle, and missing. Test an empty array.

**A5.6 Searching faster.** You have a **sorted** array of 1,000,000 numbers. Find a target in about 20 steps. (Hint: think about how you look up a word in a paper dictionary.) Test the same edge cases as A5.5, and an array of exactly one element.

**A5.7 Sorting.** Sort an array into ascending order *without* using `Array.Sort`. Then try a second, different method of sorting. Which is faster on an array that's already *almost* sorted? Why?

### 🎮 In Godot

GDScript arrays can grow, unlike C# arrays. They're closer to C#'s `List<T>`, which you'll meet in C1:

```gdscript
var damage: Array[int] = [12, 40, 7]
damage.append(33)
```

In Godot C#, you'll see C# arrays and also `Godot.Collections.Array`, which exists so data can pass between C# and the engine. For your own logic, plain C# arrays and lists are usually best.

Tile maps, inventory grids, puzzle boards and level layouts are all 2D-array-shaped problems. A5.4 is a tiny level editor.

<details>
<summary>Answers</summary>

1. First is `0`, last is `7`.
2. No. Arrays are fixed size. You'd need a new, bigger array (or a `List<T>`, coming in C1).
3. Arrays are reference types. The method receives a copy of the *reference*, which still points at the same array.

</details>

---

## A6. Methods and recursion

### The idea

A method is a named, reusable block of code. It takes **parameters** and can **return** a value:

```csharp
static int CalculateDamage(int attack, int defence)
{
    int damage = attack - defence;
    return damage < 1 ? 1 : damage;  // always at least 1
}

int hit = CalculateDamage(30, 12);  // 18
```

`void` means a method returns nothing. `static` means you call it without needing an object (you'll see the alternative in Block B).

**Overloading:** same name, different parameters. C# picks the right one:

```csharp
static int Heal(int health, int amount) => health + amount;
static int Heal(int health) => health + 10;  // default heal
```

(The `=>` is shorthand for a method body that's a single expression.)

**Optional parameters** give defaults:

```csharp
static void Log(string message, bool shout = false) { ... }
Log("hi");
Log("HI", true);
```

**Scope:** a variable declared inside a method (or inside `{}`) only exists there. Two methods can both have a variable called `i` without interfering.

**Recursion:** a method that calls itself. Every recursive method needs a **base case** (when to stop) and a **recursive case** that moves towards it:

```csharp
static int Factorial(int n)
{
    if (n <= 1) return 1;           // base case
    return n * Factorial(n - 1);    // recursive case, n gets smaller
}
```

Forget the base case and you get a `StackOverflowException`.

### Check yourself

1. What's the difference between a parameter and a return value?
2. What does `void` mean?
3. What two things does every recursive method need?

### Exercises

**A6.1** Write `IsCritical(double roll, double critChance)` returning a `bool`, and `ApplyCrit(int damage)` returning double damage. Use them together with `Random` (`new Random().NextDouble()`) to roll 10 attacks.

**A6.2** Write two overloads of `Clamp`: one for `int`, one for `double`. Each keeps a value between a min and a max. Test with values below, inside, above, and exactly *on* each boundary.

**A6.3** Recursively sum the numbers from 1 to `n`. Then do it with a loop. Which is clearer?

**A6.4** Write a method that recursively counts down from `n` and prints "Fight!" at the end.

### 🧮 Algorithm challenges

**A6.5** Rewrite your fast sorted-array search from A5.6 using recursion instead of a loop.

**A6.6** A **stack** lets you push items on top and pop them off the top: last in, first out (think: undo history). Build one using an `int[]` and a count, with methods `Push`, `Pop` and `Peek`. What should happen if you pop from an empty stack? What if you push onto a full one?

### 🎮 In Godot

You've already been writing methods in Godot! GDScript:

```gdscript
func calculate_damage(attack: int, defence: int) -> int:
    return max(attack - defence, 1)
```

The built-in callbacks (`_ready`, `_process`, `_physics_process`) are methods too. The difference is that **the engine calls them, not you**. In C#, you can see this clearly because you have to write `override`:

```csharp
public override void _PhysicsProcess(double delta)
{
    // Godot calls this every physics tick
}
```

In GDScript, a parameter you don't use (like an unused `delta`) gets an underscore, `_delta`, to tell Godot not to warn you.

Recursion turns up in Godot with the scene tree: a node has children, which have children... A method that visits every node in a tree is naturally recursive.

<details>
<summary>Answers</summary>

1. Parameters are inputs passed *into* a method; the return value is the output sent back *out*.
2. The method doesn't return a value.
3. A base case that stops it, and a recursive case that moves closer to the base case.

</details>

---

## A7. Strings and characters

### The idea

A `string` is a sequence of `char`s, and you can index into it:

```csharp
string word = "Godot";
Console.WriteLine(word[0]);       // G
Console.WriteLine(word.Length);   // 5
```

Strings are **immutable**. Every "change" makes a new string:

```csharp
string s = "hello";
s.ToUpper();          // creates "HELLO" and throws it away!
s = s.ToUpper();      // now s refers to the new string
```

Useful methods: `ToUpper`, `ToLower`, `Trim`, `Contains`, `StartsWith`, `Replace`, `Substring`, `IndexOf`, `Split`, and `string.Join`.

**chars are secretly numbers.** Each character has a numeric code, and you can do arithmetic on it:

```csharp
char c = 'a';
Console.WriteLine((int)c);         // 97
Console.WriteLine((char)(c + 1));  // b
Console.WriteLine('d' - 'a');      // 3. how far into the alphabet 'd' is
Console.WriteLine(char.IsLetter('7'));  // False
```

Building a long string in a loop with `+=` creates a new string every time. For lots of joining, use `StringBuilder`:

```csharp
var sb = new System.Text.StringBuilder();
for (int i = 0; i < 1000; i++) sb.Append(i);
string result = sb.ToString();
```

### Check yourself

1. Why does `name.Trim();` on its own line do nothing useful?
2. What's `'z' - 'a'`?
3. What does `"a,b,c".Split(',')` give you?

### Exercises

**A7.1** Check whether a word reads the same backwards and forwards. Ignore capital letters.

**A7.2** Count the vowels in a sentence.

**A7.3** A spy sends messages where every letter is moved a fixed number of places along the alphabet. With a shift of 3, `a` becomes `d`, and `y` becomes `b` (it wraps round). Capitals stay capitals; spaces and punctuation stay as they are. Write a method that encodes a message, and one that decodes it.

**A7.4** Turn `"the dark forest"` into title case: `"The Dark Forest"`.

### 🎮 In Godot

Text in games is everywhere: dialogue, names, item descriptions, save slot labels. A classic Godot effect is the **typewriter**: dialogue appearing one character at a time. That's A7 indexing plus the A4 "the engine is the loop" idea: reveal one more character each time a timer fires.

In GDScript, strings work the same way: `text[0]`, `text.length()`, `text.to_upper()`. In Godot C#, it's a normal C# `string`, and you set a label's text with:

```csharp
_label.Text = $"HP: {_health}";
```

<details>
<summary>Answers</summary>

1. Strings are immutable. `Trim()` returns a *new* trimmed string, which is thrown away unless you assign it.
2. `25`.
3. An array of three strings: `"a"`, `"b"`, `"c"`.

</details>

---

## A8. Debugging and exceptions

### The idea

**Reading errors is a skill.** When a program crashes, the exception message and **stack trace** tell you what went wrong and *where*. Read the top line (the type of error), then find the first line in the trace that's *your* file. That's usually where to look.

**Use the debugger.** In VS Code or Rider, click in the margin to set a **breakpoint**, run in debug mode, and the program pauses there. You can inspect every variable and step through one line at a time. It's like `Console.WriteLine` debugging, but you can see *everything* without editing your code.

**Exceptions** are how C# signals something went wrong at runtime. You can catch them:

```csharp
try
{
    int age = int.Parse(Console.ReadLine());
    Console.WriteLine($"Next year: {age + 1}");
}
catch (FormatException)
{
    Console.WriteLine("That wasn't a number.");
}
finally
{
    Console.WriteLine("This always runs.");
}
```

But often you don't need to. **Prefer preventing exceptions to catching them.** `TryParse` returns `false` instead of crashing:

```csharp
if (int.TryParse(Console.ReadLine(), out int age))
{
    Console.WriteLine($"Next year: {age + 1}");
}
else
{
    Console.WriteLine("That wasn't a number.");
}
```

You can **throw** your own exceptions when something is genuinely invalid:

```csharp
static void TakeDamage(int amount)
{
    if (amount < 0)
        throw new ArgumentException("Damage can't be negative.");
    // ...
}
```

Rules of thumb: catch **specific** exceptions, not every exception. Never write an empty `catch { }` that hides errors. And don't use exceptions for normal, expected situations like bad user input.

### Check yourself

1. In a stack trace, which line do you usually want to look at first?
2. What's the advantage of `int.TryParse` over `int.Parse`?
3. When does a `finally` block run?

### Exercises

**A8.1** Go back to A1.1 and make it impossible to crash with bad input. Keep asking until you get a valid age between 0 and 130.

**A8.2** This code is meant to find the average of the positive numbers in an array. It has **three bugs**. Find them using the debugger, not by staring. Note down what each one was.

```csharp
int[] values = { 4, -2, 9, 0, 7 };
int total = 0;
int count = 0;

for (int i = 0; i <= values.Length; i++)
{
    if (values[i] >= 0)
    {
        total += values[i];
        count++;
    }
}

double average = total / count;
Console.WriteLine($"Average: {average}");
```

**A8.3** Write `Divide(int a, int b)` that throws an `ArgumentException` with a clear message if `b` is zero. Call it in a `try`/`catch` and print the message.

### 🎮 In Godot

Godot's **Debugger** panel at the bottom of the editor shows errors with the file and line. Click them to jump straight there. You can set breakpoints in the Godot script editor too (click left of the line number, red dot appears).

GDScript doesn't have `try`/`catch`. Instead, Godot code tends to **check before acting**: `if node != null`, `is_instance_valid(node)`, `has_node("Path")`. That "prevent, don't catch" mindset is exactly what this stage teaches.

Also: `print()` (GDScript) and `GD.Print()` (C#) are your quick-and-dirty debug tools, and **warnings** in the editor are free bug reports. Read them.

<details>
<summary>Answers</summary>

1. The first line that points at your own code, rather than at .NET's internals.
2. It doesn't crash on bad input; it returns `false` so you can handle it normally.
3. Always: whether the `try` succeeded, threw an exception that was caught, or threw one that wasn't.

</details>

---

## Block A checkpoint

Do these cold: no notes, no searching, no running code until you've written your answer.

1. `double r = 11 / 4;` What's in `r`, and why?
2. What's `-3 % 5` in C#? Write an expression that wraps index `-3` safely into a list of 5.
3. Two `int` variables vs two array variables: explain the difference in what happens when you copy one into the other and then change the copy.
4. Write a method `CountMatches(string text, char target)` that returns how many times `target` appears.
5. What are the two required parts of a recursive method?
6. Why is `catch (Exception) { }` a bad idea?

**Mini project: Console battle, version 1.** Two fighters with names, health, attack and defence stored in variables or arrays. Each turn, the attacker rolls for a critical hit, damage is calculated by a method, and a neatly formatted line is printed. The battle ends cleanly when one fighter is defeated. All input is validated. No classes yet: you'll rebuild this properly in Block B and feel how much nicer it gets.

<details>
<summary>Checkpoint answers</summary>

1. `2`. Integer division happens first (11 / 4 = 2), then 2 is stored as `2.0`.
2. `-3`. Safe wrap: `((-3 % 5) + 5) % 5`, which is `2`.
3. `int` is a value type, so the copy is independent. Arrays are reference types, so both variables point at the same array and changes show through both.
4. One possible approach: loop over the characters, compare each with `target`, count matches, return the count.
5. A base case and a recursive case that moves towards it.
6. It silently swallows every error, including ones you didn't expect, so bugs hide instead of being fixed.

</details>
