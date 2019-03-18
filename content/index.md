---
title: "Content API"
date: "2019-01-09"
meta_title: "Ghost Content API Documentation"
meta_description: "Ghost's RESTful Content API delivers published content to the world and can be accessed by any client to render a website. Read more on Ghost Docs."
keywords:
    - "headless cms"
    - "javascript"
    - "ghost api"
---

Ghost's RESTful Content API delivers published content to the world and can be accessed in a read-only manner by any client to render in a website, app or other embedded media.

Access control is managed via an API key, and even the most complex filters are made simple with our SDK. The Content API is designed to be fully cachable, meaning you can fetch data as often as you like without limitation.


---


## API Clients

### JavaScript Client Library

We've developed an [API client for JavaScript](https://docs.ghost.org/api/javascript/), that will allow you to quickly and easily interact with the Content API.
The client is an advanced wrapper on top of our REST API - everything that can be done with the Content API can be done using the client, with no need to deal with the details of authentication or the request & response format.

### Handlebars

You can upgrade your Ghost theme to use the v2 Content API by specifying the `ghost-api` version in the `engines` field of your package.json. See the [handlebars reference](/api/handlebars-themes/packagejson/) for an example.


---


## Authentication

### Url

`https://{admin_domain}`

Your admin domain can be different to your main domain.
Using the correct domain and protocol are critical to getting consistent behaviour, particularly when dealing with CORS in the browser.
All Ghost(Pro) blogs have a `*.ghost.io domain` as their admin domain and require https.

### Path & Version

`/ghost/api/{version}/content/`

Each API is prefixed with the same path, followed by a specific version. Version strings are required and always start with `v`. The [api versioning](/faq/api-versioning/) guide explains the current available versions and stability index.

### Key

`?key={key}`

Content API keys are provided via a query parameter in the url. These keys are safe for use in browsers and other insecure environments, as they only ever provide access to public data. Sites in private mode should consider where they share any keys they create. 

The Content API URL and key can be obtained by creating a new `Custom Integration` under the **Integrations** screen in Ghost Admin. 

![Get a Ghost Content API key](/images/apikey.png)


### Working Example

```bash:title=cURL example
# Real endpoint - copy and paste to see!
curl "https://demo.ghost.io/ghost/api/v2/content/posts/?key=22444f78447824223cefc48062"
```

---

## Endpoints

The Content API provides access to Posts, Pages, Tags, Authors and Settings. All endpoints return JSON.

<table class="table">
<tbody>
<tr>
  <th>Verb</th>
  <th>Path</th>
  <th>Method</th>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/posts/</td>
  <td>Browse posts</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/posts/{id}/</td>
  <td>Read a post by ID</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/posts/slug/{slug}/</td>
  <td>Read a post by slug</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/authors/</td>
  <td>Browse authors</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/authors/{id}/</td>
  <td>Read an author by ID</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/authors/slug/{slug}/</td>
  <td>Read a author by slug</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/tags/</td>
  <td>Browse tags</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/tags/{id}/</td>
  <td>Read a tag by ID</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/tags/slug/{slug}/</td>
  <td>Read a tag by slug</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/pages/</td>
  <td>Browse pages</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/pages/{id}/</td>
  <td>Read a page by ID</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/pages/slug/{slug}/</td>
  <td>Read a page by slug</td>
</tr>
<tr>
  <td>GET</td>
  <td width="300">/settings/</td>
  <td>Browse settings</td>
</tr>
</tbody>
</table>

The Content API supports two types of request: Browse and Read.
Browse endpoints allow you to fetch lists of resources, whereas Read endpoints allow you to fetch a single resource.


---


## Resources

The API will always return valid JSON in the same structure:

```
{
    "resource_type": [{
        ...
    }],
    "meta": {}
}
```

The meta object contains [pagination](#pagination) information for browse requests.

The singular exception is [settings](#settings), which only ever returns a single object.


### Posts

The most common gotcha when fetching posts from the Content API is not using the [include](#include) parameter to request related data such as tags and authors.
By default, the response for a post will not include these:

```json
{"posts": [{
    "id":"5b7ada404f87d200b5b1f9c8",
    "uuid":"22af052d-2bc1-4306-96d1-667584c797c7",
    "title":"Welcome to Ghost",
    "slug":"welcome",
    "html":"<p>👋 Welcome, it's great to have you here.</p><p>We know that first impressions are important, so we've populated your new site with some initial <strong>getting started</strong> posts that will help you get familiar with everything in no time. This is the first one!</p>",
    "comment_id":"5b7ada404f87d200b5b1f9c8",
    "feature_image":"https://casper.ghost.org/v2.0.0/images/welcome-to-ghost.jpg",
    "featured":false,
    "page":false,
    "meta_title":null,
    "meta_description":null,
    "created_at":"2018-08-20T15:12:00.000+00:00",
    "updated_at":"2018-08-20T15:12:00.000+00:00",
    "published_at":"2018-08-20T15:12:06.000+00:00",
    "custom_excerpt":"Welcome, it's great to have you here.\nWe know that first impressions are important, so we've populated your new site with some initial getting started posts that will help you get familiar with everything in no time.",
    "excerpt":"Welcome, it's great to have you here.\nWe know that first impressions are important, so we've populated your new site with some initial getting started posts that will help you get familiar with everything in no time.",
    "codeinjection_head":null,
    "codeinjection_foot":null,
    "og_image":null,
    "og_title":null,
    "og_description":null,
    "twitter_image":null,
    "twitter_title":null,
    "twitter_description":null,
    "custom_template":null,
    "primary_author":null,
    "primary_tag":null,
    "url":"https://demo.ghost.io/welcome/",
    "canonical_url":null
}]}
```

Posts allow you to include `authors` and `tags` using `?include=authors,tags`, which will add an `authors` and `tags` array to the response, as well as both a `primary_author` and `primary_tag` object.

#### Working Example

```bash:title=cURL example
# Real endpoint - copy and paste to see!
curl "https://demo.ghost.io/ghost/api/v2/content/posts/?key=22444f78447824223cefc48062&include=tags,authors"
```

Returns:

```json
{"posts": [{
    "id": "5b7ada404f87d200b5b1f9c8",
    "uuid": "22af052d-2bc1-4306-96d1-667584c797c7",
    "title": "Welcome to Ghost",
    "slug": "welcome",
    "html": "<p>👋 Welcome, it's great to have you here.</p><p>We know that first impressions are important, so we've populated your new site with some initial <strong>getting started</strong> posts that will help you get familiar with everything in no time. This is the first one!</p><p><strong>A few things you should know upfront</strong>:</p><ol><li>Ghost is designed for ambitious, professional publishers who want to actively build a business around their content. That's who it works best for. </li><li>The entire platform can be modified and customised to suit your needs. It's very powerful, but does require some knowledge of code. Ghost is not necessarily a good platform for beginners or people who just want a simple personal blog. </li><li>For the best experience we recommend downloading the <a href=\"https://ghost.org/downloads/\">Ghost Desktop App</a> for your computer, which is the best way to access your Ghost site on a desktop device. </li></ol><p>Ghost is made by an independent non-profit organisation called the Ghost Foundation. We are 100% self funded by revenue from our <a href=\"https://ghost.org/pricing\">Ghost(Pro)</a> service, and every penny we make is re-invested into funding further development of free, open source technology for modern publishing.</p><p>The version of Ghost you are looking at right now would not have been made possible without generous contributions from the open source <a href=\"https://github.com/TryGhost\">community</a>.</p><h2 id=\"next-up-the-editor\">Next up, the editor</h2><p>The main thing you'll want to read about next is probably: <a href=\"/the-editor/\">the Ghost editor</a>. This is where the good stuff happens.</p><blockquote><em>By the way, once you're done reading, you can simply delete the default <strong>Ghost</strong> user from your team to remove all of these introductory posts! </em></blockquote>",
    "comment_id": "5b7ada404f87d200b5b1f9c8",
    "feature_image": "https://casper.ghost.org/v2.0.0/images/welcome-to-ghost.jpg",
    "featured": false,
    "page": false,
    "meta_title": null,
    "meta_description": null,
    "created_at": "2018-08-20T15:12:00.000+00:00",
    "updated_at": "2018-08-20T15:12:00.000+00:00",
    "published_at": "2018-08-20T15:12:06.000+00:00",
    "custom_excerpt": "Welcome, it's great to have you here.\nWe know that first impressions are important, so we've populated your new site with some initial getting started posts that will help you get familiar with everything in no time.",
    "excerpt": "Welcome, it's great to have you here.\nWe know that first impressions are important, so we've populated your new site with some initial getting started posts that will help you get familiar with everything in no time.",
    "codeinjection_head": null,
    "codeinjection_foot": null,
    "og_image": null,
    "og_title": null,
    "og_description": null,
    "twitter_image": null,
    "twitter_title": null,
    "twitter_description": null,
    "custom_template": null,
    "url": "https://demo.ghost.io/welcome/",
    "canonical_url": null,
    "authors": [{
        "id": "5951f5fca366002ebd5dbef7",
        "name": "Ghost",
        "slug": "ghost",
        "profile_image": "https://demo.ghost.io/content/images/2017/07/ghost-icon.png",
        "cover_image": null,
        "bio": "The professional publishing platform",
        "website": "https://ghost.org",
        "location": null,
        "facebook": "ghost",
        "twitter": "@tryghost",
        "meta_title": null,
        "meta_description": null,
        "url": "https://demo.ghost.io/author/ghost/"
    }],
    "tags": [{
        "id": "59799bbd6ebb2f00243a33db",
        "name": "Getting Started",
        "slug": "getting-started",
        "description": null,
        "feature_image": null,
        "visibility": "public",
        "meta_title": null,
        "meta_description": null,
        "url": "https://demo.ghost.io/tag/getting-started/"
    }],
    "primary_author": {
        "id": "5951f5fca366002ebd5dbef7",
        "name": "Ghost",
        "slug": "ghost",
        "profile_image": "https://demo.ghost.io/content/images/2017/07/ghost-icon.png",
        "cover_image": null,
        "bio": "The professional publishing platform",
        "website": "https://ghost.org",
        "location": null,
        "facebook": "ghost",
        "twitter": "@tryghost",
        "meta_title": null,
        "meta_description": null,
        "url": "https://demo.ghost.io/author/ghost/"
    },
    "primary_tag": {
        "id": "59799bbd6ebb2f00243a33db",
        "name": "Getting Started",
        "slug": "getting-started",
        "description": null,
        "feature_image": null,
        "visibility": "public",
        "meta_title": null,
        "meta_description": null,
        "url": "https://demo.ghost.io/tag/getting-started/"
    }
}]}
```

### Pages

Pages are structured identically to posts. The response object will look the same, only the resource key will be `pages`.

### Tags

By default, internal tags are always included, use `filter=visibility:"public"` to limit the response directly or use the [tags helper](/api/helpers/#tags) to handle filtering and outputting the response.

Tags that are not associated with a post are not returned. You can supply `include=count.posts` to retrieve the number of posts associated with a tag.

```json
{"tags": [{
    "id": "59799bbd6ebb2f00243a33db",
    "name": "Getting Started",
    "slug": "getting-started",
    "description": null,
    "feature_image": null,
    "visibility": "public",
    "meta_title": null,
    "meta_description": null,
    "url": "https://demo.ghost.io/tag/getting-started/"
}]}
```

### Authors

Authors that are not associated with a post are not returned. You can supply `include=count.posts` to retrieve the number of posts associated with an author.

```json
{"authors":[{
    "id": "5951f5fca366002ebd5dbef7",
    "name": "Ghost",
    "slug": "ghost",
    "profile_image": "https://demo.ghost.io/content/images/2017/07/ghost-icon.png",
    "cover_image": null,
    "bio": "The professional publishing platform",
    "website": "https://ghost.org",
    "location": null,
    "facebook": "ghost",
    "twitter": "@tryghost",
    "meta_title": null,
    "meta_description": null,
    "url": "https://demo.ghost.io/author/ghost/"
}]}
```

### Settings

The settings endpoint is a special case. You will receive a single object, rather than an array. This endpoint doesn't accept any query parameters.

```json
{"settings": {
  "title": "Ghost",
  "description": "The professional publishing platform",
  "logo": "https://static.ghost.org/v1.0.0/images/ghost-logo.svg",
  "icon": "https://static.ghost.org/favicon.ico",
  "cover_image": "https://static.ghost.org/v1.0.0/images/blog-cover.jpg",
  "facebook": "ghost",
  "twitter": "tryghost",
  "lang": "en",
  "timezone": "Etc/UTC",
  "codeinjection_head": null,
  "codeinjection_foot": null,
  "navigation": [
     { "label": "Home", "url": "/" },
     { "label": "Tag", "url": "/tag/getting-started/" },
     { "label": "Author", "url": "/author/ghost/" },
     { "label": "Help", "url": "https://help.ghost.org" }
  ]
}};

```


---


## Parameters

Query parameters provide fine-grained control over responses. All endpoints accept `include` and `fields`. Browse endpoints additionally accept `filter`, `limit`, `page` and `order`.

The values provided as query parameters MUST be url encoded when used directly. The [client libraries](/api/javascript/) will handle this for you.

### Include

Tells the API to return additional data related to the resource you have requested. The following includes are available:

- Posts & Pages: `authors`, `tags`
- Authors: `count.posts`
- Tags: `count.posts`

Includes can be combined with a comma, e.g. `&include=authors,tags`.

For posts and pages:

- `&include=authors` will add `"authors": [{...},]` and `"primary_author": {...}`
- `&include=tags` will add `"tags": [{...},]` and `"primary_tag": {...}`

For authors and tags:

- `&include=count.posts` will add `"count": {"posts": 7}` to the response.

### Fields

Limit the fields returned in the response object. Useful for optimising queries, but does not play well with include.

E.g. for posts `&fields=title,url` would return:

```json
{"posts": [{
    "id": "5b7ada404f87d200b5b1f9c8",
    "title": "Welcome to Ghost",
    "url": "https://demo.ghost.io/welcome/"
}]}
```

### Formats

(Posts and Pages only)

By default, only `html` is returned, however each post and page in Ghost has 3 available formats: `html`, `plaintext` and `mobiledoc`.

- `&formats=html,plaintext` will additionally return the plaintext format.


### Filter

(Browse requests only)

Apply fine-grained filters to target specific data.

- `&filter=featured:true` on posts, would return only those marked featured.
- `&filter=tag:getting-started` on posts, would return those with the tag slug that matches `getting-started`.

The possibilities are extensive! Query strings are explained in detail in the [filtering](#filtering) section.

### Limit

(Browse requests only)

By default, only 15 records are returned at once.

- `&limit=5` would return only 5 records.
- `&limit=all` will return all records - use carefully!

### Page

(Browse requests only)

By default, the first 15 records are returned.

- `&page=2` will return the second set of 15 records.

### Order

(Browse requests only)

Different resources have a different default sort order:

- Posts: `published_at DESC` (newest post first)
- Pages: `title ASC` (alphabetically by title)
- Tags: `name ASC` (alphabetically by name)
- Authors: `name ASC` (alphabetically by name)

The syntax for modifying this follows SQL order by syntax:

- `&order=published_at%20asc` would return posts with the newest post last


---


## Filtering

Ghost uses a query language called NQL to allow filtering API results. You can filter any field or included field using matches, greater/less than or negation, as well as combining with and/or. NQL doesn't yet support 'like' or partial matches.

Filter strings must be URL encoded. The [{{get}}](/api/handlebars-themes/helpers/get/) helper and [client libraries](/api/javascript) handle this for you.

At it's most simple, filtering works the same as in GMail, GitHub or Slack - you provide a field and a value, separated by a colon.

### Syntax Reference

#### Filter Expressions

A **filter expression** is a string which provides the **property**, **operator** and **value** in the form <strong>property:<em>operator</em>value</strong>:
- **property** - a path representing the field to filter on
- **:** - separator between **property** and an **operator**-**value** expression
- **operator** (optional) - how to compare values (`:` on its own is roughly `=`)
- **value** - the value to match against

#### Property

Matches: `[a-zA-Z_][a-zA-Z0-9_.]`
-   can contain only alpha-numeric characters and `_`
-   cannot contain whitespace
-   must start with a letter
-   supports `.` separated paths, E.g. `authors.slug` or `posts.count`
-   is always lowercase, but accepts and converts uppercase

#### Value

Can be one of the following
-   **null**
-   **true**
-   **false**
-   a **_number_** (integer)
-   a **literal**
    -   Any character string which follows these rules:
    -   Cannot start with `-` but may contain it
    -   Cannot contain any of these symbols: `'"+,()><=[]` unless they are escaped
    -   Cannot contain whitespace
-  a **string**
    -  `'` string here `'` Any character except a single or double quote surrounded by single quotes
    -   Single or Double quote *_MUST *_be escaped*
    -   Can contain whitespace
    -   A string can contain a date any format that can be understood by `new Date()`

#### Operators

* `-` - not
* `>` - greater than
* `>=` - greater than or equals
* `<` - less than
* `<=` - less than or equals
* `[` value, value, ... `]` - "in" group, can be negated with `-`

#### Combinations

* `+` - represents and
* `,` - represents or
* `(` filter expression `)` - overrides operator precedence

#### Strings vs Literals

Most of the time, there's no need to put quotes around strings when building filters in Ghost.
If you filter based on slugs, slugs are always compatible with literals.
However, in some cases you may need to use a string that contains one of the other characters used in the filter syntax, e.g. dates & times contain`:`. Use single-quotes for these.


---


## Pagination

All browse endpoints are paginated, returning 15 records by default. You can use the [page](#page) and [limit](#limit) parameters to move through the pages of records. The response object contains a `meta.pagination` key with information on the current location within the records:

```json
"meta":{
    "pagination":{
      "page":1,
      "limit":2,
      "pages":1,
      "total":1,
      "next":null,
      "prev":null
    }
  }
```


---


## Errors

The Content API will generate errors for the following cases:

- Status 400: Badly formed queries e.g. filter parameters that are not correctly encoded
- Status 401: Authentication failures e.g. unrecognised keys
- Status 404: Unknown resources e.g. data which is not public
- Status 500: Server errors e.g. where something has gone

Errors are also formatted in JSON, as an array of error objects.
The HTTP status code of the response along with the `errorType` property indicate the type of error.

The `message` field is designed to provide clarity on what exactly has gone wrong.

```json
{"errors": [{
    "message": "Unknown Content API Key",
    "errorType": "UnauthorizedError"
}]}
```

---


## Versioning

The v2 Content API is **stable** as of **Ghost 2.10.0**. See the [stability index](/faq/api-versioning/) for full details of the API versions.
You can disable the v0.1 Public API in the labs section of your admin panel.
