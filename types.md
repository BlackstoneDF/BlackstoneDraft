# Types
Meet the types
```
var - variable {name: "foo", scope: local}
text - "text"
richText - $"rich text" // Prove me wrong that this is a good way to represent a rich text literal 
number - 123

location - location { x: 1, y: 0, z: 1, pitch: 90, yaw: 0 }
vector - vector { x: 1, y: 1, z: 0 }
potion - potion { type: "Resistance", amplifier: 4, duration: 123 }
sound - sound { type: "Creeper Death", pitch: 1, volume: 1 } 
particle - particle {  }

list<number> - [1, 2, 4]
dictionary<text | number> - {a: "a", b: 3}
```

# init
Not decided because the current solution isn't so good.
Current proposals:
(I will be creating a location at x1 y2 z3 in these examples)

## location(1, 2, 3)
There are two problems with this approach
1. If you want to make a function called location (for some reason) you will not be able to call it. 
   But this can be solved using the `new` keyword but this introduces another hard keyword and 4 keystrokes.
2. When reading this, you need to memorize the positions but this becomes a problem when creating a variable with some optional parameters like particle. 
```rs
// Data for particle
struct Particle {
    particle: ImmutableString,
    amount: u32,
    size: f64,

    x: f64,
    y: f64,
    z: f64,

    vertical_spread: Option<i64>,
    horizontal_spread: Option<i64>,
    rgb: Option<Color>,
    color_variation: Option<i64>,
    size_variation: Option<i64>,
    motion_variation: Option<i64>,
    roll: Option<f64>,
}
```
3. 
## location {x: 1, y: 2, z: 3}
For this solution there is the same problem being that this is the constructor syntax for creating structs which the `new` keyword solves with some tradeoffs. 
And there is the problem that variables are very verbose to declare: `variable {name: "foo", scope: local}`
