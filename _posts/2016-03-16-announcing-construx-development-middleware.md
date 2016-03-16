---
title: Announcing construx development middleware
layout: blog
---

Previously, our development JIT-compile solution for all technologies (dust, less, sass, etc) was built entirely into a single module, [kraken-devtools](https://github.com/krakenjs/kraken-devtools). Over time, it became obvious that a single module attempting to support all the different types and versions of file transformations was unmanageable.

[`construx-*` modules](https://www.npmjs.com/search?q=construx) have been available for a few months now. All of our example applications have been upgraded to construx. And if you use the latest version of the [`generator-kraken`](https://www.npmjs.com/package/generator-kraken), you will get an application which uses them. For example, generating the following application:

```bash
LM-SJN-00872356:krakex medelman$ yo kraken myFreshKrakenApp

     ,'""`. 
hh  / _  _ \
    |(@)(@)|   Release the Kraken!
    )  __  (
   /,'))((`.\ 
  (( ((  )) ))
   `\ `)(' /'

Tell me a bit about your application:

? Description show new construx middleware
? Author PayPal
? Template library? Dust (via Makara 2)
? Include i18n support? Yes
? Front end package manager ? No
? CSS preprocessor library? SASS
? JavaScript library? RequireJS
```

Puts the following in `package.json` dependencies:

```js
	"construx": "^1.0.0",
    "construx-copier": "^1.0.0",
    "construx-dustjs": "^1.1.0",
    "construx-sass": "^1.0.0",
```

And the following in your `development.json` `middleware` section:

```js
"middleware": {
        "devtools": {
            "enabled": true,
            "priority": 35,
            "module": {
                "name": "construx",
                "arguments": [
                    "path:./public",
                    "path:./.build",
                    {
                        "template": {
                            "module": "construx-dustjs",
                            "files": "/templates/**/*.js",
                            "base": "templates"
                        },
                        "css": {
                            "module": "construx-sass",
                            "files": "/css/**/*.css"
                        },
                        "copier": {
                            "module": "construx-copier",
                            "files": "**/*"
                        }
                    }
                ]
            }
        }
    }
```

The main `construx` module orchestrates the filters defined by the other `construx-*` modules. Since all the technologies (dust, sass, etc) have their own module, we can more easily upgrade invididual filters when new versions of the given technology are released. It is also easy to author new filters using the [`construx-star` template repo](https://github.com/krakenjs/construx-star).

As always, contribution to these repos (and all krakenjs repos) is welcomed and encouraged. Happy Coding!