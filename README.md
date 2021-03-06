# cube.js

cube.js is a JavaScript library for modeling and solving the 3x3x3
Rubik's Cube.

Most notably, it implements [Herbert Kociemba's two-phase
algorithm](http://kociemba.org/cube.htm) for solving any state of the
cube very fast in 22 moves or less.

cube.js is written in [CoffeeScript](http://coffeescript.org/), and
runs on [node.js](http://nodejs.org/) and modern browsers.


## Examples

`cube.js` gives you basic cube manipulation:

```javascript
// Create a new solved cube instance
var cube = new Cube();

// Apply an algorithm or randomize the cube state
cube.move("U F R2 B' D2 L'");
cube.randomize();

// Create a new random cube
var randomCube = Cube.random();
```

Include `solve.js` to solve the cube.

```javascript
// This takes 4-5 seconds on a modern computer
Cube.initSolver();

// These typically take from 0.01s to 0.4s, rarely up to 2s
cube.solve();        // => "D2 B' R' B L' B ..."
randomCube.solve();  // => "R B' R U' D' R' ..."
```

To offload the solving to a web worker, use `async.js` and
`worker.js`:

```javascript
Cube.asyncInit('lib/worker.js', function() {
    // Initialized
    Cube.asyncSolve(randomCube, function(algorithm) {
        console.log(algorithm);
    });
});
```

A full-fledged random state scrambler demo is available at
http://www.digip.org/cubejs/demo/.

## API Reference

All functionality is implemented in the `Cube` object. There
are a bunch of files, of which `cube.js` is always required.

### `cube.js`

`cube.js` gives you basic cube manipulation.

#### Class methods

##### `Cube([state])`

The constructor. Without arguments, constructs an identity cube
(i.e. a solved cube). With an argument, clones another cube or
cube state.

```javascript
var cube = new Cube();
var other = new Cube(cube);
var third = new Cube(cube.toJSON());
```

##### `Cube.fromString(str)`

Returns a cube that represents the given facelet string. The string
consists of 54 characters, 9 per face:

```javascript
"UUUUUUUUUR...F...D...L...B..."
```

U means a facelet of the up face color, R means a facelet of the right
face color, etc.

The following diagram demonstrates the order of the facelets:

```
             +------------+
             | U1  U2  U3 |
             |            |
             | U4  U5  U6 |
             |            |
             | U7  U8  U9 |
+------------+------------+------------+------------+
| L1  L2  L3 | F1  F2  F3 | R1  R2  F3 | B1  B2  B3 |
|            |            |            |            |
| L4  L5  L6 | F4  F5  F6 | R4  R5  R6 | B4  B5  B6 |
|            |            |            |            |
| L7  L8  L9 | F7  F8  F9 | R7  R8  R9 | B7  B8  B9 |
+------------+------------+------------+------------+
             | D1  D2  D3 |
             |            |
             | D4  D5  D6 |
             |            |
             | D7  D8  D9 |
             +------------+
```

##### `Cube.random()`

Return a new, randomized cube.

```javascript
var cube = Cube.random();
```

##### `Cube.inverse(algoritm)`

Given an algorithm (a string, array of moves, or a single move),
returns its inverse.

```javascript
Cube.inverse("F B' R");    // => "R' B F'"
Cube.inverse([1, 8, 12]);  // => [14, 6, 1]
Cube.inverse(8);           // => 8
```

See below for numeric moves.

#### Instance methods

##### `init(state)`

Resets the cube state to match another cube.

```javascript
var random = Cube.random();
var cube = new Cube();
cube.init(random);
```

##### `identity()`

Resets the cube to the identity cube.

```javascript
cube.identity();
cube.isSolved();  // => true
```

##### `toJSON()`

Returns the cube state as an object.

```javascript
cube.toJSON();  // => {cp: [...], co: [...], ep: [...], eo: [...]}
```

##### `asString()`

Returns the cube's state as a facelet string. See `Cube.fromString()`.

##### `clone()`

Returns a fresh clone of the cube.

##### `randomize()`

Randomizes the cube in place.

##### `isSolved()`

Returns `true` if the cube is solved (i.e. the identity cube),
and `false` otherwise.

##### `move(algorithm)`

Applies an algorithm (a string, array of moves, or a single move)
to the cube.

```javascript
var cube = new Cube();
cube.isSolved();  // => true
cube.move("U R F'");
cube.isSolved();  // => false
```

See below for numeric moves.

#### Numeric moves

Internally, cube.js treats moves as numbers.

<table>
 <tr><th>Move</th><th>Number</th></tr>
 <tr><td>U</td><td>0</td></tr>
 <tr><td>U2</td><td>1</td></tr>
 <tr><td>U'</td><td>2</td></tr>
 <tr><td>F</td><td>3</td></tr>
 <tr><td>F2</td><td>4</td></tr>
 <tr><td>F'</td><td>5</td></tr>
 <tr><td>L</td><td>6</td></tr>
 <tr><td>L2</td><td>7</td></tr>
 <tr><td>L'</td><td>8</td></tr>
 <tr><td>D</td><td>9</td></tr>
 <tr><td>D2</td><td>10</td></tr>
 <tr><td>D'</td><td>11</td></tr>
 <tr><td>B</td><td>12</td></tr>
 <tr><td>B2</td><td>13</td></tr>
 <tr><td>B'</td><td>14</td></tr>
 <tr><td>R</td><td>15</td></tr>
 <tr><td>R2</td><td>16</td></tr>
 <tr><td>R'</td><td>17</td></tr>
</table>

### `solve.js`

`solve.js` implements [Herbert Kociemba's two-phase
algorithm](http://kociemba.org/cube.htm) for solving the cube quickly
in nearly optimal number of moves. The algorithm is a port of his
simple Java implementation without symmetry reductions.

For the algorithm to work, a computationally intensive precalculation
step is required. Precalculation results in a set of lookup tables
that guide the heuristic tree search. Precalculation takes 4-5 seconds
on a typical modern computer, but migh take longer on older machines.

After precalculation is done, solving a cube with at most 22 moves
typically takes 0.01-0.4 seconds, but may take up to 1.5-2 seconds.
Again, these figures apply for a modern computer, and might be bigger
on older machines.

The maximum search depth (numer of moves) can be configured. The
deeper search is allowed, the quicker the solving is. There's usually
no reason to change the default of 22 moves.

#### Class methods

##### `Cube.initSolver()`

Perform the precalculation step described above. This must be called
before calling `solve()`.

##### `Cube.scramble()`

Generate a random scramble by taking a random cube state, solving it,
and returning the inverse of the solving algorithm. By applying this
algorithm to a cube, you end up to the random state.

#### Instance methods

##### `solve([maxDepth])`

Return an algorithm that solves the cube as a string. `maxDepth` is
the maximum number of moves in the solution, and defaults to 22.


## License

cube.js is licensed under the
[MIT License](http://opensource.org/licenses/MIT).
