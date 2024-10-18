# Apigeelint bug with `@xmldom/xmldom` on v2.57.0

This is a minimal reproduction of a bug with `apigeelint` (v2.57.0) and its resolution of `@xmldom/xmldom`.
The original bug was introduced in v2.56.0, and a fix was released in v2.57.0, but it wasn't a complete fix.

In the `sample` directory, we can see that it doesn't have a correct definition of an Apigee proxy, as the `proxies`
directory is missing.

## Steps to reproduce

1. Run `yarn install` or `npm install`
2. Run `apigeelint -s sample/apiproxy` (or `npx apigeelint -s sample/apiproxy` or `yarn apigeelint -s sample/apiproxy`)
3. Notice that it fails to run with the following output:
    ```
    node:internal/modules/cjs/loader:1145
      throw err;
      ^

    Error: Cannot find module '~/projects/apigeelint-bug-repro/node_modules/apigeelint/node_modules/@xmldom/xmldom/lib/dom.js'
    Require stack:
    - ~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/myUtil.js
    - ~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/Resource.js
    - ~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/Bundle.js
    - ~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/bundleLinter.js
    - ~/projects/apigeelint-bug-repro/node_modules/apigeelint/cli.js
        at Module._resolveFilename (node:internal/modules/cjs/loader:1142:15)
        at Module._load (node:internal/modules/cjs/loader:983:27)
        at Module.require (node:internal/modules/cjs/loader:1230:19)
        at require (node:internal/modules/helpers:179:18)
        at ~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/myUtil.js:220:12
        at Object.<anonymous> (~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/myUtil.js:224:3)
        at Module._compile (node:internal/modules/cjs/loader:1368:14)
        at Module._extensions..js (node:internal/modules/cjs/loader:1426:10)
        at Module.load (node:internal/modules/cjs/loader:1205:32)
        at Module._load (node:internal/modules/cjs/loader:1021:12) {
      code: 'MODULE_NOT_FOUND',
      requireStack: [
        '~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/myUtil.js',
        '~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/Resource.js',
        '~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/Bundle.js',
        '~/projects/apigeelint-bug-repro/node_modules/apigeelint/lib/package/bundleLinter.js',
        '~/projects/apigeelint-bug-repro/node_modules/apigeelint/cli.js'
      ]
    }

    Node.js v21.7.2
    ```

## My understanding of the issue

It would seem that requiring a package that is also required by of `apigeelint` but on a different (and incompatible)
version is what's causing the issue. It leads to a `node_modules` folder being created under `node_modules/apigeelint`
with _only_ the conflicting dependencies, but the fix introduced in v2.57.0 assumes that `@xmldom/xmldom` will always
exist under the very first `node_modules` that it finds when traversing upwards.
