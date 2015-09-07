+++
title = "If you're not using Cloudflare for your budget website, you're probably doing it wrong."
author = "cbrown"
tags = ["web", "cloudflare", "optimization", "security"]
date = "2015-09-07T14:29:49-07:00"
+++

Does your website minify its HTML and Javascript? Does it compress its transfers? Is it using a CDN for static content? Does it protect your users' privace with TLS? Does it have a firewall to defend against attackers?

If your answer to any of these questions is "no", you're definitely doing things wrong. And if you're on a budget, [Cloudflare](https://www.cloudflare.com) gives you all of these things, at absolutely **no cost**. I've racked my brain over this, and I have no idea why any informed web admin *wouldn't* use Cloudflare.

Now, before I continue, let me re-emphasize the *budget* part. Cloudflare is an amazing solution when you want to keep things cheap, but if you're playing with corporate money, you'll likely want to do something else (like set up your own WAF, load balancers, and caching, which I may talk about in a future article).

Setup
=====

All you have to do to get set up is create a Cloudflare account, add your website to it, and change your hosting's nameservers to Cloudflare's. It's a very quick and painless process. When you review your DNS, make sure the status icon is of the arrow going *through* the cloud.

Once you do this, you should take a look at your page rules. On the free Cloudflare plan, you get three. That's plenty for simple sites. We use one to ensure that visitors always get HTTPS. And another to set up the caching for all of our pages. By default, Cloudflare doesn't cache HTML, so because our site is 100% static, we use this rule to make sure it caches everything. Then whenever we update the site, we just clear the relevant pages from Cloudflare's cache.

So now what?
============

Your visitors will likely immediately notice a dramatic improvement in performance. Most requests will be served by the cache of the Cloudflare edge node located closest to your visitor. They'll be minified and compressed, and subsequent visits will serve assets from their browser cache. So in addition to your visitors getting a much zippier browsing experience, you save money on bandwidth. And you can provide TLS as easily as flipping a switch, without having to pay a certificate authority.

Cloudflare also provides some firewall functionality. It will detect several types of attacks and shut them down for you. And if it doesn't detect the attack on its own, you can raise the security level and ban users either manually or from your server using tools like [Fail2ban](http://www.fail2ban.org/) (Cloudflare provides a REST API that can be used.).

And you can get all of this on the free plan.

So why wouldn't you use Cloudflare for a budget site? If you have a reason, please enlighten me in the comments.