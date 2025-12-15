---
title: "Migrating to Zensical"
icon: material/file-move
draft: true
#date:
#  created: 2025-12-13
#  updated: 2025-12-14
categories:
  - how-to
  - git
  - www
  - code
  - web
  - mkdocs
  - zensical
---

# :material-file-move: Migrating to Zensical

This post covers discovering [the decision by the mkdocs-material team to move away from mkdocs itself](https://squidfunk.github.io/mkdocs-material/blog/2025/11/05/zensical/) as a core dependancy, and describes how the migration went for me. In short, I see why they went this route, and I'm excited to see how well the new documentation engine works.

!!! abstract "What's Maintenance Mode?"

    In December of 2025 I kept running into a bug where the local instance of mkdocs-material would not render changes live anymore. This made it a rough experience to write and visualize the changes in my dev environment before publishing them. At a certain point I was even just pushing changes before seeing them render, because it was also taking a long time to reload per-change. Finally, I had to check in on the GitHub issues page. To my surprise, the project had two open issues and hardly any PR's. This caught my attention, and [thankfully, the pinned issue had all the details](https://github.com/squidfunk/mkdocs-material/issues/8523).

    To summarize, mkdocs-material is built on top of mkdocs. mkdocs itself has an uncertain enough future, that the mkdocs-material team has opted to build Zensical to replace mkdocs as a core dependancy.

    > MkDocs must be considered a supply chain risk, since it's unmaintained since August 2024. It has seen no releases in over a year and is accumulating unresolved issues and pull requests. These developments have forced us to cut our ties to MkDocs as a dependency.

    - [Maintenance Mode Notice](https://github.com/squidfunk/mkdocs-material/issues/8523)
    - [Zensical Announcement and Reasoning Behind the Decisison](https://squidfunk.github.io/mkdocs-material/blog/2025/11/05/zensical/)

So that was it. I had a free window of time to check out the new documentation engine (I'll just call it an engine for now since the mkdocs-material team is building the entire stack) and it works exactly the same, only better. Blogs were the only major feature / plugin not ported over yet, since this is within the first few months of Zensical being available to use in production, I was shocked that's about the only thing not already supported on the feature parity list. I believe the social cards feature was also missing, but just like blogs, this is on the roadmap.

I had also been wondering how to revise the structure of my notes and GitHub pages site for a while, and ultimately, after trying a few things in the default site you can spin up with `zensical new .; zensical serve`, I decided now's the time to get this done and make these changes.


## Reasoning

Part of me just wanted to learn about the new documentation generator, and see how it felt to work with it, but I can point to three main reasons I jumped in and migrated immediately:

- Vastly improved search and render time for live note taking (I rely on this since these are meant to replace my notes)
- `mkdocs`, the core dependancy, has seemingly been abandoned, so this needed to happen eventually
- Wanted to revise parts of the site that were "stuck" in the blog structure, and needed a reason to spend time doing so


## Compatability

So long as you don't rely on something like the blog plugin that isn't supported just yet, Zensical was designed to be 100% compatible with `mkdocs.yml` files. The switch was in fact seamless, and I tested this by simply removing the blog components of the site and loading the YAML file, which had a number of the features enabled.

!!! quote "[Compatability](https://zensical.org/compatibility/)"

    Zensical is designed for seamless compatibility with Material for MkDocs. In many cases, you can install Zensical and build your projects immediately - no changes to configuration or content.

I archived the mkdocs-material version of my site in a [snapshot branch](https://github.com/straysheep-dev/straysheep-dev.github.io/tree/straysheep-dev/mkdocs-archival-snapshot) before working on the Zensical migration in another branch.


## Porting YAML to TOML

I read up on all of the things Zensical has on the roadmap, and one of the notes was eventually `mkdocs.yml` files will only be supported via a module. I've also never worked with TOML outside of changing settings in specific tools, so I ended up porting the configuration section-by-section. AI may be able to do this for you, but you should take care to review each line anyway. At the time of writing GPT 5.2 didn't know about Zensical and needed to pull that information from the web.

!!! note ""

    The day before doing this I updated the documentation on my CI/CD notes for linting in both YAML and JSON, so I was curious about TOML. I'll be adding those details to the [CI/CD](./cicd.md) and [Resources](../resources.md) pages.


## Metadata or Front Matter

This is the main issue I bumped into during the migration. Since I won't be leveraging any kind of blog plugin, I discovered the date fields in the front matter section of each page caused Zensical to fail to render and parse those pages correctly.

!!! tip "The Easiest Fix"

    I considered moving the metadata into the page itself, but really, my main use-case for this site is searching it for things I already wrote. The site structure itself was already being changed with this migration, so as long as I keep the `created` and `updated` dates in that front matter section, but comment them out, Zensical loads them just fine.

    This also allows me to move them back to a blog module later if I really decide to go that route again in the future.


## Fixing (Updating) the Logo

As soon as I got all of the pages to load, I browsed around and eventually tried changing the theme from dark to light. I realized the logo would not display correctly on the new header / menu section, since it no longer was a solid band of color, and that logo file ends up rendered as a raster `img`, not a vector graphic.

Initially I tried the option using two different logo files via [overrides](https://zensical.org/docs/customization/?h=overrides#configuring-overrides) on `partials/logo.html` and `stylesheets/extra.css`. This didn't work no matter what I tried, and always showed *two* "failure to load" images where the logo should have been.

- [Support for different custom logo for each palette scheme Issue:6176](https://github.com/squidfunk/mkdocs-material/issues/6176)
- [Different Logo-Icon for Dark Mode and Light Mode Issue:2729](https://github.com/squidfunk/mkdocs-material/discussions/2729)

I pivoted, and found a better solution. The key was [this advice](https://github.com/squidfunk/mkdocs-material/discussions/2097#discussioncomment-170031):

!!! quote ""

    Remove all colors from the SVG logo, so you can use CSS to set the fill property (as with the Material logo)

I had to remove the `fill:` key from my SVG entirely. Once I did this, the icon and logo both successfully adapted based on light or dark themes.

- [How to make SVGs respect light/dark mode? Issue:2097](https://github.com/squidfunk/mkdocs-material/discussions/2097)
- [mkdocs-material: logo.svg](https://github.com/squidfunk/mkdocs-material/blob/master/src/templates/.icons/logo.svg?short_path=763eb2c)


## More Control

Things simply "looked" nicer, and relearning how everything works under the hood helped me move the pieces around, make the changes I needed to, and note for myself a few things going forward:

- The `nav` key in `zensical.toml` lets you decide what's visible in the nav bar, and in what order things appear
- How overrides and custom CSS works (or doesn't)
- The limitations of what you can do with a static site (I still would like to redirect from `/blog` to `/notes`, since I changed that)
    - This is possible via HTML, but I want to research this more or wait for an official module first
- This was also an exercise in making a sweeping change across all pages at once, it's good to know how that would go


## Publishing

The last thing to do once I fixed the metadata of each page and got everything moved around and working as intended locally, was publish.

!!! tip ""

    [Zensical: Publishing Your Site](https://zensical.org/docs/publish-your-site/)

Even this early in Zensical's development, it deployed without issue to GitHub pages using all the existing methods to deploy a page on a custom domain. I just had to drop in the updated CI file using the example Zensical's documentation (above) provides.

Now everything lives here, which you would know if you're reading this. If you have documentation built on mkdocs-material, you have roughly until November of 2026 to make the move; this is just one anecdote showing it's relatively easy (keeping in mind my setup isn't very complex). The two things I believe are missing are the previously mentioned blog module, and social cards, which I don't believe are active *yet*. Otherwise, this will work better and is the way to go for day-to-day editing and publishing.
