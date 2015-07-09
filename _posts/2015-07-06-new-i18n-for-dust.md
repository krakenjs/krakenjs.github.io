---
title: New I18n Support for DustJS
layout: blog
---

Last week we soft-released a new version of [makara] and [adaro], the internationalization components for Kraken and Dust templating. It's a whole re-engineering of the system to solve a bunch of flaws and to future-proof it.

Three main things drove the redesign:

* [Dust 2.7] came out and broke one key interface that Kraken's [adaro] had relied on. `dust.load` was moved internal, where before it was just undocumented private method exposed to the world. Because of this, adaro's hacks to override the loader no longer work and a whole house of cards come tumbling down. We worked with the dust team on [dust 2.7.1] to get an interface we can use cleanly. (Just as a side note: dust doesn't use semantic versioning, so its _minor_ releases are API breaking changes, at least in the 2.x series.)
* We want to add support for number and gender agreement in localized messages, so that means that building a copy of the template, pre-localized, for each locale is no longer feasible: strings aren't static and can't be reduced to simple dust variable substitutions.
* Build times with many locales and many templates get very high. If you ship 600 locales and 1000 templates, that's 600,000 generated files, mostly duplicated information.

The changes ended up affecting nearly every i18n package, particularly because the integration with Dust changed so much.

[localizr] and [grunt-localizr] are deprecated. They're tools for doing the static pre-inlining of messages into templates.

We introduced [dust-makara-helpers], which is a wrapper around [dust-usecontent-helper] providing `{@useContent}` to load content into the context, and [dust-message-helper], providing `{@message}` and a backward compatibility alias `{@pre}`, to do the message insertion and rendering at template render time, instead of precomputed at build time. In the future, [dust-intl]'s `{@formatMessage}` helper can be used with a similar syntax to [dust-message-helper]'s and will support gender and number agreement.

[engine-munger] has been reworked to just be a replacement View class for Express, backporting the asynchronous lookup of templates from Express 5, and extending that API to allow looking up content as well as partials and templates. It calls the template engine the same way Express 5 will, with `this` being an instance of itself, so its methods can be used from engines.

[adaro] has been completely reworked. It's now just a stand-alone dust template engine for Express. It can use the Express 5-compatible lookup interface provided by [engine-munger] though, so it supports internationalization that way.

All of this gets rolled up in the new version of [makara], made relatively easy to configure all in one place, with some strong defaults and backward compatibility with existing kraken projects turned on by default.

[makara]: http://krakenjs.com/makara
[adaro]: http://krakenjs.com/adaro
[localizr]: https://github.com/krakenjs/localizr
[grunt-localizr]: https://github.com/krakenjs/grunt-localizr
[dust-makara-helpers]: https://github.com/krakenjs/dust-makara-helpers
[dust-usecontent-helper]: https://github.com/krakenjs/dust-usecontent-helper
[dust-message-helper]: https://github.com/krakenjs/dust-message-helper
[Dust 2.7]: https://github.com/linkedin/dustjs/releases/tag/v2.7.0
[dust 2.7.1]: https://github.com/linkedin/dustjs/releases/tag/v2.7.1
[dust-intl]: http://formatjs.io/dust/
