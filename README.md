# ⚡ How to eliminate render-blocking JavaScript and CSS

[Csaba Palfi](https://csabapalfi.github.io), Dec 2016

Most web performance recommendations are simple to put into practice. Minifying assets or optimizing images is a matter of configuring some basic tooling.

Other optimizations assume you have more knowledge of browser internals but can really pay off in terms of user experience. (Especially for mobile users on a slow connection.)

This post is about eliminating render blocking resources and prioritizing above-the-fold content. These practices enable [optimized (progressive) rendering](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/) of your web pages.

## Do you have the problem?

### PageSpeed Insights

Do you fail the [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) rule below?

> **Eliminate render-blocking JavaScript and CSS in above-the-fold content**

Hint: use [ngrok](https://ngrok.com/) to run PageSpeed Insights against localhost or a private network.

### WebPageTest.org

Is your [Speed index](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/metrics/speed-index), [Start Render ](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/quick-start-quide#TOC-Start-Render:) and Visually Complete times too high? Ideally your Visually Complete time should be less than a second.

(See the Details tab on the results page for the Visually Complete time.)

### Chrome Developer Tools

Enable throttling (e.g 2G or 3G) in the Network tab then go to your page. Are you staring at a blank screen for seconds? (Be sure to start from `about:blank` to really see how long it takes for rendering to start).

You can also look at the screenshots/filmstrip in the [Timeline tab](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool).

## How CSS resources can block rendering?

It's important to understand the critical steps that need to happen before a single pixel can be painted on the screen. The browser:

* parses your [HTML into the DOM tree and your CSS into a CSSOM tree](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model)
* these are [combined into a render tree](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction) that only contains the visible DOM content
* the layout and paint steps depend on the render tree which depends on the CSSOM - [nothing can be put on the screen before that's ready](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css)

When the browser encounters a link tag for a stylesheet it'll have to go and fetch it as it's needed for building the render tree. Sometimes that CSS file is your bundle containing the styles for your whole page (maybe even the whole website).

Your users could be waiting for the styles for something that's not even on the screen! (i.e. below the fold). For someone on a slow connection that can mean seeing a blank screen for seconds.

Side-note: The main reason why the browser block rendering until it has the styles is to avoid [FUOC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content).

## How to fix render blocking CSS?

### Inline critical styles

Critical-path styles are the styles that are required to render correctly everything that's **above the fold**.

You should inline these styles to avoid additional HTTP requests blocking initial rendering. You'll find that there's a [collection of tools to help with this](https://github.com/addyosmani/critical-path-css-tools).

```html
<head>
  ...
  <style>critical CSS rules here</style>
  ...
</head>
```

Some additional tips:
* You should consider figuring out what above-the-fold means for your users based on web analytics.
* Also make sure you don't use `@import` statements in critical CSS as that'll also trigger additional HTTP requests.
* Be sure not to inline too much CSS as that'll also slow down your initial render.
* Read some more explanation/tips from [Google](https://developers.google.com/speed/docs/insights/OptimizeCSSDelivery) and [Varvy](https://varvy.com/pagespeed/optimize-css-delivery.html).

### Load non-critical CSS asynchronously

Inlining your critical CSS is not enough as the browser will still block rendering as soon as it encounters your non-critical CSS referenced using a `link` tag.

Luckily there's a new browser feature called [Preload (aka `rel=preload`)](https://www.smashingmagazine.com/2016/02/preload-what-is-it-good-for/) that allows us to start fetching our CSS but prevent it from blocking. At the moment it's [only supported on Chrome](http://caniuse.com/#search=preload) but it can be [polyfilled on other browsers](https://github.com/filamentgroup/loadCSS).

See recommended usage in the [polyfill docs](https://github.com/filamentgroup/loadCSS#recommended-usage-pattern). Your code for this will look something like this:

```html
<head>
  ...
  <link rel="preload" href="/non-critical.css" as="style" onload="this.rel='stylesheet'">
  <script>inlcude loadCSS polyfill here</script>
  ...
</head>
```

Tip: The linked polyfill is not in any module format but with Webpack you can just use the script-loader to make sure it runs properly.

## How about render-blocking JavaScript?

JavaScript can [also block rendering](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/adding-interactivity-with-javascript). When the browser sees a `script` tag it'll need to pause building the DOM tree and execute the script. That's one of the reasons why it's generally recommended to have script tags at the bottom of you page. Fortunately it's also much easier to make JavaScript loading asynchronous since the script tag has the `async` attribute.

You might also want to inline some (small amount of!) JavaScript that's critical for above the fold content (and of course the loadCSS polyfill if you're using `rel=preload` for styles)

![](https://ga-beacon.appspot.com/UA-29212656-1/eliminate-render-blocking?pixel)
