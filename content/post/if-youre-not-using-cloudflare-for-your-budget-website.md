+++
title = "If you're not using Cloudflare for your budget website, you're probably doing it wrong."
author = "cbrown"
tags = ["web", "cloudflare", "optimization", "security"]
date = "2015-09-09T21:45:48-07:00"
+++

Does your website minify its HTML and Javascript? Does it compress its transfers? Is it using a CDN for static content? Does it protect your users' privace with TLS? Does it have a firewall to defend against attackers?

If your answer to any of these questions is "no", you're definitely doing things wrong. And if you're on a budget, [Cloudflare](https://www.cloudflare.com) gives you all of these things, at absolutely **no cost**. I've racked my brain over this, and I have no idea why any informed web admin *wouldn't* use Cloudflare.

Now, before I continue, let me re-emphasize the *budget* part. Cloudflare is an amazing solution when you want to keep things cheap, but if you're playing with corporate money, you'll likely want to do something else like set up your own WAF, load balancers, and caching. I may discuss that in a future article, but in the meantime, Amazon's recently published [AWS Best Practices for DDoS
Resiliency](https://d0.awsstatic.com/whitepapers/DDoS_White_Paper_June2015.pdf) is a great read to get started with.

Setup
=====

All you have to do to get set up is create a Cloudflare account, add your website to it, and change your hosting's nameservers to Cloudflare's. I'm not gonna go through the process step-by-step, but it's very quick and painless.

Once you do this, you should take a look at your page rules. On the free Cloudflare plan, you get three. That's plenty for simple sites. We use one to ensure that visitors always get HTTPS. And another to set up the caching for all of our pages. By default, Cloudflare doesn't cache HTML, so because our site is 100% static, we use this rule to make sure it caches everything. Then whenever we update the site, we just clear the relevant pages from Cloudflare's cache with an [AWS Lambda](https://aws.amazon.com/lambda/) function and Cloudflare's REST API.

When you're done with your setup, head over to [WebPagetest](http://www.webpagetest.org/) to see how you're doing. If you don't have near-perfect scores in everything, you might still be doing things wrong!

So now what?
============

Your visitors will likely immediately notice a dramatic improvement in performance. Most requests will be served by the cache of the Cloudflare edge node located closest to your visitor. They'll be minified and compressed, and subsequent visits will serve assets from their browser cache. So in addition to your visitors getting a much zippier browsing experience, you save money on bandwidth. And you can provide TLS as easily as flipping a switch, without having to pay a certificate authority.

Cloudflare also provides some firewall functionality. It will detect several types of attacks and shut them down for you. And if it doesn't detect the attack on its own, you can raise the security level and ban users either manually or from your server using tools like [Fail2ban](http://www.fail2ban.org/) with Cloudflare's REST API.

And you can get all of this on the free plan.

So why wouldn't you use Cloudflare for a budget site? If you have a reason, please enlighten me in the comments.