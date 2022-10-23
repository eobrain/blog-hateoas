---
layout: post
title: Here's the thing that has ended the code-formatting wars. 
date:   2020-01-25
categories: Programming
image: /img/standard-js-sticker.svg
image-text: standard JS logo
image-link: https://standardjs.com/
---

> War is over, if you want it
>
> War is over now
>
> -- Lennon

Have you been a belligerent in the code-formatting wars? Do you have strong
opinions on tabs vs. spaces or on the merits of The One True Brace Style over
other [indentation styles][1]?

Something has now changed that renders these wars obsolete.

When I joined Google one culture shock was discovering how picky Google was
about seemingly trivial details of formatting. Time and again my code reviewers
objected to what I thought was my creative expression of coding craft. They were unmoved
by my assertion that `b*b - 4*a*c` was more readable than `b * b - 4 * a * c` or
that `FOO` was a better name for a constant than `kFoo`.

Quickly I came to peace with the style guides and recognized their value. I
realized that operating in Google's [fabled monorepo][2] I was reading more code
than I was writing, making me appreciative of how a consistent style made the
code much easier to read. Also as a code reviewer and tech lead I appreciated
how just a link to a section of the quasi-legal style guides could cut short
tedious bikeshedding on formatting issues and allow us to concentrate on the
important issues.

Initially it was a bit tedious getting all the spacing right and remembering all
the subtle line-breaking rules.

But over time it got better. First the linting tools improved and covered more
of the style guide so that I generally could find and fix all the issues before
sending code changes for review.

But then it got even better, with the linting tools supplemented by tools that
automatically reformatted code according to the style guide for that language.
Now I just basically no longer spend any brain cycles on formatting: I write
arbitrary sloppily-formatted code and have it fixed up when I save the file.

The designers of the Go language were Googlers, and they bought into this
philosophy. Pretty much all Go code is formatted by the gofmt program, which always reformats to the same standard style.

So when I started a [personal JavaScript project][4] I really wanted a
JavaScript equivalent to gofmt which enforces a standard style.

In the JavaScript ecosystem, [Eslint][5] is the standard linter, which not only
can warn against style violations, but can automatically fix them. And what's
more it can go beyond simple indentation and spacing and fix more complex rules
(all the ones marked with a 🔧 wrench in the [list of rules][6]).

Initially I assumed I would want to use the Google JavaScript style guide and the eslint rules that enforce it. I
configured Eslint with

```json
 {
    "extends": [
        "eslint:recommended",
        "google"
    ],
    "env": {
        "browser": true,
        "node": true
    },
    "parser": "esprima",
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module"
    }
}
```

However, it was a bit unsatisfying to have a non-trivial config file with
various things that need deciding and tweaking. It's also a bit annoying
that it cannot not autofix all the problems it finds, so you still have to manually fix some of them.

Nevertheless it works fine, and produces nicely formatted code like below in
standard Google format.

```js
export const newCards = () => {
  const cards = [];
  for (let i = 0; i < 2; ++i) {
    const reversed = (i === 1);
    forEachPhrase((phrase) => {
      const responses = [];
      cards.push({phrase, reversed, responses});
    });
  }
  return cards;
};
```

But then I discovered [standard JS][7]. This has everything a auto style
formatter should have. It is really easy to install, has good VS Code support,
requires no configuration, and can fix all the problems it finds. It produces
code like this:

```js
export const newCards = () => {
  const cards = []
  for (let i = 0; i < 2; ++i) {
    const reversed = (i === 1)
    forEachPhrase((phrase) => {
      const responses = []
      cards.push({ phrase, reversed, responses })
    })
  }
  return cards
}
```

The most notable difference is that it omits semicolons.

Omitting semicolons is something the JavaScript language allows, but the exact
rules for when you can do so have some subtle corner cases that can cause
ambiguity and unexpected interpretation of code, so the conventional wisdom has
long been to always include semicolons. That's the view of the Google style
guide.

But the standard JS community have decided that the cleaner semicolonless look
is preferred and have tackled the corner cases (which are rare) by making sure
they are handled by other standard JS rules.

Personally, I don't care whether there are semicolons or not, but I'm going to
stick with using standard JS for my projects as the tooling is better and it
looks like it's gaining momentum in the JavaScript community.

Now of course pretty-printing has existed for a long time. The [indent][3]
formatter dates from 1976, while text editors and IDEs have long been able to
autoformat. But all these pretty-printers have one fatal flaw: they are too
flexible, and can be configured with different style options.

The revolutionary novelty of Go's gofmt, standard JS, and Google's internal tool
is that **they have no options**. They enforce a single standard style.

The wars are over! Everyone should just use a standard auto-formatter with no
options.

[1]: https://en.wikipedia.org/wiki/Indentation_style
[2]: https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext
[3]: https://en.wikipedia.org/wiki/Indent_(Unix)
[4]: https://github.com/eobrain/mergi
[5]: https://eslint.org/
[6]: https://eslint.org/docs/rules/
[7]: https://standardjs.com/
