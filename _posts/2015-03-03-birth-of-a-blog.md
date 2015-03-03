---
title: The Birth of a Blog
layout: post
summary: An obligatory welcome post.
---

I'm starting a blog[^1] for the obvious reasons. It's a great way to share all the rad stuff I've been doing, when bragging to my friends just doesn't cut it. All the cool kids are blogging these days, anyway; I can't be cool if I'm not blogging, right?

Making a blog is also an excellent way to put to use all the domains that I keep buying.

In particular, I made *this* blog because I wanted to make a site that catered to my hacking habits. Wordpress? Please. The site is hosted on a small [DigitalOcean][do] VPS (thanks to credit from [GitHub's Student Pack][student]), statically powered by [Jekyll][jekyll].

Thanks to Jekyll, I can compose all of my pages and posts in Vim[^2] as Markdown files, and when I'm ready to deploy an update, I just `git push` my blog's repo up to the VPS. There's a [post-receive hook][hook] there that tells Jekyll to chop, shred, bake and broil my assortment of posts, settings, and templates into the digital delicacy you see here. Apache serves it up, and we're ready to go. I even found a Jekyll `fortune` [plugin][fortune], for a build-dependent error on the [404 page][404].

Even though I'm not hosting the site with [Pages][pages], I'm using a [GitHub repo][mirror] as a backup mirror just in case everything goes south. Everything's open source; go check it out! The result wouldn't be so *delectable* if not for [John Otander][john]'s awesome [Pixyll][pixyll] theme for Jekyll. Thanks for being cool, John.

[^1]: My ego isn't big enough to call it a [blag][blag].
[^2]: My Vim dotfiles are mirrored [here][dotfiles].

[do]: http://digitalocean.com
[student]: https://education.github.com/pack
[jekyll]: http://jekyllrb.com
[hook]: http://jekyllrb.com/docs/deployment-methods/#git-post-receive-hook
[fortune]: https://github.com/jakhead/jekyll-fortune
[404]: /this-is-not-a-real-page
[john]: http://johnotander.com/
[pixyll]: http://pixyll.com/
[pages]: https://pages.github.com/
[mirror]: https://github.com/ProtractorNinja/protractor-blog
[blag]: http://blog.xkcd.com
[dotfiles]: https://github.com/ProtractorNinja/dotfiles
