Linear Matching
===

A [JavaScript proposal](https://github.com/tc39/proposals) to provide RegExp matching capabilities without the risk of catastrophic, unrecoverable failure.

**Stage:** [0](https://tc39.es/process-document)

**Champions:** Michael Ficarra, Aurèle Barrière, Clément Pit-Claudel

## Motivation

Due to the use of a backtracking strategy in the RegExp engines embedded in all modern JavaScript engines, performing a RegExp match can be a dangerous operation. In these engines, depending on the pattern, the match operation may never complete, causing the program to enter an unrecoverable error state. Programs where this condition can be induced through user or environmental inputs are said to be vulnerable to [ReDoS](https://en.wikipedia.org/wiki/ReDoS). ReDoS vulnerabilities have been [highly prevalent in the JavaScript ecosystem for years](https://dl.acm.org/doi/abs/10.1145/3236024.3236027). From January to May 2026, ReDoS vulnerabilities in JavaScript libraries have [resulted in a CVE every 2.7 days](https://www.cve.org/CVERecord/SearchResults?query=ReDoS). Current mitigation strategies are often either too costly, too limiting, or inadequate.

## Presentations to Committee

- [May 2026](https://aurele-barriere.github.io/ecma_amsterdam_handout.pdf)

## Proposal

We should provide solutions for the following use cases:

1. A programmer wants to match using a pattern that was derived from an untrusted source such as user input or a function that generates patterns dynamically.
1. A programmer wants to match using a fixed pattern against untrusted user input.
1. A programmer wants to provide fallback behaviour in the case that a pattern is unable to be matched in a reasonable amount of time for the given input instead of trying to run a match that may never complete.
1. A pattern producer wants to ensure that naïve consumers use a linear matching strategy for that pattern.

## Considered Design Space

As this is an early stage proposal, the design space is still very open. But we have thought through some possible components of a solution that may be proposed at a later stage.

### "linear matching is possible" indicator

```js
if (re.canMatchLinearly) {
  re.exec(...);
} else {
  // fallback behaviour
}
```

After constructing a RegExp, a predicate or other indicator (such as a `RegExp.prototype` getter) can be used to provide fallback behaviour if the engine is unable to match the pattern in linear time.

### linear `exec` variant

```js
let match = re.execLinear(input);
```

A new `RegExp.prototype` method that is like `exec` but opts in to linear matching.

### `l` flag

```js
try {
  let re = /pattern/l;
} catch {
  // fallback behaviour
}
```

A new RegExp `l` flag could be used both to indicate to `exec` that a linear matching strategy is preferred as well as to throw on construction if the pattern cannot be matched linearly by the engine.

### a timeout, input multiplier, or fuel parameter for `exec`

```js
let match = re.exec(input, 10e3);
```

Some way for the programmer to communicate that a backtracking implementation should be used until some kind of resource exhaustion, at which point a linear implementation should be used.

## Prior Art

### other languages

- Ruby: [selective memoisation strategy](https://ieeexplore.ieee.org/document/9519427) and [`Regexp.linear_time?`](https://docs.ruby-lang.org/en/3.2/Regexp.html#method-c-linear_time-3F) predicate
- Rust: [`regex` crate](https://crates.io/crates/regex) omits features with no known linear implementation
- C++ and others: the [RE2](https://github.com/google/re2) library has wrappers available for node.js, OCaml, WebAssembly, etc.
- Go: [`regexp` package](https://pkg.go.dev/regexp) omits features with no known linear implementation

### JS libraries

- [node-re2](https://www.npmjs.com/package/re2): Node.js bindings for [the RE2 linear engine](https://github.com/google/re2)
- [regolith](https://www.npmjs.com/package/regolith): TypeScript/JavaScript server-side lib using [the Rust linear engine](https://docs.rs/regex/latest/regex/)
- [re2js](https://www.npmjs.com/package/re2js): RE2 ported to JavaScript
- [@iter-tools/regex](https://www.npmjs.com/package/@iter-tools/regex) and [@bablr/regex-vm](https://www.npmjs.com/package/@bablr/regex-vm): streaming regex implementation

### V8 experimental engine

- https://v8.dev/blog/non-backtracking-regexp
- V8 flag `--enable-experimental-regexp-engine`
- Incomplete and unmaintained
