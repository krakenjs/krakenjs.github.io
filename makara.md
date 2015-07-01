---
title: Makara
layout: documentation
logo: Makara.svg
description: Internationalization support for Kraken with Dust Templating
---

Internationalizing an application is the process of making all the culture and language-senstitive parts are factored out, handled systematically and set up for translation. Internationalization is usually abbreviated as 'i18n'.

There are many techniques for i18n, but the practices we've found that work are codified in the `makara` module.

There is an option in the [Kraken generator] to generate an application that includes i18n support. To configure it manually in an existing project, see [adding Makara to a project].

Makara is mostly a configuration of its component parts, suitable for dropping into a Kraken application and working with relatively little configuration.

It consists of [bundalo] for loading localized strings for use by application logic, [engine-munger] for controlling the lookup of templates and associated localized strings, and [adaro] as the template engine, connecting [dustjs-linkedin] to Express.

Makara can be used in plain Express apps, but the examples on this page will all be using Kraken.

Adding Makara to an existing project
------------------------------------

In your `config/config.json`, add the middleware exported by makara to the middleware configuration.

```javascript
        "makara": {
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

The middleware, on the first request, sets up the Express View class to be the one provided by [engine-munger].

In this case, we're using two `config:` shortstop handlers to put those configuration blocks somewhere consistent, at the root of the configuration object. Both the i18n and specialization configuration are optional, though if you're not using either, you could just use [adaro] on its own as your view engine without using makara at all.

Add the i18n and specialization configuration to `config/config.json`

```javascript
    "i18n": {
        "contentPath": "path:./locales",
        "fallback": "en-US"
    },

    "specialization": {
        "oldtemplate": [
            {
                "is": "newtemplate",
                "when": {
                    "testmode": "beta"
                }
            }
        ]
    },
```

We've left the specialization configuration very limited for the time being; for more information on template specialization for experimentation and A/B testing, see the documentation on the [karka rule engine].

Add a middleware that sets `res.locals.locale` and `req.locale`:

```javascript
        "bcp47": {
            "priority": 10,
            "enabled": true,
            "module": {
                "name": "express-bcp47",
                "arguments": [ { "defaultLocale": "en-US", "vary": true } ]
            }
        },
```

Notice the low priority: this should get set for every request that could in any way involve rendering a template.

You can write your own middleware to do this if you base deciding what locale to use for a user on something other than browser `Accept-Language:` headers. This could be from a lookup from a user object, or a cookie, or any other factor -- and in fact, that could be a middleware that overrides this one in only some cases, leaving `bcp47` as a fallback strategy.

Set up the view (template) engines:

```javascript
    "view engines": {
        "dust": {
            "module": "makara",
            "renderer": {
                "method": "dust",
                "arguments": [
                    { "cache": true, "helpers": "config:dust.helpers" }
                ]
            }
        },
        "js": {
            "module": "makara",
            "renderer": {
                "method": "js",
                "arguments": [
                    { "cache": true, "helpers": "config:dust.helpers" }
                ]
            }
        }
    },
```

The `js` version is used for production time builds, keeping the compilation of templates completely offline. However, the compilation is cached, so the `dust` engine should suffice for most situations.

Add the dust helpers configuration:

```javascript
    "dust": {
        "helpers": [
            "dust-makara-helpers"
        ]
    },
```

That loads the `dust-makara-helpers`, which supply the `{@message}` and `{@useContent}` helpers.

Configuration
-------------

Makara's middleware takes several configuration values:

* `i18n.contentPath` - `String`, the root to search for content in. Required. In generated Kraken apps, this will be `path:./locales`, to tell makara that the language strings are in the `locales` directory in the project root.
* `i18n.fallback` - `String` or `Object` as [`bcp47`] creates, the locale to use when content isn't found for the locale requested. Required. If your content starts off in US English, use `en-US` as a fallback. If your app's native language is another, using that is probably what you want. When you can't tell what your user's langauge is, what should it be? Plan your design accordingly.
* `specialization` - `Object`, if you're doing rule-driven template specialization, for experiments or handling certain cases cleanly, you'll want to specify the specialization rule map here.

There are others in the [makara] documentation if you need them, such as `i18n.formatPath` and `enableMetadata`.

Content
-------

Content intended for localization is stored in `.properties` files as simple `key=value` pairs.

These are the files that hold the content strings for the different languages your application supports.

Normally, you are likely to start with a master set of content (likely in English) and the localization process will populate corresponding files for the other languages you will need.

### Placement of `.properties` files

The root of the `.properties` content files is the locales folder at the top level of your project. Under it will be a folder per country (as in `US/`, `DE/`, et cetera).  Below each country folder is one or more language folders (as in `en/`, `de/`).  So `locales/US/en/` will be the likely location for your master set of `.properties` files.

`.properties` files are correlated with the dust templates that use them, by path and name.

If you have a top level `index.dust` file, its content `.properties` file will be at `locales/US/en/index.properties` This holds all the external content strings used by that template. If your template is at `widgets/display.dust` then the content for US English will be at `locales/US/en/widgets/display.properties`.  If you have content you want to share across pages, then you should factor out use of that content into a separate partial and use that partial to achieve
content sharing.

You can override this filename mapping by providing a `formatPath` function to the makara i18n configuration.

### What's in a `.properties` file

The parser for this file format is [spud].

The format is simple: `key=value` with one message per line encoded as UTF-8.  Comments are prefixed with `#`.

Let's look at some samples and then use them to discuss various points.

`index.properties` file:

```
index.title=PayPal Merchant
index.callToAction=Enroll now!
index.greeting=Welcome {userName}

# A list
index.ccList[0]=Visa
index.ccList[1]=Mastercard
index.ccList[2]=Discover

# A map
index.states[AL]=Alabama
index.states[AK]=Alaska
index.states[AZ]=Arizona
index.states[CA]=California
```

We are using the name of the file to start our key on each line. This is strictly a convention that makes the path to the file clear. There's duplication between the two, but it makes debugging easier.

Text to the right of the `=` sign is a simple message string with the text of the message.

If you have runtime values to be inserted, use dust brace to select the value from the dust template context as in the `index.greeting` line. Note that there is no restriction on inserting HTML tags into the messages. They are just another string of characters as far as the content processing is concerned.

In addition to simple strings, we support lists and maps. The `index.ccList` above might be used to provide a list of values to go in a list of allowed credit cards.

The `index.states` might be used to populate a dropdown list of states with the key as the option tag value and the full state name as the visible text.

To support writing the key part in natural languages other than English, all UTF-8 characters are allowed with a few exceptions needed to make the key=value syntax work. The exceptions are:

* No equal sign in key part (e.g. first equal sign starts the value)
* No periods in key part (used to allow keys like a.b.c)
* No square brackets (used for subscript and map key notation)
* May not start with # (Used for comments)

Additional detail is in the documentation for [makara].

If you use keys that are both objects and strings, there are some edge cases discussed in [key promotion]

[key promotion]: https://github.com/krakenjs/makara#key-promotion

### Referencing internationalized content from a dust template

This is done using the `{@message}` helper tag. A sample usage of `@message` is:

```
{@message type="content" key="index.title"/}
```

Lists and maps are bit trickier when it comes to inlining.

There are two approaches available. The first uses three additional attributes on the `@message tag`, `before="xxx"` and `after="yyy"` and `sep="z"`.  When emitting the list elements, each will be prefixed by the "before" string, if there is one, suffixed by the "after" string, if there is one, and separated by the "sep" string, if there is one. With sep, the last element is not followed by the separator. Note that the value `{$idx}` can be used in the before/after attribute strings and it will be replaced by the current iteration count when inlining the lists. Similarly, `{$key}` will be replaced with the current key when inlining a map. No replacement is done in the sep string.

In some cases inlining won't do, even with before/after/sep. For example, if you need to pass the list as a parameter to a templating partial that might implement a dropdown functionality.

For this, `@message` with a `mode="paired"` attribute offers more flexibility.

The `mode="paired"` parameter produces the content list such that you can use both the index of the element for the value in an option tag and the value for the displayable text.

The `mode="paired"` attribute delivers the content in the form of a JSON object, which in the case of a list of months might look like:

```json
[{$id:0,$elt:"Jan"}, {$id:1,$elt:"Feb"},.. ]
```

This gives you more ability to work with both the list/map value and the element value in your template.

In addition to `mode="paired"` there is an alternate form, `mode="json"` This generates the content list or map as a standard JavaScript array or an object with properties, respectively.

For more on using the `@provide` helper, see the [advanced helper] docs

[engine-munger]: https://github.com/krakenjs/engine-munger
[`bcp47`]: http://npmjs.org/package/bcp47
[@provide]: https://github.com/rragan/dust-motes/tree/master/src/helpers/data/provide
[advanced helper]: ADVANCED.md
[adaro]: https://github.com/krakenjs/adaro
[bundalo]: https://github.com/krakenjs/bundalo
[dustjs-linkedin]: http://dustjs.com/
[spud]: https://github.com/krakenjs/spud
[karka rule engine]: https://github.com/krakenjs/karka
[adding Makara to a project]: #adding-makara-to-an-existing-project
[Kraken generator]: https://github.com/krakenjs/generator-kraken
