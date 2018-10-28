---
slug: "2018-10-28-building-a-blog-posts"
date: "2018/10/28"
title: "Building a blog: Posts"
intro: "This blog doesn’t have a database yet. It may look like it is dynamic – may adjust as I add blog posts – but it is more like a flat file CMS. Only without the CMS…"
---

-   [Part 1: introduction](/post/2018-10-24-building-something-new)
-   Part 2: posts

This blog doesn’t have a database yet. It may look like it is dynamic – may adjust as I add blog posts – but it is more like a flat file CMS. Only without the CMS…

You see, I wanted to build something quickly, At first, I would only be publishing blog posts, so I could afford to use something simple. It wasn’t long before I was writing Markdown files and reading them from the file system.

I started with this format:

```md
# A post title

Some introductory text, to display on the home page.

---

# A post title (which was repeated)

Some introductory text, to display on the home page (repeated).

Now, the rest of the post...
```

> This was from `posts/yyyy-mm-dd-a-post-title.md`

I figured; it would be simple enough to read the contents and split it on `---`. It meant I needed to maintain a list of file particulars in the PHP code of my app, but that was ok.

After a while, this list got annoying. There were also the problems of repetition and specificity. I was repeating the title and first paragraph every time (repetition), and I couldn’t target the titles for different styles on different pages (specificity).

Then [Sebastian De Deyne suggested](https://twitter.com/sebdedeyne/status/1055027799659081728) I use a [Spatie package](https://github.com/spatie/yaml-front-matter), which would allow a kind of meta data:

```md
---
title: A post title
slug: yyyy-mm-dd-a-post-title
date: yyyy-mm-dd
intro: Some introductory text, to display on the home page.
---

Some introductory text, to display on the home page (repeated).

Now, the rest of the post...
```

> This is from `posts/yyyy-mm-dd-a-post-title.md`

I still need to repeat some text (which, to be honest, was always optional), but now I can treat the `intro` and `title` as the meta data they always were.

Furthermore, I don’t need to store the date and slug (which is a common name for a URL-friendly identifier) in a PHP array. The next step is to load these files:

```php
async public static function find($slug)
{
    static $posts;

    if (!$posts) {
        $posts = [];
    }

    if (!isset($posts[$slug])) {
        $base = realpath(.."/../../posts");
        $file = realpath(.."/../../posts/{$slug}.md");

        if (!$slug || strncmp($file, $base, strlen($base)) !== 0) {
            throw new Exception("Nope");
        }

        $posts[$slug] = static::hydrate(await get($file));
    }

    return $posts[$slug];
}
```

> This is from `app/Model/Post.pre`

We begin by defining a `$posts` array. I don’t generally use the `static` keyword like this, but we need to memoize the posts as this function will be called in a single script execution, for all visitors. This is an asynchronous site…

Then, if this post hasn’t already been loaded; we check to see that provided slug isn’t some sort of direct traversal nonsense (in case someone’s trying to load a file outside of the posts directory).

> There are other reasons they’d fail to do this, but it’s always good to think about the security of each function independently.

If the file is within the `posts` directory, we try to load it using the [`Amp\File\get`](https://github.com/amphp/file) function. This function returns a coroutine, so it’s appropriate to use[`Pre\Async`’s `async` and `await`](https://github.com/preprocess/pre-async) keywords.

It may interest you to know what the generated code looks like:

```php
public static function find($slug): \Amp\Promise
{
    return \Amp\call(function () use (&$posts, &$slug, &$base, &$file) {
        static $posts;

        if (!$posts) {
            $posts = [];
        }

        if (!isset($posts[$slug])) {
            $base = realpath(__DIR__ . "/../../posts");
            $file = realpath(__DIR__ . "/../../posts/{$slug}.md");

            if (!$slug || strncmp($file, $base, strlen($base)) !== 0) {
                throw new Exception("Nope");
            }

            $posts[$slug] = static::hydrate(yield get($file));
        }

        return $posts[$slug];
    });
}
```

> This is from the generated `app/Model/Post.php`

It’s not too much extra code, but I’m glad I never have to type that boilerplate out again…

Next up, we need to pass the file contents to the `hydrate` method:

```php
private static function hydrate($content)
{
    static $converter;

    if (!$converter) {
        $converter = new CommonMarkConverter();
    }

    $document = YamlFrontMatter::parse($content);

    $intro = $document->intro;
    $body = $document->body();

    return new static([
        "slug" => $document->slug,
        "date" => new Carbon($document->date),
        "title" => $document->title,
        "introMarkdown" => $intro,
        "introMarkup" => $converter->convertToHtml($intro),
        "bodyMarkdown" => $body,
        "bodyMarkup" => $converter->convertToHtml($body),
    ]);
}
```

> This is from `app/Model/Post.pre`

Using the front-matter parser, and The [PHP League’s CommonMark](http://commonmark.thephpleague.com/) converter; we get a new instance of the `Post` class:

```php
class Post
{
    private $data = [];

    private function __construct($data)
    {
        $this->data = $data;
    }

    public function __get($key)
    {
        if (isset($this->data[$key])) {
            return $this->data[$key];
        }

        return null;
    }

    public function __set($key, $value)
    {
        $this->data[$key] = $value;
    }

    // ...snip
}
```

> This is from `app/Model/Post.pre`

There’s a little more to this class, but we’ll look at it next time…
