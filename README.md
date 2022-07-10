# Function.prototype.once for JavaScript
ECMAScript Stage-1 Proposal. 2022.

Co-champions: Hemanth HM; J. S. Choi.

## Rationale
It is often useful to ensure that callbacks execute only once, no matter how
many times are those callbacks called. To do this, developers frequently use
“once” functions, which wrap around those callbacks and ensure they are called
at most once. This proposal would standardize such a once function in the
language core.

## Description
The Function.prototype.once method would create a new function that calls the
original function at most once, no matter how much the new function is called.
Arguments given in this call are passed to the original function. Any
subsequent calls to the created function would return the result of its first
call.

```js
function f (x) { console.log(x); return x * 2; }

const fOnce = f.once();
fOnce(3); // Prints 3 and returns 6.
fOnce(3); // Does not print anything. Returns 6.
fOnce(2); // Does not print anything. Returns 6.
```

## Real-world examples
The following code was adapted to use this proposal.

From [execa@6.1.0][]:
```js
export function execa (file, args, options) {
  /* … */
  const handlePromise = async () => { /* … */ };
  const handlePromiseOnce = handlePromise.once();
  /* … */
  return mergePromise(spawned, handlePromiseOnce);
});
```

From [glob@7.2.1][]:
```js
function Glob (pattern, options, cb) {
  /* … */
  if (typeof cb === 'function') {
    cb = cb.once();
    this.on('error', cb);
    this.on('end', function (matches) {
      cb(null, matches);
    })
  } /* … */
});
```

From [Meteor@2.6.1][]:
```js
// “Are we running Meteor from a git checkout?”
export const inCheckout = (function () {
  try { /* … */ } catch (e) { console.log(e); }
  return false;
}).once();
```

From [cypress@9.5.2][]:
```js
cy.on('command:retry', (() => { /* … */ }).once());
```

From [jitsi-meet 1.0.5913][]:
```js
this._hangup = (() => {
  sendAnalytics(createToolbarEvent('hangup'));
  /* … */
}).once();
```

[execa@6.1.0]: https://github.com/sindresorhus/execa/blob/v6.1.0/index.js
[glob@7.2.1]: https://github.com/isaacs/node-glob/blob/v7.2.1/glob.js
[Meteor@2.6.1]: https://github.com/meteor/meteor/blob/release/METEOR%402.6.1/tools/fs/files.ts
[cypress@9.5.2]: https://github.com/cypress-io/cypress/blob/v9.5.2/packages/driver/cypress/integration/commands/waiting_spec.js
[jitsi-meet 1.0.5913]: https://github.com/jitsi/jitsi-meet/blob/stable/jitsi-meet_7001/react/features/toolbox/components/HangupButton.js

## Precedents and web compatibility

There is a [popular NPM library called once][NPM once] that allows monkey
patching, which may raise concerns about Function.prototype.once’s web
compatibility.

However, since its first public version, the once library’s monkey patching has
been opt-in only. The monkey patching is not conditional, and there is no actual web-compatibility risk from this library.

```js
// The default form exports a function.
once = require('once');
fOnce = once(f);

// The opt-in form monkey patches Function.prototype.
require('once').proto();
fOnce = f.once();
```

Other popular once functions from libraries (e.g., [lodash.once][], [Underscore][] and [onetime][]) also do not use conditional monkey patching.

A [code search for `!Function.prototype.once`][code search] (as in `if
(!Function.prototype.once) { /* monkey patching */ }`) also gave no results in
any indexed open-source code. It is unlikely that any production code on the
web is conditionally monkey patching a once method into Function.prototype.

[NPM once]: https://www.npmjs.com/package/once
[lodash.once]: https://www.npmjs.com/package/lodash.once
[Underscore]: https://www.npmjs.com/package/underscore
[onetime]: https://www.npmjs.com/package/onetime
[code search]: https://sourcegraph.com/search?q=context:global+%21Function.prototype.once&patternType=literal
