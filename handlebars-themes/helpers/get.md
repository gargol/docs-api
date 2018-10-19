---
title: "get"
date: "2018-10-01"
meta_title: "Handlebars Theme Helpers: get"
meta_description: "Make custom queries to the Ghost API using the get helper and fetch publicly available data. Read more about Ghost themes! 👻"
keywords:
    - api
    - handlebars
    - themes
    - helpers
sidebar: "handlebars"
---

Usage: `{{#get "posts"}}{{/get}}`

## Description

`{{#get}}` is a special block helper that makes a custom query to the Ghost API to fetch publicly available data. These requests are made server-side, before your templates are rendered. This means you can fetch additional data, separate from what is provided by default in each [context](/api/handlebars-themes/context/).

In its most basic form it can perform a 'browse' query to create a block of data that represents a list of your posts, authors (users) or tags. That block of data can then be iterated over using the `{{#foreach}}` helper.

It can also be used to perform a 'read' query that fetches one specific **author**, **post** or **tag** if the relevant *resource field* - E.g. **id** or **slug** is provided as an attribute.

**Please note**: This is a powerful tool, that has the potential to be overused and cause problems on a site.

### Simple Examples

A basic request for posts, this will fetch 15 posts from the API including their related tags and authors.

```handlebars
{{#get "posts" include="tags,authors"}}
    {{#foreach posts}}
        {{title}}
    {{/foreach}}
{{/get}}

```

A basic request for a single post with id of 2, including its related tags and author data, using a block parameter.

```handlebars
{{#get "posts" id="2" include="tags,authors" as |post|}}
    {{#post}}
        {{title}}
    {{/post}}
{{/get}}
```

Fetch all tags, and output them using the tags helper:

```handlebars
{{#get "tags" limit="all"}}{{tags}}{{/get}}
```

## Usage

The `{{#get}}` helper has many more options than most helpers, the following section walks through the various options and how they can be used.

## Parameters

The first parameter passed in is the name of the resource that you want to query. This can be either `"posts"`, `"tags"` or `"users"` (authors).

**posts** - only published posts can be retrieved
**tags** - currently all tags can be retrieved
**users** - only active users can be retrieved

Example:

```handlebars
{{#get "posts"}}
    {{! Loop through our posts collection }}
    {{#foreach posts}}
        {{title}}
    {{/foreach}}
{{/get}}
```

## Block Parameters

As with the `{{#foreach}}` helper, it is possible to use block parameters to rename your returned data collection to make it easier to reference or more distinguishable.

> Note: Block Params are entered between pipe symbols, which are vertical bars: `|`

Usage: `as |featuredposts pagination|`

The `{{#get}}` helper supports two parameters entered here. The first entry in the *pipe* refers to your returned data collection. The second entry refers to your pagination object.

Example using block parameters:

```handlebars
{{#get "posts" as |articles pages|}}
    {{! Loop through our articles collection }}
    {{#foreach articles}}
        {{title}}
    {{/foreach}}
    {{! Use our pages (pagination) object }}
    {{pages.total}}
{{/get}}
```

In the example above we are fetching **posts** which we will then refer to **as articles** in our loop and a pagination object called **pages**.

## Using {{else}}

All block helpers support the `{{else}}` helper, which allows you to output content when the first block doesn't match. In the case of the `{{get}}` helper, this only happens if there is an error and is mostly useful for debugging whilst developing.

If you want to output different content when there are no results, you will need to use `{{else}}` with the `{{#foreach}}` helper.

```handlebars
{{#get "posts" filter="featured:true"}}
    {{! Loop through our featured posts }}
    {{#foreach posts}}
        {{title}}
    {{else}}
    {{! If there are no featured posts}}
       <p>No posts!</p>
    {{/foreach}}
{{else}}
  <p class="error">{{error}}</p>
{{/get}}
```

## Attributes

The attributes that can be passed to the `{{#get}}` helper exactly match up to the query parameters that you can use in the Ghost JSON API. These allow you to specify the data to look for and how much data is returned. If you're making a 'browse' request (fetching multiple items) you can use any of these attributes and if you're making a 'read' request (fetching a single item by **id** or **slug** only **include** is available.

### *limit*

Specify the size of your collection
Allowed values: positive integer and 'all'
Default value: 15

It is possible to use the global "posts per page" setting which is **5** by default or it can be configured via the active theme's `package.json` file, this global value is available via the `@config` global as `@config.posts_per_page`.

Examples:

```handlebars
{{! Fetch the 20 most recently published posts }}
{{#get "posts" limit="20"}}{{/get}}

{{! Fetch all published posts }}
{{#get "posts" limit="all"}}{{/get}}

{{! Use the posts_per_page setting}}
{{#get "posts" limit=@config.posts_per_page}}{{/get}}
```

### *page*

The resulting collection from the `{{#get}}` query may be paginated, therefore you can choose which page of that collection you want to fetch.

Example:

```handlebars
{{! Fetch the 4th page of results }}

{{#get "posts" limit="5" page="4"}}{{/get}}
{{! In this case where limit = 5, we are accessing posts 16 - 20 }}
```

### *order*

Specify how your data is ordered before being returned. You can choose any valid resource *field* in ascending (`asc`) or descending (`desc`) order.

Examples:

```handlebars
{{! Fetch the oldest 5 posts }}
{{#get "posts" limit="5" order="published_at asc"}}{{/get}}

{{! Fetch the 5 most recently published posts }}
{{#get "posts" limit="5" order="published_at desc"}}{{/get}}

{{! Fetch posts in alphabetical order of title ([0-9], A->Z) }}
{{#get "posts" limit="5" order="title asc"}}{{/get}}
```

### *include*

When making an API request, the resulting response will only contain base data from the Resource itself.

A Resource may have additional related data that can be included to expand your collection.

Base Resource data:
* **Post**
* **Tag**
* **User**

There can be multiple *includes* separated by a comma.

The *Post* resource by default has a tags and authors array. These can be expanded to include more information.

Include options for *Post*: "authors" - expands authors, "tags" - expands tags.

The *User* and *Tag* resources can be expanded to include the post count for each resource.

Include options for *User* and *Tag*: "count.posts"

Note: If you include count.posts you can use it to **order** your collection.

Examples:

```handlebars
{{! Fetch posts with author }}
{{#get "posts" limit="5" include="authors"}}
    {{#foreach posts}}
        <span>Written by: {{authors}}</span>
    {{/foreach}}
{{/get}}

{{! Fetch posts with author and tags }}
{{#get "posts" limit="5" include="authors,tags"}}
    {{#foreach posts}}
        <p>Written by: {{authors separator=", "}}</p>
        <p>keywords: {{tags separator=", "}}</p>
    {{/foreach}}
{{/get}}
```

### *filter*

This is a powerful tool that allows you to make a complex logic-based queries on the data to fetch. In its most basic form, you can choose to fetch posts that meet a simple boolean logic such as *featured posts*:

```handlebars
{{#get "posts" limit="all" filter="featured:true"}}
    {{#foreach posts}}
        <a href="{{slug}}">{title}}</a>
    {{/foreach}}
{{/get}}
```

Filtering can be used to specify multiple rules using *and* or *or** and can check for booleans, match against strings, look for items within a group, and many other things. For a full breakdown of the filtering syntax and how to use it, please see the Filter documentation in the API docs.

#### Passing data to `filter`

When used with the `{{#get}}` helper, filters can be passed data which is already available within your theme template. For example, if in your `post.hbs` file you wanted to get 3 more posts by the author of the current post, you can do so as shown here:

```handlebars
{{#post}}
    <h3><a href="{{url}}">{{title}}</a></h3>
    <section class="author-meta">
    <p>Post by: {{primary_author}}</a></p>

    {{#get "posts" filter="authors:{{primary_author.slug}}+id:-{{id}}" limit="3"}}
        <p>More posts by this author:
        <ol>
            {{#foreach posts}}
            <li><a href="{{url}}">{{title}}</a></li>
            {{/foreach}}
        </ol>
        </p>
    {{/get}}
{{/post}}
```

In this example, we look for posts with the `primary_author` (which is an alias of `authors[0].slug`) that matches the current author and that does not have the same `id` as the current post, so that we will get 3 different posts.

In some cases, you'll need to wrap the data you pass in with quotes, for example if you pass a title rather than a slug, because it contains spaces, and also if you want to use dates.

```handlebars
{{#post}}
    {{#get "posts" filter="published_at:<='{{published_at}}'+id:-{{id}}" limit="3"}}
    ...
    {{/get}}
{{/post}}
```

Also be aware that, if you want to filter based on dates, you need to use the data attributes e.g.`{{published_at}}`, not the `{{date}}` helper, as helper functions do not get called inside of a filter.

#### Filtering by primary tag
The primary_tag is a virtual post property and we support filtering by this property

```handlebars
{{#post}}
    {{#get "posts" filter="primary_tag:{{primary_tag.slug}}" limit="3"}}
        {{#foreach posts}}
            <li><a href="{{url}}">{{title}}</a></li>
        {{/foreach}}
    {{/get}}
{{/post}}
```

In this example we fetch 3 posts which have the same primary tag as the current post.

#### Filtering by primary author
The primary_author is a virtual post property and we support filtering by this property.


```handlebars
{{#post}}
    {{#get "posts" filter="primary_author:{{primary_author.slug}}" limit="3"}}
        {{#foreach posts}}
            <li><a href="{{url}}">{{title}}</a></li>
        {{/foreach}}
    {{/get}}
{{/post}}
```

## Limitations

There are a few known limitations with the `{{#get}}` helper at the moment:

- `{{#get}}` helpers may not work correctly when nested
- Other async helpers may not work when nested inside a get block (you may see an **aSyNcId_###** error on your page).
- you cannot yet filter on count.posts
- ordering alphabetically may be case sensitive depending on database
- `{{pagination}}` won't output anything sensible when used inside  `{{#get}}` block - because the get helper can only fetch existing data, it doesn't create a /page/2/ version of the data you are fetching.
