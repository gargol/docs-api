---
title: "Post"
date: "2018-10-01"
meta_title: "Post Context: Ghost Themes - Documentation"
meta_description: "The post context is used in Ghost themes to render posts in a publication. Learn more about contexts and building a custom theme!"
keywords:
    - api
    - handlebars
    - themes
    - contexts
sidebar: "handlebars"
---


Use: `{{#is "post"}}{{/is}}` to detect this context

## Description

Whenever you're viewing a single site post, you're in the `post` context. The `post` context is not set on static pages, which uses the page context instead.


## Routes

The URL used to render a single post is configurable in the Ghost admin. The default is `/:slug/`. Ghost also has an option for date-based permalinks, and can support many other formats if the setting is edited in the database.


## Templates

The default template for a post is `post.hbs`, this template is required in all Ghost themes.

Additionally, you can provide a custom template for a specific post. If there is a `post-:slug.hbs` file with the `:slug` matching the post's slug this will be used instead.

For example, if you have an '1.0 Announcement' post with the url /1-0-announcement/, adding a template called `post-1-0-announcement.hbs` will cause that template to be used for the announcement post, instead of `post.hbs`.

Another option is to use a "global" custom post template. If you add a template to your theme called `custom-gallery.hbs` it will be available in a dropdown in the post settings menu so that it can be used on as many posts (or pages) as you want.

These templates exist in a hierarchy. Ghost looks for a template which matches the slug (`post-:slug.hbs`) first, then looks for a custom template (`custom-gallery.hbs` if selected in the post settings) and finally uses `post.hbs` if no slug-specific template exists and no custom template is specified.


## Data

The `post` context provides access to the post object which matches the route. As with all contexts, all of the `@blog` global data is also available.

When outputting the post, you can use a block expression (`{{#post}}{{/post}}`) to drop into the post scope and access all of the attributes. See a full list of attributes below:

### Post object attributes

- **id** - the Object ID of the post
- **comment_id** - The old, pre-1.0 incremental id of a post if present, or else the new Object ID
- **title** - the title of your site post title helper
- **slug** - slugified version of the title (used in urls and also useful for class names)
- **excerpt** - a short preview of your post content excerpt helper
- **content** - the content of the post content helper
- **url** - the web address for the post page (see url helper) and special attributes
- **feature_image** - the cover image associated with the post img_url helper
- **featured** - indicates a featured post. Defaults to `false`
- **page** `true` if the post is a page. Defaults to `false`
- **meta_title** - custom meta title for the post meta_title helper
- **meta_description**  Custom meta description for the post meta_description helper
- **published_at:** date and time when the post was published date helper
- **updated_at:** date and time when the post was last updated date helper
- **created_at:** date and time when the post was created
- **authors** - a list of post's authors
- **primary_author** - direct reference to the first author associated with with the post
- **tags** - a list of tags associated with the post
- **primary_tag** - direct reference to the first tag associated with with the post

## Helpers

Using the `{{#post}}{{/post}}` block expression is the key trick to having a happy time theming your post page. Once inside of the post, you can use any of these useful helpers (and many more) to output your post's data:

`{{title}}`, `{{content}}`, `{{url}}`, `{{authors}}`, `{{date}}`, `{{excerpt}}`, `{{img_url}}`, `{{post_class}}`, `{{tags}}`.

## Example code

```handlebars:title=post.hbs
<!-- Everything inside the #post tags pulls data from the post -->
{{#post}}

<article class="{{post_class}}">
  <header class="post-header">
    <h1 class="post-title">{{title}}</h1>
    <section class="post-meta">
      <time class="post-date" datetime="{{date format='YYYY-MM-DD'}}">
        {{date format="DD MMMM YYYY"}}
      </time>
      {{tags prefix=" on "}}
    </section>
  </header>
  <section class="post-content">
    {{content}}
  </section>
</article>

{{/post}}

```

## Special attributes

The post model is the most complex model in Ghost, and it has a couple of special attributes, which are calculated by the API rather than come direct from the DB.

### URL

URL is a calculated, created based on the site's permalink setting and the post's other properties. It exists as a data attribute, but should always be output using the special `{{url}}` helper rather than referenced as a data attribute.

That means always open a context and use `{{url}}` explicitly, and is the same for _all_ resources, but especially important because post has a data attribute present. So, **always** do `{{#post}}{{url}}{{/post}}` **never** do `{{post.url}}`.


### Comment ID

Comment ID is a special attribute provided purely for compatibility reasons. In Ghost 1.0, there was a major change, which converted post IDs from being incremental IDs to being a longer 24 character ID. However, post IDs were used as identifiers in some cases, most notably when embedding Disqus comments.

For those sites, when the IDs changed after 1.0, Disqus was no longer able to associate comments with old posts correctly. Therefore we added `{{comment_id}}` to solve that problem. We recommend always using `{{comment_id}}` as your disqus identifier, or in any other scenario where IDs are used to identify content to 3rd parties.

Example: The Disqus config in Ghost should look like this, with the code **inside** a `{{#post}}{{/post}}` block.

```handlebars:title=post.hbs
{{#post}}
<script>
  var disqus_config = function () {
    this.page.url = '{{url absolute=true}}';
    this.page.identifier = '{{comment_id}}';
  };
</script>
{{/post}}

```

For a more complete example of Disqus integration see the [Disqus code in Casper](https://github.com/TryGhost/Casper/blob/d92dda3523c27d68fa78088cd1138300b96bc7c8/post.hbs#L72-L93).


### Primary tag

Each post has a list of 0 or more tags associated with it, which can be accessed via the `tags` property and magical `{{tags}}` helper. The first tag in the list is considered more important, and can be accessed via a special `primary_tag` calculated property. This is a path expression, which points to a whole tag object, rather than a helper function.
