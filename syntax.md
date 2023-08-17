# Statements

## Assignment
Example
```ts
let a = 1;
```
Syntax
```ts
{let|alias|sel} {iden} = {expr};
```
## Call
Example
```ts
// Call c from the file a/b/c.bls 
// Which is a chain function that returns a mystery namespace 
// d is called from the namespace returned by c()
a.b.c().d() 
```
Syntax
```ts
:a
{iden} delim by . no before and trailing
{{args}}
if ; then 
  end
else if . then 
  goto a
```
## return, break, continue and end
Example
```ts
return 1; 
break;
continue;
end; // Ends the event thread
```
Syntax
```ts
return (expr);
```
## If
Example
```ts
if event.isCanceled() {}
if 1 == 1 {} // Because 1 == 1 is not an expression, you can technically do `if 1 = 1 {}` but I am not sure if we would want to
``` 
Syntax
```ts
if {functionCall} {body}
```

## Repeat
Example
```ts
forever {}
for i in repeat.sphere(loc(50, 50, 50), 3, 2) {}
for i in 0..10 step 2 {}
for i in [1, 2, 4] {}
for i in {a: 1, b: 3} {}
// Can only accept bindings 
while player.if.is_sneaking() {}
```
Syntax
```ts
forever {body}
for {iden} in {function} {body}
for {iden} in {num}..{num} {optional {num}} {body}
for {iden} in {expr} {}
while {function}
```


# Bindings

## Action 
For tags, the rule is everything that is not specified in the action dump like  
1. Tag identifiers (ex: "Align Mode" to align)
2. Tag name identifiers (ex: "Regular" to regular)
but wont include information like 
1. Default tags
2. Icon data

Example
```ts
bind action sendMessage[sel](msg: text) <
  // align has no quotes because it has to be a valid identifier
  ("Align Mode": align) regular { 
    "Regular": regular,
    "Centered": centered, 
  },
  ("Text Value Merging": "merge") none {
    "Add Spaces": spaces,
    "No Spaces": none
  }
> to
  // Using quotes because sometimes they are " Action " for some reason
  player_action "SendMessage" [sel] (msg) <tags>; // tags is a soft keyword

// Due to type safety each GetDirection has to have it's own binding for each return type
bind action getDirection3D(loc: location): "north" | "east" | "south" | "west" | "up" | "down" to
  // Idk if it is "GetDirection" or " GetDirection "
  set_var "GetDirection" (return, loc) <"Return Type": "Text (3D)">;

bind if isSneaking[selection]() to
  if_player "IsSneaking" [sel];

bind repeat sphere(center: location, radius: number, points: number) <
  ("Point Locations Inwards": pointInwards) true {
    "True": true,
    "False": false,
  }
> to
  repeat "Sphere" (changed, center, radius, points) <tags>; // changed is also a soft keyword :(

// The inner constraints are in the action dump but I just cannot be bothered
bind repeat while(inner: if_player, if_entity, if_game, if_var) to 
  repeat "While" (inner);

// TODO: Figure out how to make a literal thing
bind value loc(x: number, y: number, z: number) to
  location 
```
Syntax
```ts
bind action {iden}{optional [{iden}]}({params (has event binding)}) {optional <
  {({string}: {iden}) {iden} {{ 
    {{string}: {iden} delim ,}
  }} delim ,}
>} {optional :{type}} to
  {iden} {string} [{iden}] ({{iden | return} delim ,}) <tags>;
```
## Event
Example
```ts
bind event BreakBlock to player_event "BreakBlock": Cancelable {
  selectors {
    // I do not know what selectors there are for BreakBlock
    // I am in the middle of the forest with no internet
  }
}
bind event_trait Cancelable {}
bind event_trait Cancelable; // Is also fine
// player and entity event Traits are special and get implemented when using a player_event or entity_event
bind event_trait player {
  selectors {
    default: player,
    selection: selection // idk if it is selection or selector
  }
}
bind action cancel_event(event: Cancelable) to 
  game_action "CancelEvent";
```

## definitions.blsd
This file contains what to replace something like a `let`.
It has special syntax to it and a different parser and probably will not have type checking
```ts
// Built ins should'nt have any selectors or event bindings or tags
def assignVariable(name, value) to 
  std.builtin.assignVariable(name, value);
```
And when
```ts
let name = 1;
// or
var("name") = 1;
// or 
alias a = var("name");
a = 1;
```
is typed, it gets turned into
```ts
std.builtin.assignVariable("name", 1);
```

# Items
## Decelerations
```ts
fn add(a: number, b: number): number {
  return add(a, b); // Costs 1 block
}
// Usually wouldnt be allowed because they have the same iden 
// Not sure if it would be possible to allow returns in macros because there is no return function in a macro
macro add(a: number, b: number): number {
  return a + b; // basically `/num %math(%var(a) + %var(b))`
}

bind allMobs() {
  select_object "AllMobs";
}

// In std.selection.new
chain allMobs(): std.selection {
  std.selection.bindings.allMobs();
}

chain filter(inner): std.selection {

}



// I dont like this syntax lol
proc anything(event: player | entity) <
  targets: current | selection | none | forEach default current, // targets: any also works
  variables: none | copy | share default none, 
// Returns are not allowed because no async :)
> {}

proc again(event: player, msg: text) <
  target: selection,
  // variables: none // Not allowed if there are args
  variables: share,
> { 
  player.sendMessage[selection]("${msg}!")
  event.wait(20);
  player.sendMessage("${msg} again!"); // The selection selector is there by default
}
fn call(event: player) {
  add(1, 2);
  anything(targets=none); // Default will take over if you dont specify
  again("Hello", targets=none); // Not allowed because not on list
  again("Hello");
}
```
Syntax
```ts
// Same with `macro`
fn {iden}({params}){optional {: {type}}} {{
  {body}
}} 
// I am not writing the proc syntax because pain
```