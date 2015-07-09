---
title: Upgrading to Makara 2.0.0
layout: blog
---

With the [release of makara 2.0.0] last week, I figured it would be a good time to illustrate how to migrate to the new internationalization system from an application using the Kraken 1.0.0 components like [localizr] and [engine-munger].

First, let's install the new components and remove the old:

```
npm rm --save adaro dustjs-helpers dustjs-linkedin engine-munger grunt-localizr
npm install --save "makara@^2.0.0" "dust-makara-helpers@^4.0.0"
```

The new makara depends on its own adaro, which depends on its own dustjs, so you don't need to manage those yourself in the application as `peerDependencies` any longer.

Next, let's change the view engine in the Kraken configurations. If you have generated your kraken application using the generator, you'll have a `config/config.json` and a `config/development.json` that both have view engines defined. You'll need to edit all of the configuration files that define view engines. They look something like this:

```javascript
       "dust": {
            "module": "engine-munger",
            "renderer": {
                "method": "dust",
                "arguments": [
                    { "cache": false },
                    {
                        "views": "config:express.views",
                        "view engine": "config:express.view engine",
                        "specialization": "config:specialization"
                    }
                ]
            }
        }
```

The new version is a bit simpler:

```javascript
       "dust": {
            "module": "makara",
            "renderer": {
                "method": "dust",
                "arguments": [
                    { "cache": false, "helpers": "config:dust.helpers" }
                ]
            }
        }
```

Do the same for the `"js"` engine. The configuration is exactly the same as the `"dust"` engines.

At this point, you may want to consider using the `"dust"` engine even in production, but with `"cache"` turned on (the default). Caching now works properly with multiple languages, and won't continuously recompile templates in that case. The only reason to disable caching is for development.

The engines do a lot less work because the express view class does more of the work itself and handles what looks up where. The `"helpers"` property is new, though, and is a list of references to modules to load as dust helpers. Let's add it now, at the root of the `config/config.json`:

```javascript
    "dust": {
       "helpers": [
           "dust-makara-helpers"
       ]
    },
```

This loads the new [dust-makara-helpers] into the copy of dust that [adaro] requires. It now uses its own private copy of dust, so if you configure helpers elsewhere in your application, you'll have to move those into configuration like this.

Now, let's add the Express View class replacement. In plain Express, you can just set this as a setting like `app.set("view", something here)`, but Kraken can't quite express what's needed in its config files, so we wrote a middleware that does this on its first invocation (with the first request to the app). Here's the configuration to add to the `"middleware"` section of the configuration:

```javascript
        "expressView": {
            "priority": 100,
            "enabled": true,
            "module": {
                "name": "makara",
                "arguments": [
                    {
                        "i18n": "config:i18n",
                        "specialization": "config:specialization"
                    }
                ]
            }
        },
```

This loads the View class, and configures its `i18n` and `specialization`. If you're not using either, you probably don't want anything in this document at all and could just use [adaro] directly, but if you are, you can leave out either if you're not using them. Here's a basic configuration at the root of `config/config.json` compatible with Kraken 1.0 generated apps:

```javascript
   "i18n": {
        "contentPath": "path:./locales",
        "fallback": "en-US"
    },

    "specialization": {
        "jekyll": [
            {
                is: "hyde",
                when: {
                    "whoAmI": "badGuy"
                }
            }
        ]
    },
```

At this point, rendering with the `"dust"` engine should work. For the precompiled `"js"` engine, we'll need to add some build tools.

There's no need for the old localizr task. Let's remove this from `tasks/i18n.js`:

```javascript
    grunt.registerTask('i18n', [ 'clean', 'localizr', 'dustjs', 'clean:tmp' ]);
```

becomes

```javascript
    grunt.registerTask('i18n', [ 'clean', 'dustjs', 'clean:tmp' ]);
```

And the `tasks/dustjs.js` task is going to need some clean-up, particularly the `fullname` function, but also it can pull its templates from `public/templates/` directly, since there's no localizr putting things in `tmp/`. Let's replace it with this:

```javascript
'use strict';

var path = require('path');

module.exports = function dustjs(grunt) {
    // Load task
    grunt.loadNpmTasks('grunt-dustjs');

    // Options
    return {
        build: {
            files: [
                {
                    expand: true,
                    cwd: 'public/templates/',
                    src: '**/*.dust',
                    dest: '.build/templates',
                    ext: '.js'
                }
            ],
            options: {
                fullname: function (filepath) {
                    return path.relative('public/templates/', filepath).replace(/[.]dust$/, '');
                }
            }
        }
    };
};
```

And the `tasks/localizr.js` isn't needed at all:

```
git rm tasks/localizr.js
```

Finally, make sure that each template has a corresponding `.properties` file with its localized strings, or, if that makes a lot of empty or duplicate content bundles, add `{@useContent}` helpers to your templates like so:

```javascript
    "dust": {
        "helpers": [
            {
                "module": "dust-makara-helpers",
                "arguments": [ { "autoloadTemplateContent": false } ]
            }
        ]
    },
```

And add `{@useContent}` helpers to each template to select which bundles to load from. See [makara] for more information there.

[makara]: http://krakenjs.com/makara
[adaro]: http://krakenjs.com/adaro
[engine-munger]: https://github.com/krakenjs/engine-munger
[release of makara 2.0.0]: {% post_url 2015-07-06-new-i18n-for-dust %}
[localizr]: https://github.com/krakenjs/localizr
[dust-makara-helpers]: https://github.com/krakenjs/dust-makara-helpers
