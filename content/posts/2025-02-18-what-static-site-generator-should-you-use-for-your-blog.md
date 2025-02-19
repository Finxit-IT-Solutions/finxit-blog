---
title: What static site generator should you use for your blog?
date: 2025-02-18T02:19:00+01:00
draft: false
---
Hello and welcome to my new blog. Let's kick it off with a technical introduction. (TL;DR at the end)

The amount of possible technologies to start a blog is sheer unlimited. It was quite clear for me, it needs to be something _simple_. With simple, I mean no database or any fancy stuff involved. I'm here to spread my word, the content doesn't change every second, so we don't need a database. No Wordpress on my watch. Also, I don't need any complex frontend, that involves a lot of JavaScript. What happened with simple HTML?

Any way, I'm drifting off. Blog. Simple. Static HTML. Ideally managed in some kind of Git repository. Requirements check.

Now, the strange part: I had never used a static site generator before. Don't ask me how that happened. My main company website for [Finxit](https://www.finxit.at/) runs on (kinda) static HTML. That content doesn't change often, has no sub-pages, so I simply never needed static site generators. If you want to run a blog, however, that obviously changes.

So I started to take a look into the world of static site generators, and - oh boy - are there many options. And half of them are JavaScript. Remember: no fancy client side shenanigans here. If you are looking for that, then this blog post is not the correct one for you.

### The beginnings

After some problems with the `ruby` setup (I'm using the [`fish`](https://fishshell.com/) shell, so it required adaptions from the standard guide), I started experimenting with [Jekyll](https://jekyllrb.com/). It kinda is (was?) the default static site generator, if you are aiming for [GitHub Pages](https://pages.github.com/). As that's where this blog is hosted, it was the first option I explored. And although the compilation took a couple of seconds each time, it worked. I even successfully setup [Sveltia CMS](https://github.com/sveltia/sveltia-cms), a headless CMS, so I could create and edit posts in the browser (we will talk about this later).

However, something simply didn't feel exactly right with me. I can't really explain it, some parts felt a little bit outdated. A lot of the available free templates, for example, hadn't had any recent commits, sometimes for years. And as I wasn't to happy with my theme selection any way, I decided to explore more options.

Don't get me wrong: `Jekyll` is by no means dead. It still is a fantastic static site generator, and if you are happy with it, you have no reason to migrate away from it.

### Introducing Hugo

I can't remember exactly, how I learned about [Hugo](https://gohugo.io/), but it was my next contender on the list.

Reddit had a split opinion on this. You can't tell me that they are not the same person.
![Two people on Reddit exclusively recommending Jekyll and Hugo](/assets/posts/reddit_opinion_jekyll_hugo.png "You can't tell, that this is not the same person")

So I gave `Hugo` a try, and honestly, it felt a lot better. One pro argument you often read about `Hugo` is, that is better for big sites, like 1k pages+. However, that speed benefit trickles down to very small sites too. `hugo server` starts instantly for me, while `bundle exec jekyll serve` took a couple seconds for only two pages. This might not sound like a lot, but each change, each hot reload took a couple of seconds, so this was a huge improvement.

Furthermore, `Hugo` comes with a lot of stuff build in, where you would need plugins with `Jekyll`. On the downside: it doesn't support plugins at all. However, I was still able to integrate the `sveltia-cms` via a [`Hugo` module](https://github.com/privatemaker/headless-cms). No idea what that is, but it works really well.

From a technical point of view: `Hugo` uses `Go templates` as it's templating engine, which might be a benefit if you are doing a lot of Cloud Native stuff and are already familiar with it. `Jekyll` - in comparison - uses liquid, which was created by Shopify. (TIL)

### The CMS to rule the Git(Hub) repo

Remember when I said no fancy stuff? Well, I went a little bit fancy. Having a CMS is still a major benefit for blogging, especially on mobile. Fortunately, [Dylan Beattie blogged about Sveltia CMS just a couple of days ago](https://dylanbeattie.net/2025/02/13/sveltiacms-jekyll-and-github-pages.html), and I really liked it. `sveltia-cms` is a headless CMS (no backend*), that will simply push your changes into your Git repo. Did I say no backend? Well, almost no backend. (Un)fortunately you only need a [Cloudflare worker](https://github.com/sveltia/sveltia-cms-auth) for the OAuth Authentication with GitHub.

Last but not least, Sveltia CMS needs to be configured to edit / save the correct files. The documentation is the biggest drawback at the moment, as it mostly simply doesn't exist. However, as Sveltia CMS is a drop-in replacement for Decap CMS, you can use their [documentation](https://decapcms.org/docs/intro/).

I went with the following configuration (add to `hugo.toml`). If you want to use it like that, you need to replace some values like `site_url`, `repo` or `base_url`. Check out the links at the end of the post for the used tutorials / guides.

```toml
  [params.headless_cms]
    engine = "sveltia"
    site_url = "https://blog.finxit.at"
    media_folder = "static/assets/" # use static folder, till I figured out the asset pipeline

    [params.headless_cms.backend]
      name = "github"
      repo = "Finxit-IT-Solutions/finxit-blog"
      branch = "main"
      base_url = "https://sveltia-cms-auth.office-f07.workers.dev"

    [params.headless_cms.collections.blog]
      name = "blog"
      label = "Blog Post"
      create = true
      folder = "content/posts"
      slug = "{{year}}-{{month}}-{{day}}-{{slug}}"

      [params.headless_cms.collections.blog.sortable_fields]
        fields = ["title", "date"]

        [params.headless_cms.collections.blog.sortable_fields.default]
          field = "date"
          direction = "descending"

      [[params.headless_cms.collections.blog.fields]]
        label = "Title"
        name = "title"
        widget = "string"

      [[params.headless_cms.collections.blog.fields]]
        label = "Date"
        name = "date"
        widget = "datetime"
        default = "{{now}}"
        format = "YYYY-MM-DDTHH:mm:ssZ"

      [[params.headless_cms.collections.blog.fields]]
        label = "Body"
        name = "body"
        widget = "markdown"

      [[params.headless_cms.collections.blog.fields]]
        label = "Draft"
        name = "draft"
        widget = "boolean"
        default = true
```

### Technical details

I used the following combination of frameworks, tools and services:

- [Hugo](https://gohugo.io/)
- [Maverick Theme](https://themes.gohugo.io/themes/maverick/)
- [Headless CMS Hugo Module](https://github.com/privatemaker/headless-cms)
- [Sveltia CMS](https://github.com/sveltia/sveltia-cms)
- [Sveltia CMS Authenticator](https://github.com/sveltia/sveltia-cms-auth)
- [GitHub Pages](https://pages.github.com/)
- [Cloudflare Workers](https://workers.cloudflare.com/)
