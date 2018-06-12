# Redaction

To redact sensitive information, supply paths to keys that hold sensitive data 
using the `redact` option:

```js
var pino = require('.')({
  redact: ['key', 'path.to.key', 'stuff.thats[*].secret']
})

pino.info({
  key: 'will be redacted',
  path: {
    to: {key: 'sensitive', another: 'thing'}
  },
  stuff: {
    thats: [
      {secret: 'will be redacted', logme: 'will be logged'},
      {secret: 'as will this', logme: 'as will this'}
    ]
  }
})
```

This will output:

```JSON
{"level":30,"time":1527777350011,"pid":3186,"hostname":"Davids-MacBook-Pro-3.local","key":"[Redacted]","path":{"to":{"key":"[Redacted]","another":"thing"}},"stuff":{"thats":[{"secret":"[Redacted]","logme":"will be logged"},{"secret":"[Redacted]","logme":"as will this"}]},"v":1}
```

The `redact` option can take an array (as shown in the above example) or 
an object. This allows control over *how* information is redacted.

For instance, setting the censor: 

```js
var pino = require('.')({
  redact: {
    paths: ['key', 'path.to.key', 'stuff.thats[*].secret'],
    censor: '**GDPR COMPLIANT**'
  }
})

pino.info({
  key: 'will be redacted',
  path: {
    to: {key: 'sensitive', another: 'thing'}
  },
  stuff: {
    thats: [
      {secret: 'will be redacted', logme: 'will be logged'},
      {secret: 'as will this', logme: 'as will this'}
    ]
  }
})
```

This will output: 

```JSON
{"level":30,"time":1527778563934,"pid":3847,"hostname":"Davids-MacBook-Pro-3.local","key":"**GDPR COMPLIANT**","path":{"to":{"key":"**GDPR COMPLIANT**","another":"thing"}},"stuff":{"thats":[{"secret":"**GDPR COMPLIANT**","logme":"will be logged"},{"secret":"**GDPR COMPLIANT**","logme":"as will this"}]},"v":1}
```

The `redact.remove` option also allows for the key and value to be removed from output:

```js
var pino = require('.')({
  redact: {
    paths: ['key', 'path.to.key', 'stuff.thats[*].secret'],
    remove: true
  }
})

pino.info({
  key: 'will be redacted',
  path: {
    to: {key: 'sensitive', another: 'thing'}
  },
  stuff: {
    thats: [
      {secret: 'will be redacted', logme: 'will be logged'},
      {secret: 'as will this', logme: 'as will this'}
    ]
  }
})
```

This will output

```JSON
{"level":30,"time":1527782356751,"pid":5758,"hostname":"Davids-MacBook-Pro-3.local","path":{"to":{"another":"thing"}},"stuff":{"thats":[{"logme":"will be logged"},{"logme":"as will this"}]},"v":1}
```

See [pino options in API](api.md#pino) for `redact` API details.

## Overhead

Pino's redaction functionality is built on top of [`fast-redact`](http://github.com/davidmarkclements/fast-redact)
which adds about 2% overhead to `JSON.stringify` when using paths without wildcards.

When used with pino logger with a single redacted path, any overhead is within noise - we haven't found
a way to deterministically measure it's effect. This is because its not a bottleneck.

However, wildcard redaction does carry a non-trivial cost relative to explicitly declaring the keys 
(50% in a case where four keys are redacted across two objects). See 
the [`fast-redact` benchmarks](https://github.com/davidmarkclements/fast-redact#benchmarks) for details.

## Safety

The `redact` option is intended as an initialization time configuration option. 
It's extremely important that path strings do not originate from user input.  
The `fast-redact` module uses a VM context to syntax check the paths, user input
should never be combined with such an approach. See the [`fast-redact` Caveat](https://github.com/davidmarkclements/fast-redact#caveat)
and the [`fast-redact` Approach](https://github.com/davidmarkclements/fast-redact#approach) for in-depth information. 

