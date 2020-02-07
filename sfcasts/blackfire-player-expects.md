# Expectations/Tests with Blackfire Player

We just used `blackfire-player` to execute our first "scenario". It's pretty
simple: it goes to the homepage then clicks the "Log In" link. But... it works!

But... we're not *doing* after we visit these pages. The *true* power of
`blackfire-player` is that you can add *tests* to your scenario - or even scrape
pages and save that data somewhere.

## Adding an Expectation/Test to a Page

To add a "test" - or "assertion" - to the homepage say `expect` followed by - you
guessed it! - an *expression*: `status_code() == 200`. Copy that and add it to the
login page as well.

Ok, try `blackfire-player` again!

```terminal-silent
blackfire-player run scenario.bkf --ssl-no-verify -v
```

Woo! It still passes and *now* it's starting to be useful.

## What's Possible in the expect Expression?

Let's break this down. First, *just* like we saw with the `metrics` stuff, this
is an *expression* - it's Symfony's ExpressionLanguage once again - basically
JavaScript. And second... this expression has a *ton* of functions built-in.

Search the `blackfire-player` docs for `status_code`... and keep searching until
you find a big functions list. Here it is. Yep, we can use `current_url()`,
`header()` to get a header value and many others. The `css()` function is
especially useful: it allows us to dig into the HTML on the page. We'll use that
in a minute. The docs also have good examples of how to do more complex things.
But we're not going to become Blackfire player experts right now... I just want
you to get comfortable with writing scenarios.

## Asserting HTML Elements with css()

Let's try to write a *failing* expectation to see what that looks like. Let's see...
we could find this table and assert that it has more than 500 rows... which it
definitely does *not*. Let's find a CSS selector we could use... hmm. Ah, we can
look for a `<tbody>` with this `js-sightings-list` class and then count its
`<tr>` elements.

Back inside the scenario file, add another expect. This time use the `css()`
function and pass it a CSS selector: `tbody.js-sightings-list tr`. Internally,
The `blackfire-player` uses Symfony's `Crawler` object from the `DomCrawler`
component, which has a `count()` method on it. Assert that this is `> 500`.

Let's see what happens!

```terminal-silent
blackfire-player run scenario.bkf --ssl-no-verify -v
```

And... yes! It fails - with a nice error: the `count()` of that CSS element is
25, which is not greater than 500.

Go back change this to 10. The data is dynamic data... so we don't *really*
know how many rows it will have. But since our fixtures add more than 10... and
because there will probably be at least 10 if we ever ran this against production,
this is probably a safe value.

Try it again and:

```terminal-silent
blackfire-player run scenario.bkf --ssl-no-verify -v
```

It passes!

## Typos in Expressions

Another thing that `blackfire-player` does well is its *errors* when I mess
something up. Make a typo - change `count()` to `ount()`, and rerun the tests,

```terminal-silent
blackfire-player run scenario.bkf --ssl-no-verify -v
```

> Unable to call method ount of object `Crawler`.

That's a *huge* hint to tell you what object you're working on so you can figure
out what methods it *does* have. Change that back to `count()`.

## Performance Assertions in the Scenarios?

As we've seen, the `blackfire-player` has *nothing* to do with the Blackfire
profiler. It's just a useful little tool for visiting pages, clicking on links
and adding assertions. But... if it *truly* had nothing to do with the profiler,
I wouldn't have talked about it. In reality, the concept of "scenarios" will
become *super* important later when we introduce "builds".

And actually, there is one *little* integration between `blackfire-player` and
the profiler: you can add *performance* assertions to your scenario! To do that,
say `assert` and then use any performance assertion you want - the same strings
that you can use inside a test. For example: `metrics.sql.queries.count <= 30`

When we execute this:

```terminal-silent
blackfire-player run scenario.bkf --ssl-no-verify -v
```

it *does* still pass. But if you played with this value - like set it to `< 1`
and re-ran the scenario:

```terminal-silent
blackfire-player run scenario.bkf --ssl-no-verify -v
```

Yea, it *still* passes... even though this page is *definitely* making more than
one query. The reason is that the `assert` functionality *won't* work inside
a scenario until we introduce Blackfire "environments" - which we will very soon.
They are one of my absolute *favorite* parts of Blackfire.

For now, I'll leave a comment that this *won't* until then.

Next, let's deploy to production! Because once our site is deployed, we can
*finally* talk about cool things like "environments" and "builds". You can use
anything, but to deploy, *we* will use SymfonyCloud.