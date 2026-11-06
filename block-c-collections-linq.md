# Block C: Collections, Generics, LINQ, Events

Real programs juggle lots of data: a party of characters, an inventory of items, a log of everything that happened in a battle. Block C gives you the proper tools for that: flexible collections, generics so one piece of code works for many types, LINQ for asking questions about data, and events so objects can announce things without knowing who's listening.

By the end, you'll have met every major idea you need to read real-world C#, including the C# that runs Godot games.

**Done when:** every stage's exercises run, and the Block C final project is finished.

---

## C1. Collections and generics

### The idea

Arrays are fixed size. Most of the time you want a **`List<T>`**, which grows and shrinks:

```csharp
var party = new List<Character>();
party.Add(new Character("Ash", 100));
party.Add(new Character("Bramble", 80));

Console.WriteLine(party.Count);   // 2  (Count, not Length!)
Console.WriteLine(party[0].Name); // Ash

party.Remove(party[1]);
party.Insert(0, new Character("Cinder", 60));
bool hasAsh = party.Exists(c => c.Name == "Ash");  // more on => in C2
```

The `<T>` is a **generic type parameter**: "a list of *something*". `List<int>`, `List<string>` and `List<Character>` are all the same code, specialised to hold one type safely. You can't accidentally add a string to a `List<int>`.

A **`Dictionary<TKey, TValue>`** maps keys to values, for looking things up by name or ID:

```csharp
var inventory = new Dictionary<string, int>();
inventory["Potion"] = 3;
inventory["Ether"] = 1;

inventory["Potion"]--;   // use one

// Reading a missing key with [] throws an exception! Check first:
if (inventory.TryGetValue("Phoenix Down", out int count))
{
    Console.WriteLine($"You have {count}.");
}

foreach (var (item, qty) in inventory)
{
    Console.WriteLine($"{item} x{qty}");
}
```

Other built-in collections worth knowing:

| Collection | Use it for |
|---|---|
| `List<T>` | An ordered list you add to and remove from |
| `Dictionary<TKey, TValue>` | Looking things up by a key |
| `HashSet<T>` | A set of unique items: "have we seen this before?" |
| `Queue<T>` | First in, first out (the one you built in B1.6) |
| `Stack<T>` | Last in, first out (the one you built in A6.6) |
| `PriorityQueue<TElement, TPriority>` | Always gives you the lowest-priority-number item next |

**Writing your own generics.** Methods and classes can take type parameters too:

```csharp
static T PickRandom<T>(List<T> items)
{
    int index = Random.Shared.Next(items.Count);
    return items[index];
}

string name = PickRandom(new List<string> { "Ash", "Bramble" });
int roll = PickRandom(new List<int> { 1, 2, 3, 4, 5, 6 });
```

You can **constrain** a generic to types that meet a requirement:

```csharp
static T Strongest<T>(List<T> items) where T : Character
{
    T best = items[0];
    foreach (T item in items)
    {
        if (item.Attack > best.Attack) best = item;  // allowed, because T must be a Character
    }
    return best;
}
```

**How fast are they?** Looking something up in a `Dictionary` by key takes roughly the same time whether it holds 10 items or 10 million. Finding an item in a `List` by checking each one gets slower the bigger the list. Choosing the right collection is one of the most important performance decisions you'll make.

### Check yourself

1. Arrays use `.Length`. What do lists use?
2. What happens if you read `dict["missing"]` for a key that isn't there? What should you use instead?
3. What does the `T` in `List<T>` mean?

### Exercises

**C1.1** Convert your Block B battle to use `List<Character>` for the party and enemies. Defeated enemies should be removed from the list. The battle ends when either list is empty.

**C1.2** Build an `Inventory` class wrapping a `Dictionary<string, int>`. Methods: `Add(item, qty)`, `Use(item)` (returns `false` if you have none), `Has(item)`, and `Print()`. Items with a quantity of 0 should disappear from the inventory.

**C1.3** Turn order: each character has a `Speed`. Each round, the fastest acts first. Use a `PriorityQueue` to decide who goes next. (Careful: it gives you the *lowest* priority first, so how do you make the fastest come out first?)

**C1.4** Write a generic `Pair<T>` class that holds two values of the same type, with a `Swap()` method. Test it with `int`, `string` and `Character`.

**C1.5** A player can only discover each secret area once. Track discovered areas so that discovering the same one twice doesn't give a second reward. Which collection fits best?

### 🧮 Algorithm challenges

**C1.6 Build your own hash map.** Make a `SimpleMap` class that stores string keys and int values using an array of "buckets" (each bucket is a `List` of key/value pairs). Use `key.GetHashCode()` and `%` to pick a bucket (watch out: hash codes can be negative, remember A3!). Implement `Set`, `Get` and `ContainsKey`. Then answer: what happens to the speed if every key lands in the same bucket? Why is a good hash function important? Now you know what `Dictionary` is doing under the hood.

**C1.7 Graphs.** A dungeon is a set of rooms connected by corridors. Store it as a `Dictionary<string, List<string>>` (each room maps to the rooms it connects to). Then write two methods: one that explores **level by level** using a `Queue` (all rooms one step away, then all rooms two steps away...) and one that explores **as deep as possible first** using a `Stack` or recursion. Use the first one to find the fewest corridors between the entrance and the treasure room.

### 🎮 In Godot

GDScript has `Array` and `Dictionary` built in, and you can type them:

```gdscript
var party: Array[Character] = []
var inventory: Dictionary = {"Potion": 3, "Ether": 1}

party.append(Character.new())
if inventory.has("Potion"):
    inventory["Potion"] -= 1
```

In Godot C#, you get **two** sets of collections:

- `System.Collections.Generic` (`List<T>`, `Dictionary<K,V>`): normal C#. Use these for your game logic.
- `Godot.Collections` (`Godot.Collections.Array`, `Godot.Collections.Dictionary`): these can cross into the engine, e.g. to be `[Export]`ed to the Inspector or passed to GDScript.

A good habit: keep game logic in plain C# collections, and only convert at the edge where you hand data to the engine.

Godot's `get_children()` and `get_tree().get_nodes_in_group("enemies")` hand you collections of nodes, which you'll loop over, filter and search all the time.

And C1.7 is genuinely how enemy pathfinding starts. Godot's built-in `AStar2D` is a fancier version of the level-by-level search you just wrote.

<details>
<summary>Answers</summary>

1. `.Count`.
2. It throws a `KeyNotFoundException`. Use `TryGetValue` (or `ContainsKey` first).
3. It's a placeholder for a type. `List<int>` fills it in with `int`, so the list can only hold ints.

</details>

---

## C2. Delegates, lambdas and LINQ

### The idea

**A delegate is a variable that holds a method.** You can pass behaviour around just like data. The built-in delegate types you'll use most:

- `Func<int, int>`: takes an `int`, returns an `int`. (The last type is always the return type.)
- `Func<Character, bool>`: takes a `Character`, returns a `bool`.
- `Action<string>`: takes a `string`, returns nothing.

```csharp
static int Double(int x) => x * 2;

Func<int, int> transform = Double;
Console.WriteLine(transform(5));  // 10
```

A **lambda** is a quick, unnamed method written inline. `x => x * 2` means "take `x`, give back `x * 2`":

```csharp
Func<int, int> triple = x => x * 3;
Func<int, int, int> add = (a, b) => a + b;
Action<string> shout = msg => Console.WriteLine(msg.ToUpper());

Func<Character, bool> isLow = c => c.Health < c.MaxHealth / 4;
```

Why bother? Because now you can write methods that take **behaviour as a parameter**:

```csharp
static List<Character> Filter(List<Character> people, Func<Character, bool> test)
{
    var result = new List<Character>();
    foreach (var c in people)
        if (test(c)) result.Add(c);
    return result;
}

var injured = Filter(party, c => c.Health < c.MaxHealth);
var alive   = Filter(party, c => c.IsAlive);
```

One `Filter` method, infinite possible questions.

**LINQ** is a whole library of methods like that, built into C#. Add `using System.Linq;` (usually automatic) and every collection gets them:

```csharp
var alive        = party.Where(c => c.IsAlive);
var names        = party.Select(c => c.Name);
var byHealth     = party.OrderBy(c => c.Health);
var weakest      = party.OrderBy(c => c.Health).First();
int totalHealth  = party.Sum(c => c.Health);
int highestAtk   = party.Max(c => c.Attack);
bool anyDown     = party.Any(c => !c.IsAlive);
bool allDown     = party.All(c => !c.IsAlive);
int poisoned     = party.Count(c => c.Status is Poison);
```

Chain them together to ask complex questions in one readable line:

```csharp
var targets = enemies
    .Where(e => e.IsAlive)
    .OrderBy(e => e.Health)
    .Take(3)
    .Select(e => e.Name)
    .ToList();
// the names of the three weakest living enemies
```

Two powerful ones for **grouping and flattening**:

```csharp
// GroupBy: bundle items that share a key
var byType = loot.GroupBy(item => item.Type);
foreach (var group in byType)
{
    Console.WriteLine($"{group.Key}: {group.Count()} items, worth {group.Sum(i => i.Value)}");
}

// SelectMany: flatten a list of lists into one list
var allAbilities = party.SelectMany(c => c.Abilities);
```

**Deferred execution.** Most LINQ methods don't run straight away. They build a *recipe* that runs when you actually loop over the result or call something like `.ToList()`, `.Count()` or `.First()`. If the underlying list changes in between, you get the new data. Call `.ToList()` when you want a snapshot.

### Check yourself

1. What does `Func<string, int>` take and return?
2. What's the difference between `Where` and `Select`?
3. What does `.ToList()` force to happen?

### Exercises

**C2.1** Without using LINQ, write your own `MyWhere`, `MySelect` and `MyCount` methods that take a `List<T>` and a delegate. Test them on your party. (This shows you there's no magic in LINQ.)

**C2.2** Using LINQ, answer these about your party and enemies, one line each:
- The names of all living characters, alphabetically.
- The average health of the enemies.
- The character with the highest attack.
- Whether every enemy is weak to fire.
- How many characters are below half health.

**C2.3** Smart enemy AI: write a targeting method that takes a `Func<Character, int>` "scoring" lambda and attacks whichever party member scores highest. Create three different AI personalities just by passing different lambdas (e.g. "attack the weakest", "attack the strongest", "attack whoever is poisoned").

**C2.4** A shop has a list of sales. Each sale records an item name, a category (weapon, armour, potion) and a price. Each customer also has a list of their own past purchases. Find:
- The total money made per category, biggest first.
- A single list of every item bought by every customer.
- The category of the single most expensive item ever sold.

**C2.5** Go back to your B3 battle log. Use LINQ to print end-of-battle stats: total damage dealt by each character, the biggest single hit, and how many crits there were.

### 🎮 In Godot

GDScript has lambdas and the same "pass behaviour around" style, built into arrays:

```gdscript
var alive = party.filter(func(c): return c.health > 0)
var names = party.map(func(c): return c.name)
var total = party.reduce(func(acc, c): return acc + c.health, 0)
```

That's `Where`, `Select` and `Sum` with different names. In GDScript, a function stored in a variable is called a **Callable**, which is Godot's version of a delegate.

In Godot C#, all of LINQ works on your normal C# collections. A really common pattern:

```csharp
var nearestEnemy = GetTree().GetNodesInGroup("enemies")
    .OfType<Node2D>()
    .OrderBy(e => e.GlobalPosition.DistanceTo(GlobalPosition))
    .FirstOrDefault();
```

"Get every enemy, keep the ones that are 2D nodes, sort by distance to me, take the closest (or null if there are none)." One readable statement, and exactly the kind of thing that powers turret targeting, lock-on cameras and homing projectiles.

One caution: LINQ creates small temporary objects. In code that runs every single frame for hundreds of objects, a plain loop can be kinder to performance. For most game code, readability wins.

<details>
<summary>Answers</summary>

1. Takes a `string`, returns an `int`.
2. `Where` keeps or drops items (filtering). `Select` transforms each item into something else (mapping).
3. It makes the query run immediately and stores the results in a new list, a snapshot that won't change if the original does.

</details>

---

## C3. Events

### The idea

So far, if something happens to a character (it takes damage, it dies), any code that cares has to keep *asking*: "are you dead yet? are you dead yet?". That's messy. **Events** flip it round: the character *announces* what happened, and anything interested *listens*.

```csharp
public class Character
{
    public event Action<Character, int>? Damaged;  // who, how much
    public event Action<Character>? Died;

    public void TakeDamage(int amount)
    {
        Health = Math.Max(Health - amount, 0);
        Damaged?.Invoke(this, amount);    // announce it (?. = only if anyone's listening)

        if (Health == 0)
            Died?.Invoke(this);
    }
}
```

Other code **subscribes** with `+=` and **unsubscribes** with `-=`:

```csharp
var ash = new Character("Ash", 100);

ash.Damaged += (who, amount) => Console.WriteLine($"{who.Name} took {amount}!");
ash.Died += who => Console.WriteLine($"{who.Name} has fallen...");

ash.TakeDamage(40);   // prints the damage line
ash.TakeDamage(100);  // prints the damage line AND the death line
```

The big win is **decoupling**. `Character` doesn't know that a combat log exists. Or a sound system. Or an achievement tracker. Each of those subscribes on its own, and you can add or remove them without touching `Character` at all.

Why the `event` keyword, rather than a public `Action` field? Because `event` means **only the class that owns it can trigger it**. Outside code can subscribe and unsubscribe, but can't fake a `Died` announcement or wipe out everyone else's subscriptions with `=`.

**Remember to unsubscribe** when a listener is finished with. If a long-lived object keeps a subscription to something, it stays alive in memory and keeps reacting, even when you think it's gone.

### Check yourself

1. What's the difference between `+=` and `=` on an event?
2. Why use `?.Invoke` instead of just calling the event?
3. Why is it good that `Character` doesn't know about the combat log?

### Exercises

**C3.1** Add `Damaged`, `Healed` and `Died` events to your `Character`. Create a separate `CombatLog` class that subscribes to every character in the battle and records messages, then prints the full log at the end.

**C3.2** Create an `AchievementTracker` class that listens to the same events and unlocks achievements: "First Blood" (first enemy defeated), "Overkill" (a hit for over 50 damage), "Close Call" (a party member survives on under 10% health). Don't change `Character` to make this work.

**C3.3** Add a `StatusApplied` event. Make a `Boss` that listens to its *own* `Damaged` event and enrages (attack goes up) the first time its health drops below half.

**C3.4** Make sure that when an enemy dies and is removed from the battle, everything that was listening to it unsubscribes.

### 🎮 In Godot

**Events are signals.** Everything in this stage is the idea behind Godot's signals, which you've probably already used when you connected a button's `pressed` signal.

In GDScript:

```gdscript
signal health_changed(new_health: int)
signal died

func take_damage(amount: int) -> void:
    health = max(health - amount, 0)
    health_changed.emit(health)
    if health == 0:
        died.emit()

# somewhere else, e.g. the health bar UI:
player.health_changed.connect(_on_health_changed)
```

In Godot C#, a signal is declared as a delegate with the `[Signal]` attribute. The name **must** end in `EventHandler`:

```csharp
[Signal]
public delegate void HealthChangedEventHandler(int newHealth);

public void TakeDamage(int amount)
{
    Health = Mathf.Max(Health - amount, 0);
    EmitSignal(SignalName.HealthChanged, Health);
}

// connecting from C# looks exactly like a normal C# event:
player.HealthChanged += OnHealthChanged;
```

That's the same `+=` you learned in this stage. Godot signals and C# events are the same pattern; Godot just adds the ability to connect them in the editor and across GDScript and C#.

The golden rule Godot developers live by is **"call down, signal up"**. A parent node can call methods on its children directly (it owns them). A child should *never* reach up and grab its parent; instead it emits a signal, and whoever cares connects to it. Your health bar listens to the player; the player doesn't know the health bar exists. That's C3.1, at the scale of a whole game.

<details>
<summary>Answers</summary>

1. `+=` adds your handler alongside everyone else's. `=` would replace all of them, which is why `event` forbids it from outside the class.
2. If nobody has subscribed, the event is `null`, and calling it directly would crash.
3. Each system can be changed, added or removed independently. `Character` stays small and focused, and you can reuse it in a different game with a different UI.

</details>

---

## Block C final project: The battle, finished

Bring everything together into a complete console battle game:

- **Classes and inheritance:** characters, enemies, a boss, abilities via an interface, status effects via an abstract class.
- **Enums and records:** damage types and weaknesses; damage results as records.
- **Collections:** the party and enemies as lists, an inventory as a dictionary, turn order via a priority queue.
- **LINQ:** enemy AI targeting, end-of-battle statistics.
- **Events:** a combat log, an achievement tracker, and a boss that reacts to its own events.
- **Robustness:** no input can crash it.

Keep your **game rules** in plain classes that have nothing to do with the console. Only one small part of your program should use `Console.ReadLine` and `Console.WriteLine`.

### Why that last rule matters: the Godot port

If you followed it, your `Character`, abilities, status effects and inventory don't care whether they're running in a console or a game engine. So:

**Bonus:** create a Godot C# project, copy your rules classes in unchanged, and build a simple battle screen on top: a `Label` per character showing their health, a few `Button`s for abilities, and your events driving the UI updates. The engine code should be the thin layer on top. Your C# is the game.

That's the whole point of learning the language properly first. You're not learning "Godot code" and "console code". You're learning C#, and Godot is just one more place to run it.
