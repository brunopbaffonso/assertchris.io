---
slug: 2018-10-24-building-something-new
date: 2018/10/24
title: Building something new
intro: I’ve been building this blog, for a few days now, and it’s been a fun experience. This is partly because it’s an asynchronous application, and partly because it uses /a lot/ of preprocessing.
---

I’ve been building this blog, for a few days now, and it’s been a fun experience. This is partly because it’s an asynchronous application, and partly because it uses /a lot/ of preprocessing.

I thought it would be interesting for me to describe how it is put together, in an ad-hoc sort of series. In the series, I’ll show bits of code and links to libraries I’ve used. I’ll talk about why I’ve chosen to do things in certain ways, and what I could improve (short of deleting everything and starting over with a mature framework).

I’m not going to start today, because there’s a bit of work involved in making sure code snippets are rendered nicely. Instead, I’ll show you a tiny bit of code (to whet your appetite, I’m sure), and also to give me something to highlight.

This is the code which renders the introductory paragraphs, on the home page:

```php
namespace App\Component;

use function App\render;

class PostIntroComponent
{
    public function __invoke($props)
    {
        $date = $props->date->format("Y-m-d");

        return (
            <div className={"mb-6"}>
                <h2>{$props->title}</h2>
                <blockquote>Published on {$date}</blockquote>
                {$props->children}
                <a href={$props->uri} className={"mt-3 text-blue-dark"}>
                    Read more...
                </a>
            </div>
        );
    }
}
```

> This is from `app/Component/PostIntroComponent.pre`

You’re probably a little familiar with the PHP language (since that’s the community where this post is likely to be shared). Maybe not enough to recognise that this isn’t 100% “normal” PHP code.

This strange HTML-like syntax is inspired by JSX, which is a similar HTML-like Javascript syntax. I made it because I’ve used XHP before; and because I’ve done a lot of ReactJS development recently.

I also make it because I was looking to build a simple compiler for integration into [preprocess.io](https://preprocess.io). This file is preprocessed, by thanks to the `.pre` extension, and Composer’s autoloader.

In fact, this whole blog is built on preprocessed code. Why? Because I want to see what I can build, using a technology that 99% of vocal PHP developers (read: people on Reddit and Twitter) scoff at. I want to show that preprocessing works well and is popular in other languages.

That it is a tool with tradeoffs and benefits. Not just a toy or an object of ridicule. That the concept has as much right to exist and the messy and useful language I have implemented it in.

Ok, enough for now.
