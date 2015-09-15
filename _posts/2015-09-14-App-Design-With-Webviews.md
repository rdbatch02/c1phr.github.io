---
layout: post
title: App Design with Webviews
comments: true
---
Since starting school again, it's been difficult to find time to work on my app. Even though I haven't been writing any code, I've still been thinking about it and trying to plan for what I need to do next. Fast-foward to this morning, an article appeared in my Twitter feed [about webviews](https://medium.com/@dlpasco/apple-tv-a-world-without-webkit-5c428a64a6dd). Sure, the article is about the absence of webviews in Apple's new [tvOS](https://www.apple.com/tv/), but it actually made me realize that webviews might be the secret to getting me to launch day in a reasonable time-frame.

## Why webviews?
A good chunk of my app code was going to be duplicated across both platforms, but I was initially trying to design and lay this information out within storyboards in Apple's interface builder. As much as interface builder is great, there's a lot of work involved to get semi-detailed interfaces put together. Then, after all that work, I would have to do it all over again once I want to port to Android. Given my current time constraint problems, doing more work to port to Android might mean those users can get in on the action by next NHL season if I'm lucky.

I've considered a few options to the cross-platform problem:

* Xamarin - This is probably my favorite solution since it means I get to write in my favorite language (C#). Unfortunately, licenses are expensive, and I really wouldn't be learning native development for either platform.
* Cordova - This option seems to be gaining popularity, especially as JavaScript frameworks like AngularJS gain momentum. This has the same problem of not teaching me much about native development, and I also really don't like front-end JavaScript frameworks. But, this option is essentially free, so maybe I won't throw it off the table quite yet.

I still want to write some native code, but I could probably do a much better job laying out some of my interfaces if I could do it in HTML and CSS. On top of making it easier, I could also re-purpose that code for both platforms. Enter webviews.

## So, webviews
Both iOS and Android support a UI element called Webviews. A webview is basically just an object that will render HTML, CSS and JavaScript inside of a native application. Webviews don't provide any additional controls for the web page, so it's a seamless element that can be essentially indistinguisable from native UI to users.

On top of being more straightforward to use, these webview UIs are portable between platforms. The only code that I'll have to tweak is CSS to make sure everything looks good on multiple screen sizes. Webviews are going to help me get to launch day much quicker than before, and I have a feeling that Android won't be too far behind.