# highs-js

This is a a one-line change to `build.sh` (`-s MODULARIZE -s SINGLE_FILE -s EXPORT_NAME=HiGHS`) to generate a [modularized](https://emscripten.org/docs/compiling/Modularized-Output.html) [single-file](https://emscripten.org/docs/tools_reference/settings_reference.html#single-file) `.js` that has the WASM inside:

```html
<script src="build/highs.js"></script>
```
```js
const highs = await HiGHS();
highs.solve( ... );
```

There is also an [ES6 module](https://emscripten.org/docs/compiling/Modularized-Output.html#generating-es6-modules) version `-o highs.mjs`:

```js
// no script tag, just
import HiGHS from 'build/highs.mjs';
const highs = await HiGHS();
```

---

[![npm version](https://badge.fury.io/js/highs.svg)](https://www.npmjs.com/package/highs)
[![CI status](https://github.com/lovasoa/highs-js/actions/workflows/CI.yml/badge.svg)](https://github.com/lovasoa/highs-js/actions/workflows/CI.yml)
[![package size](https://badgen.net/bundlephobia/minzip/highs)](https://bundlephobia.com/result?p=highs)

This is a javascript mixed integer linear programming library.
It is built by compiling a high-performance C++ solver developed by the University of Edinburgh, ([HiGHS](https://highs.dev)), to WebAssembly using emscripten.

## Demo

See the online demo at: https://lovasoa.github.io/highs-js/

## Usage

```js
const highs_promise = require("highs")();

const PROBLEM = `Maximize
 obj:
    x1 + 2 x2 + 4 x3 + x4
Subject To
 c1: - x1 + x2 + x3 + 10 x4 <= 20
 c2: x1 - 4 x2 + x3 <= 30
 c3: x2 - 0.5 x4 = 0
Bounds
 0 <= x1 <= 40
 2 <= x4 <= 3
End`;

const EXPECTED_SOLUTION = {
  Status: 'Optimal',
  ObjectiveValue: 87.5,
  Columns: {
    x1: {
      Index: 0,
      Status: 'BS',
      Lower: 0,
      Upper: 40,
      Type: 'Continuous',
      Primal: 17.5,
      Dual: -0,
      Name: 'x1'
    },
    x2: {
      Index: 1,
      Status: 'BS',
      Lower: 0,
      Upper: Infinity,
      Type: 'Continuous',
      Primal: 1,
      Dual: -0,
      Name: 'x2'
    },
    x3: {
      Index: 2,
      Status: 'BS',
      Lower: 0,
      Upper: Infinity,
      Type: 'Continuous',
      Primal: 16.5,
      Dual: -0,
      Name: 'x3'
    },
    x4: {
      Index: 3,
      Status: 'LB',
      Lower: 2,
      Upper: 3,
      Type: 'Continuous',
      Primal: 2,
      Dual: -8.75,
      Name: 'x4'
    }
  },
  Rows: [
    {
      Index: 0,
      Name: 'c1',
      Status: 'UB',
      Lower: -Infinity,
      Upper: 20,
      Primal: 20,
      Dual: 1.5
    },
    {
      Index: 1,
      Name: 'c2',
      Status: 'UB',
      Lower: -Infinity,
      Upper: 30,
      Primal: 30,
      Dual: 2.5
    },
    {
      Index: 2,
      Name: 'c3',
      Status: 'FX',
      Lower: 0,
      Upper: 0,
      Primal: 0,
      Dual: 10.5
    }
  ]
};

async function test() {
  const highs = await highs_promise;
  const sol = highs.solve(PROBLEM);
  require("assert").deepEqual(sol, EXPECTED_SOLUTION);
}
```

The problem has to be passed in the [CPLEX .lp file format](http://web.mit.edu/lpsolve/doc/CPLEX-format.htm).

For a more complete example, see the [`demo`](./demo/) folder.

## Passing custom options

HiGHS is configurable through [a large number of options](https://ergo-code.github.io/HiGHS/dev/options/definitions/).

You can pass options as the second parameter to `solve` : 

```js
const highs_promise = require("highs")();
const highs = await highs_promise;
const sol = highs.solve(PROBLEM, {
  "allowed_cost_scale_factor": 2,
  "run_crossover": true,
  "presolve": "on",
});
```
