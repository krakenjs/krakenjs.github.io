---
title: Adaro
layout: documentation
logo: Adaro.svg
description: Templating using DustJS
---

Using DustJS templating in your application
-------------------------------------------

If you're not using [makara] for internationalization, which uses Adaro and other components internally, you can use Adaro directly.

You configure it in your `config/config.json` in the `"view engines"` section:

```javascript
    "view engines": {
        "dust": {
            "module": "adaro",
            "renderer": {
                "method": "dust",
                "arguments": [
                    { "cache": true, "helpers": "config:dust.helpers" }
                ]
            }
        },
        "js": {
            "module": "adaro",
            "renderer": {
                "method": "js",
                "arguments": [
                    { "cache": true, "helpers": "config:dust.helpers" }
                ]
            }
        }
    },
```

You'd add any helpers you want to load in a `dust.helpers` section:

```javascript
    "dust": {
        "helpers": [
            "my-dust-helper-package",
            "path:./lib/local-dust-helpers"
        ]
    }
```

Configuration
-------------

Adaro takes several configuration values:

* `cache` - defaults to `true`. Setting it to `false` will disable caching of templates in the dust cache.
* `helpers` - an array of dust helpers to load as Adaro is loaded. Adaro uses a private instance of dustjs, so you have to tell Adaro about helpers, not just modify the dust object elsewhere.


Helpers
-------

Helpers that Adaro accepts must be wrapped in a function to instantiate them:

```javascript
module.exports = function (dust, options) {
    // options are optional.
    // dust is a private instance, provided by Adaro.
    dust.helpers.something = function (chunk, context, bodies, params) {
    };
};
```

[makara]: makara.html
