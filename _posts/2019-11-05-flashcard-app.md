---
layout: post
title: Spaced Repetition Algorithm for a Flashcard App  
date:   2019-11-05
categories: Programming
image: /img/step3.webp
image-text: Flash Card
image-link: https://www.kartoj.com/
---

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">So, I&#39;ll be going to Mexico in a few months and I wanted to brush up my Spanish vocabulary ...<br><br>So I built <a href="https://t.co/lNgilbwR3o">https://t.co/lNgilbwR3o</a> with flashcards that have Spanish on one side, and images from a web search in Mexico on the other side.<br><br>Source code at: <a href="https://t.co/Phl3B2Dd1K">https://t.co/Phl3B2Dd1K</a> <a href="https://t.co/xNtR6WrmZz">pic.twitter.com/xNtR6WrmZz</a></p>&mdash; Eamonn O&#39;Brien-Strain 👨‍💻🔍🌁🇮🇪🇪🇺🇺🇲🇺🇳⚛️ (@eob) <a href="https://twitter.com/eob/status/1191894849470906368?ref_src=twsrc%5Etfw">November 6, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

There were some interesting aspects to building this web app.

1. The [spaced repetition][1] algorithm for displaying flash cards to maximise learning.
2. Using locale-specific web search to fetch the images 
2. Architecting the app as a static web site that uses local storage to store user state.
3. Using vanilla JavaScript with no framework.
4. Using 3D animations for flipping cards.
5. Custom creating all the CSS styling.

In this post, I'll discuss the spaced-repetition algorithm, and defer the other
aspects to future posts.

From Wikipedia

> Spaced repetition is an evidence-based learning technique that is usually
> performed with flashcards. Newly introduced and more difficult flashcards are
> shown more frequently while older and less difficult flashcards are shown less
> frequently in order to exploit the psychological spacing effect. The use of
> spaced repetition has been shown to increase rate of learning.

There is a variety of published research on spaced-repetition, and different
flashcard apps use a variety of different algorithms, some quite complicated.

I decided to start with something much simpler. Every time a user sees the card
they say how accurately they predicted the reverse side of the card. We store a
`response` object that has this `correctness` score along timestmp `t` of the
response. For each card we calculate an average score, but weighted by time so
that more recent scores count for more. Then we sort the list of cards so that
the cards with the lowest scores come first.

The core of the algorithm is in [common.js][2]:

```js
const TAO = 1000.0 * 60 * 60 * 24

export const score = (responses) => {
  if (responses.length === 0) {
    return -1
  }
  const now = Date.now()
  const weightedSum = responses
    .map((response) => response.correctness * Math.exp((response.t - now) / TAO))
    .reduce((sum, x) => sum + x)
  return weightedSum / responses.length
}
```

The `score` function goes through the array of responses and sums up the
correctness values weighted by an exponential decay function with a time
constant `TAO` of 24 hours. This means, for example, that a day-old response is
weighted at about 0.37 of a recent response.

This `score` is called from the `sort` function in [index.js][3]:

```js
  const sort = () => {
    cards.sort((a, b) => score(a.responses) - score(b.responses))
  }
  ```

My hypothesis is that this simple algorithm with its single TAO parameter is as
effective as the more complex algorithms. Hopefully by tuning TAO I can emulate
those algorithms.

An observant reader will have noticed some significant algorithmic
inefficiencies in the above naive implementation of the algorithm. In particular
there are some superlinear big-O complexities that I'll probably need to
mitigate as I add more cards to the app.

[1]: https://en.wikipedia.org/wiki/Spaced_repetition
[2]: https://github.com/eobrain/mergi/blob/b154a912bb1fd0aed98df77a6b8265e36d0d9214/site/common.js
[3]: https://github.com/eobrain/mergi/blob/b154a912bb1fd0aed98df77a6b8265e36d0d9214/site/index.js



