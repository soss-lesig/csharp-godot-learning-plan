# Block B: Object-Oriented C#

This is the heart of C#. Until now, your battle data has been loose variables and parallel arrays: a `names` array, a `healths` array, an `attacks` array, all kept in step by index. It works, but it's fragile. Block B is where you bundle data and the code that acts on it into **objects**, and it's exactly how Godot itself is built.

**Done when:** every stage's exercises run, you can explain *why* you designed your classes the way you did, and you've rebuilt the Block A battle using classes.

---

## B1. Classes, objects, properties, constructors, encapsulation

### The idea

A **class** is a blueprint. An **object** is a thing built from that blueprint:

```csharp
public class Character
{
    // Properties: the object's data
    public string Name { get; }
    public int MaxHealth { get; }
    public int Health { get; private set; }

    // Constructor: runs when you create the object
    public Character(string name, int maxHealth)
    {
        Name = name;
        MaxHealth = maxHealth;
        Health = maxHealth;
    }

    // Methods: the object's behaviour
    public void TakeDamage(int amount)
    {
        Health -= amount;
        if (Health < 0) Health = 0;
    }

    public bool IsAlive => Health > 0;
}
```

```csharp
var ash = new Character("Ash", 100);
var bramble = new Character("Bramble", 80);

ash.TakeDamage(30);
Console.WriteLine($"{ash.Name}: {ash.Health}/{ash.MaxHealth}");  // Ash: 70/100
Console.WriteLine(bramble.Health);  // 80. each object has its own data
```

**Encapsulation** is the big idea. Look at `Health { get; private set; }`: anyone can *read* health, but only the class itself can *change* it. So nothing outside can do `ash.Health = -500`. The only way to lower health is through `TakeDamage`, which enforces the rules. The class protects its own data from being put into an invalid state.

Access modifiers:

- `public`: anyone can use it.
- `private`: only code inside this class (the default if you write nothing).
- `protected`: this class and classes that inherit from it (B2).

**Fields vs properties.** A field is a raw variable inside a class. A property looks like a field from outside but can control access and run code:

```csharp
private int _gold;                 // field (private, underscore by convention)

public int Gold                    // property wrapping it
{
    get => _gold;
    set => _gold = value < 0 ? 0 : value;  // never negative
}
```

Rule of thumb: fields are `private`; anything other classes need goes through a property or method.

**`static` members** belong to the class itself, not to any one object:

```csharp
public class Character
{
    public static int TotalCreated { get; private set; }
    // in the constructor: TotalCreated++;
}

Console.WriteLine(Character.TotalCreated);
```

**Classes are reference types** (A1 again!). `var a = new Character(...); var b = a;` gives you one character with two names, not two characters.

### Check yourself

1. What's the difference between a class and an object?
2. Why make `Health` have a `private set`?
3. If `b = a` where `a` is a `Character`, and you call `b.TakeDamage(10)`, does `a.Health` change?

### Exercises

**B1.1** Build the `Character` class above, then extend it:
- `Heal(int amount)`, which can never raise health above `MaxHealth`.
- `Attack` and `Defence` properties set by the constructor.
- `TakeDamage` should reduce incoming damage by `Defence`, but always deal at least 1.
- A `ToString()` override that returns something like `"Ash (70/100 HP)"`.

Test the boundaries: heal at full health, damage that exactly equals remaining health, damage less than defence.

**B1.2** Rebuild your Block A battle using two `Character` objects. Notice how much simpler the main program becomes.

**B1.3** A school library needs a program to track books. Each book has a title and author, and can be on loan to one student at a time, or on the shelf. A book can't be lent out if it's already on loan, and can't be returned if it isn't. Design and write the class, then test the rules. Before coding, write down: what's public, what's private, and why?

### 🧮 Algorithm challenges

**B1.4 Linked list.** Build a `Node` class that holds an `int` and a reference to the next `Node` (or `null`). Then a `LinkedList` class with `AddFirst`, `AddLast`, `Contains` and `Print`. This only works *because* classes are reference types: each node points at the next. How many steps does `AddFirst` take? `AddLast`? Can you make `AddLast` faster?

**B1.5** Rebuild your A6.6 stack as a proper `Stack` class, with the array and count `private`.

**B1.6 Queue.** A queue is first in, first out (like a line at a shop). Build one using a fixed-size array where the front and back positions *wrap around* when they reach the end (sound familiar from A3?). Methods: `Enqueue`, `Dequeue`, `Peek`, `IsEmpty`, `IsFull`.

### 🎮 In Godot

**Every script you write in Godot is a class.** In GDScript, the file is the class, and `extends` says what it inherits from:

```gdscript
class_name Player
extends CharacterBody2D

var health: int = 100

func take_damage(amount: int) -> void:
    health = max(health - amount, 0)
```

In Godot C#, it's explicit:

```csharp
using Godot;

public partial class Player : CharacterBody2D
{
    public int Health { get; private set; } = 100;

    public void TakeDamage(int amount)
    {
        Health = Mathf.Max(Health - amount, 0);
    }
}
```

When you drag a scene into the level, Godot is calling `new` for you: each enemy in your level is a separate **object** made from the same **class**.

GDScript has no `private` keyword. The leading underscore (`_health`) is a *convention* meaning "please don't touch this from outside", but nothing enforces it. C# actually enforces it. That's encapsulation with teeth.

**Don't put game rules only in scene scripts.** A plain `Character` class like B1.1, with no Godot code in it, can be tested in the console, then used *by* a Godot node. Keeping your rules separate from your engine code is how real games stay manageable.

<details>
<summary>Answers</summary>

1. A class is the blueprint; an object is one specific thing built from it, with its own data.
2. So only the class can change it, through methods that enforce the rules (never below 0, never above max).
3. Yes. `a` and `b` refer to the same object.

</details>

---

## B2. Inheritance, interfaces, polymorphism, abstraction

### The idea

**Inheritance** lets one class build on another. The child gets everything the parent has, and can add or change things:

```csharp
public class Enemy : Character
{
    public int XpReward { get; }

    public Enemy(string name, int maxHealth, int xpReward)
        : base(name, maxHealth)   // call the parent constructor
    {
        XpReward = xpReward;
    }
}
```

To let a child **change** a parent's behaviour, the parent marks a method `virtual` and the child uses `override`:

```csharp
public class Character
{
    public virtual void TakeDamage(int amount) { /* normal damage */ }
}

public class ArmouredKnight : Character
{
    public override void TakeDamage(int amount)
    {
        base.TakeDamage(amount / 2);  // takes half damage, then uses the normal logic
    }
}
```

**Abstract classes** are blueprints that are deliberately incomplete. You can't create one directly, and they can force children to fill in the gaps:

```csharp
public abstract class StatusEffect
{
    public int TurnsLeft { get; protected set; }

    protected StatusEffect(int turns) { TurnsLeft = turns; }

    public abstract void Tick(Character target);  // every child MUST write this

    public bool HasExpired => TurnsLeft <= 0;
}

public class Poison : StatusEffect
{
    public Poison(int turns) : base(turns) { }

    public override void Tick(Character target)
    {
        target.TakeDamage(5);
        TurnsLeft--;
    }
}
```

**Interfaces** are pure contracts: "anything that implements me promises to have these members". No data, no constructor:

```csharp
public interface IAbility
{
    string Name { get; }
    void Use(Character user, Character target);
}

public class Fireball : IAbility
{
    public string Name => "Fireball";
    public void Use(Character user, Character target) => target.TakeDamage(25);
}

public class Heal : IAbility
{
    public string Name => "Heal";
    public void Use(Character user, Character target) => user.Heal(20);
}
```

**Polymorphism** is the payoff. Code can work with the *general* type and let each object behave in its own way:

```csharp
IAbility[] abilities = { new Fireball(), new Heal() };

foreach (IAbility ability in abilities)
{
    ability.Use(ash, bramble);  // each one does its own thing
}
```

The loop doesn't know or care what kind of ability it's holding. Add a `Lightning` class next week and this loop works with it unchanged.

**Abstract class or interface?**
- Abstract class: related things that share **data and code** ("is a kind of"). A `Poison` *is a* `StatusEffect`.
- Interface: unrelated things that share **a capability** ("can do"). A chest, a door and an NPC can all be `IInteractable` without being related at all.
- A class can only inherit from **one** class, but can implement **many** interfaces.

You can check what something really is with pattern matching:

```csharp
if (character is Enemy enemy)
{
    Console.WriteLine($"Defeated! +{enemy.XpReward} XP");
}
```

### Check yourself

1. What do `virtual` and `override` do?
2. Why can't you write `new StatusEffect(3)`?
3. A barrel, a treasure chest and a shopkeeper can all be "interacted with". Interface or abstract base class? Why?

### Exercises

**B2.1** Give your `Character` an array of `IAbility`. Write at least three abilities, including one that affects the user rather than the target. In the battle, let the player pick an ability by number each turn.

**B2.2** Add status effects using the abstract `StatusEffect` class: `Poison` (damage each turn), `Regeneration` (heal each turn) and one of your own design. Each character can have one active status effect, ticked at the start of their turn and removed when it expires.

**B2.3** Create an `Enemy` subclass and a `Boss` subclass of `Enemy`. The boss should override something to behave differently (e.g. it gets angrier below half health). Use `is` to award XP when an enemy is defeated.

**B2.4** Design exercise, on paper first: a game has doors, levers, chests and NPCs. Players can interact with all of them. Some can be locked. Some can be damaged. Sketch which classes and interfaces you'd use, then write them with simple `Console.WriteLine` behaviour.

### 🎮 In Godot

Godot is built *entirely* on inheritance. Look at the inheritance chain at the top of any node's docs:

```
CharacterBody2D → PhysicsBody2D → CollisionObject2D → Node2D → CanvasItem → Node → Object
```

When your script says `extends CharacterBody2D`, it inherits *everything* up that chain: position, collision, `move_and_slide()`, the lot.

And `_ready`, `_process` and `_physics_process` are **virtual methods**. `Node` defines empty versions; you override them, and the engine calls yours. In GDScript it happens silently. In C#, you can see it:

```csharp
public override void _Ready()
{
    GD.Print("I'm in the scene!");
}
```

GDScript has no `interface` keyword, so Godot projects often fake one with `has_method("interact")` or groups. Godot C# lets you use real interfaces:

```csharp
public interface IInteractable
{
    void Interact(Player player);
}

public partial class Chest : Area2D, IInteractable
{
    public void Interact(Player player) => GD.Print("Opened!");
}

// elsewhere, when the player presses interact next to some node:
if (node is IInteractable thing)
{
    thing.Interact(this);
}
```

That's B2.4, running in a real game.

<details>
<summary>Answers</summary>

1. `virtual` lets a parent method be replaced; `override` is how the child replaces it.
2. It's abstract: deliberately incomplete (its `Tick` has no body), so it can only be used through a child class.
3. An interface. They're unrelated things that share a capability, not kinds of the same thing. And they might each need to inherit from something else already.

</details>

---

## B3. Type system: structs, enums, records, tuples

### The idea

**Enums** give names to a fixed set of options. Much safer than magic strings or numbers:

```csharp
public enum DamageType { Physical, Fire, Ice, Poison }

DamageType type = DamageType.Fire;

string message = type switch
{
    DamageType.Fire   => "It burns!",
    DamageType.Ice    => "Frozen solid!",
    DamageType.Poison => "Feeling sick...",
    _                 => "Ouch."
};
```

Typo `"Frie"` in a string and the bug hides until runtime. Typo `DamageType.Frie` and it won't even compile.

**Structs** are like classes but are **value types** (A1 again, for the last time... probably). Copying a struct copies all its data:

```csharp
public struct Point
{
    public int X;
    public int Y;
    public Point(int x, int y) { X = x; Y = y; }
}

Point a = new Point(1, 2);
Point b = a;   // full copy
b.X = 99;
Console.WriteLine(a.X);  // 1. a is unaffected
```

Use structs for **small, simple values** that are naturally copied: coordinates, colours, a dice roll result. Use classes for **things with identity** that you share: a character, an enemy, an inventory.

**Records** are made for data. One line gives you a type with properties, a constructor, a nice `ToString()`, and **value equality**:

```csharp
public record DamageResult(int Amount, DamageType Type, bool WasCritical);

var r1 = new DamageResult(30, DamageType.Fire, true);
var r2 = new DamageResult(30, DamageType.Fire, true);

Console.WriteLine(r1 == r2);  // True! same data = equal
Console.WriteLine(r1);        // DamageResult { Amount = 30, Type = Fire, WasCritical = True }

var r3 = r1 with { WasCritical = false };  // copy with one change
```

Compare: two *class* objects with identical data are **not** equal with `==`, because `==` checks whether they're the *same object*. Records are also immutable by default: perfect for "this happened" information like a damage report or a log entry.

**Tuples** bundle a few values together without making a whole type. Handy for returning more than one thing:

```csharp
static (int min, int max) Range(int[] values)
{
    int min = values[0], max = values[0];
    foreach (int v in values)
    {
        if (v < min) min = v;
        if (v > max) max = v;
    }
    return (min, max);
}

var (low, high) = Range(new[] { 4, 9, 1, 7 });  // "deconstruct" into two variables
```

If a tuple starts appearing all over your code, it probably wants to be a record.

### Check yourself

1. You copy a struct into a new variable and change the copy. Does the original change? What about a class?
2. Two different `record` objects have the same values. Are they `==`? What about two class objects?
3. When would you choose an enum over a string?

### Exercises

**B3.1** Add a `DamageType` enum and give each character a weakness and a resistance. Weak = double damage, resistant = half. Use a switch expression.

**B3.2** Make `TakeDamage` return a `DamageResult` record (amount dealt, type, crit, whether it was super-effective). Use it to print battle messages, and keep every result in an array as a battle log.

**B3.3** Write a `Point` struct and a `PointClass` class with identical fields. Write a test program that shows, side by side, every way they behave differently (assigning, passing to a method, comparing with `==`).

**B3.4** Write a method that returns the strongest and weakest characters in a party as a tuple.

### 🧮 Algorithm challenge

**B3.5 Binary search tree.** A BST is made of nodes, each with a value and up to two children: everything smaller goes left, everything bigger goes right. Build one with `Insert` and `Contains`. Then write three methods that visit every node in different orders: left-node-right, node-left-right, and left-right-node. Which one prints the values in sorted order? Why? How does the shape of the tree depend on the *order* you insert values in?

### 🎮 In Godot

**`Vector2` is a struct.** This is the explanation promised back in A1. In C#, `Velocity` is a property that hands you a *copy*, and changing `.Y` on a copy would do nothing:

```csharp
Velocity.Y = -300f;   // error: you'd be changing a temporary copy

Vector2 v = Velocity; // copy it out
v.Y = -300f;          // change your copy
Velocity = v;         // put the whole thing back
```

GDScript hides this from you by doing the copy-back automatically. In C#, now you know *why* it can't.

Enums work beautifully in Godot, and `[Export]` makes them a dropdown in the Inspector:

```gdscript
enum DamageType { PHYSICAL, FIRE, ICE }
@export var weakness: DamageType
```

```csharp
public enum DamageType { Physical, Fire, Ice }

[Export] public DamageType Weakness { get; set; }
```

That means designers can set an enemy's weakness by clicking, without touching code.

Enums are also the foundation of **state machines**: `enum State { Idle, Run, Jump, Fall }`, then a `match`/`switch` on the current state in `_physics_process`. That one pattern runs the movement of a huge number of 2D games.

<details>
<summary>Answers</summary>

1. Struct: no, the copy is independent. Class: yes, both variables refer to the same object.
2. Records: yes, they compare by value. Classes: no (by default), they compare by identity: are these the same object?
3. When there's a fixed set of options. The compiler catches typos, and your editor can autocomplete the options.

</details>

---

## Block B checkpoint

Cold, no notes:

1. Explain encapsulation in one sentence, using a health bar as the example.
2. What's the difference between `virtual` and `abstract`?
3. A class can implement how many interfaces? Inherit from how many classes?
4. Write a `Coin` record with a `Value` and a `Colour`, and show the `with` expression changing only its colour.
5. Why does `Velocity.Y = 5f;` fail in Godot C#?
6. Give an example of polymorphism from your own code this block.

**Mini project: Console battle, version 2.** A party of characters versus a group of enemies, all as objects. Abilities via an interface. Status effects via an abstract class. Damage types via an enum. Damage results as records, stored in a log you can print at the end. Players choose abilities and targets by number. The main program should be short and readable; the classes should do the work.

<details>
<summary>Checkpoint answers</summary>

1. Something like: the class keeps health private and only changes it through methods like `TakeDamage`, so nothing can ever set it to an invalid value.
2. `virtual` provides a default the child *may* override. `abstract` provides no body, so the child *must* override it (and the class itself must be abstract).
3. As many interfaces as you like; only one class.
4. `public record Coin(int Value, string Colour);` then `var gold = new Coin(10, "gold"); var silver = gold with { Colour = "silver" };`
5. `Vector2` is a struct, and `Velocity` returns a copy. Changing a field of a temporary copy would have no effect, so C# refuses.
6. Any case where code calls a method on a general type (like `IAbility` or `StatusEffect`) and each concrete object does its own thing.

</details>
