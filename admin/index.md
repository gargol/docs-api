---
title: "Admin API"
date: "2019-01-09"
meta_title: "Ghost Admin API Documentation"
meta_description: "Manage content via Ghost's Admin API, with secure role-based authentication. Read more on Ghost Docs 👉"
keywords:
    - "admin api"
    - "javascript"
    - "ghost api"
---

It's possible to create and manage your content using the Ghost Admin API. Our content management interface, Ghost Admin, uses the Admin API - which means that everything Ghost Admin can do is also possible with the API, and a whole lot more!

Secure authentication is available either as a User with role-based permissions, or as an integration with a single standard set of permissions designed to support common publishing workflows.

The API is RESTful with predictable resource URLs, standard HTTP verbs, response codes and authentication used throughout. Requests and responses are JSON-encoded with consistent patterns and inline relations and responses are customisable using powerful query parameters.

## API Clients

### JavaScript Client Library

We've developed an [API client for JavaScript](/api/javascript/#admin-api), that simplifies authenticating with the Admin API, and makes reading and writing data a breeze. The client is designed for use with integrations, supporting token authentication and the endpoints available to integrations.


## Structure

### Base URL

`https://{admin_domain}/ghost/api/{version}/admin/`

All Admin API requests start with this base URL.

#### Admin Domain

Your admin domain can be different to your main domain, and may include a subdirectory. Using the correct domain and protocol are critical to getting consistent behaviour, particularly when dealing with CORS in the browser. All Ghost(Pro) blogs have a `*.ghost.io domain` as their admin domain and require https.

#### Version

Version strings are required and usually start with `v`. The [api versioning](/faq/api-versioning/) guide explains the current available versions and stability index. The Admin API also has a stability index for specific [endpoints](/api/admin/#endpoints). 


### JSON Format

The API uses a consistent JSON structure for all requests and responses:

```json
{
    "resource_type": [{
        ...
    }],
    "meta": {}
}
```

The `resource_type` will always match the resource name in the URL. All resources are returned wrapped in an array, with the exception of specific singular resources returned from `/site/` or `/settings/`. 

The meta object contains [pagination](/api/content/#pagination) information for browse requests.

When composing JSON payloads to send to the API, you must always use this same format.

### Parameters

Query parameters provide fine-grained control over responses. All endpoints accept `include` and `fields`. Browse endpoints additionally accept `filter`, `limit`, `page` and `order`. Some endpoints have their own specific parameters.

The values provided as query parameters MUST be url encoded when used directly. The [client libraries](/api/javascript/) will handle this for you.

### Pagination

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

### Filtering

See the [Content API](/api/content/#pagination).

## Authentication

There are two methods for authenticating with the Admin API: token authentication and user authentication.

Most applications integrating with the Ghost Admin API should use token authentication. User authentication is intended for fully-fledged clients where different users login and manage various resources as themselves. Token authentication is intended for integrations that handle common workflows, such as publishing new content, or sharing content to other platforms.

The JavaScript Admin API Client supports token authentication.

### Token Authentication (Integrations)

Token authentication is a simple, secure authentication mechanism using JSON Web Tokens (JWTs). Each integration is issued with an admin API key, which is used to generate a JWT token and then provided to the API via the standard HTTP Authorization header.

The Admin API key must be kept private, therefore token authentication is not suitable for browsers or other insecure environments, unlike the Content API key.

#### Key

Admin API keys can be obtained by creating a new `Custom Integration` under the Integrations screen in Ghost Admin.

![Get a Ghost Admin API key](/images/apikey.png)

Admin API keys are made up of an id and secret, separated by a colon. These values are used separately to get a signed JWT token, which is used in the Authorization header of the request:


```bash
curl -H "Authorization: 'Ghost $token'" https://{admin_domain}/ghost/api/{version}/admin/{resource}/
```

The Admin API JavaScript client handles all the technical details of generating a JWT from an Admin API key, meaning you only have to provide your url, version and key to start making requests.

#### Token Generation

If you're using a language other than JavaScript, or are not using our client library, you'll need to generate the tokens yourself. It is not safe to swap keys for tokens in the browser, or in any other insecure environment.

There are a myriad of [libraries](https://jwt.io/#libraries) available for generating JWTs in different environments. 

JSON Web Tokens are made up of a header, a payload and a secret. The values needed for the header and payload are:

```json:title=Header:
{
    "alg": "HS256",
    "kid": {id}, // ID from your API key
    "typ": "JWT"
}
```
  
<br>

  
```json:title=Payload:
{
    // Timestamps are seconds sine the unix epoch, not milliseconds
    "exp": {timestamp}, // Max 5 minutes after 'now'
    "iat": {timestamp}, // 'now' (max 5 minutes after 'exp')
    "aud": "/{version}/admin/"
}
```

The libraries on https://jtw.io all work slighly differently, but all of them allow you to specify the above required values, including setting the signing algorith to the required HS-256. Where possible, the API will provide specific error messages when required values are missing or incorrect.

Regardless of language, you'll need to: 

1. Split the API key by the `:` into an `id` and a `secret`
2. Decode the hexadecimal secret into the original binary byte array
3. Pass these values to your JWT library of choice, ensuring that the header and payload are correct.

#### Token Generation Examples

These examples show how to generate a valid JWT in various languages & JWT libraries. The bash example shows step-by-step how to create a token without using a library.

<div class="code-tabs">
    <input name="tabs" type="radio" id="tab-bash" checked="checked"/>
    <label for="tab-bash">Bash</label>
    <div class="panel">


`embed:admin-jwt/auth.sh`

</div>
 <input name="tabs" type="radio" id="tab-js"/>
    <label for="tab-js">JavaScript</label>
    <div class="panel">


`embed:admin-jwt/auth.js`

</div>
    <input name="tabs" type="radio" id="tab-ruby"/>
    <label for="tab-ruby">Ruby</label>
    <div class="panel">
    

`embed:admin-jwt/auth.ruby`    
    
</div>
</div>

### User Authentication

User Authentication is an advanced, session-based authentication method that provides access to all API endpoints and actions according to the role of the user being authenticated.

Authenticating as a user requires an application to collect a user's email and password, and swap the credentials for a cookie. The cookie is then used to maintain a session.

#### Creating a Session

The session and authentication endpoints have custom payloads, different to the standard JSON resource format.

```javascript
POST /sessions/
```

To create a new session, send a username and password to the sessions endpoint, in this format:

```json:title=POST /admin/sessions/
{
    "username": "{email address}",
    "password": "{password}"
}
```

**Response:**

`201 Created`: A successful session creation will return HTTP  `201` response with an empty body and a  `set-cookie` header, in the following format:

```
set-cookie: ghost-admin-api-session={session token}; Path=/ghost; Expires=Mon, 26 Aug 2019 19:14:07 GMT; HttpOnly; SameSite=Lax
```

This cookie should then be provided with every API request:

- When making the request from a browser using the `fetch` api,  pass `credentials: 'include'` to ensure cookies are sent. 
- When using XHR you should set the `withCredentials` property of the xhr to `true`
- When using cURL you can use the `--cookie` and `--cookie-jar` options to store and send cookies from a text file.

Authenticating as a user with the Owner or Admin role will give access to the full set of API endpoints. Many endpoints can be discovered by inspecting the requests made by Ghost Admin, but all undocumented endpoints should be considered unstable. 


## Endpoints

These are the endpoints & methods currently available to integrations. More endpoints are available through user authentication. Each endpoint has a stability index, see [versioning](#versioning) for more information.

<table class="table">
<tbody>
<tr>
  <th>Resource</th>
  <th>Methods</th>
  <th>Stability</th>
</tr>
<tr>
<tr>
  <td><a href="/api/admin/#posts">/posts/</a></td>
  <td>Browse, Read, Edit, Add, Delete</td>
  <td>Stable</td>
</tr>
<tr>
  <td><a href="/api/admin/#pages">/pages/</a></td>
  <td>Browse, Read, Edit, Add, Delete</td>
  <td>Stable</td>
</tr>
<tr>
  <td>/tags/</td>
  <td>Browse, Read, Edit, Add, Delete</td>
  <td>Experimental</td>
</tr>
<tr>
  <td>/users/</td>
  <td>Browse, Read</td>
  <td>Experimental</td>
</tr>
<tr>
  <td><a href="/api/admin/#images">/images/</a></td>
  <td>Upload</td>
  <td>Stable</td>
</tr>
<tr>
  <td>/themes/</td>
  <td>Upload</td>
  <td>Experimental</td>
</tr>
<td><a href="/api/admin/#site">/site/</a></td>
  <td>Read</td>
  <td>Stable</td>
</tr>
<tr>
  <td>/webhooks/</td>
  <td>Add, Delete</td>
  <td>Experimental</td>
</tr>
<tr>
  <td>/subscribers/</td>
  <td>Browse, Read, Edit, Add, Delete</td>
  <td>Experimental</td>
</tr>
</tbody>
</table>


## Posts

Posts are the [primary resource](/concepts/posts/) in a Ghost site, providing means for publishing, managing and displaying content. 
At the heart of every post is a mobiledoc field, containing a standardised JSON-based representation of your content, which can be rendered in multiple formats.


```JavaScript
GET /admin/posts/
GET /admin/posts/{id}/
GET /admin/posts/slug/{slug}/
POST /admin/posts/
PUT /admin/posts/{id}/
DELETE /admin/posts/{id}/
```

### The Post Object

Whenever you fetch posts, or create or edit a post, the API will respond with an array of one or more post objects. These objects will include all related tags, authors, and author roles.

By default, the API expects and returns content in the mobiledoc format only. To include html in the response use the `formats` parameter:

```json:title=GET /admin/posts/?formats%3Dhtml,mobiledoc
{
    "posts": [{
        "id": "5b7ada404f87d200b5b1f9c8",
        "uuid": "22af052d-2bc1-4306-96d1-667584c797c7",
        "title": "Welcome to Ghost",
        "slug": "welcome",
        "mobiledoc": "{\"version\":\"0.3.1\",\"atoms\":[],\"cards\":[],\"markups\":[],\"sections\":[[1,\"p\",[[0,[],0,\"👋 Welcome, it's great to have you here.\"]]]]}",
        "html": "<p>👋 Welcome, it's great to have you here.</p>",
        "comment_id": "5b7ada404f87d200b5b1f9c8",
        "feature_image": "https://casper.ghost.org/v2.0.0/images/welcome-to-ghost.jpg",
        "featured": false,
        "status": "published",
        "meta_title": null,
        "meta_description": null,
        "created_at": "2018-08-20T15:12:00.000Z",
        "updated_at": "2018-08-20T15:12:00.000Z",
        "published_at": "2018-08-20T15:12:06.000Z",
        "custom_excerpt": "Welcome, it's great to have you here.",
        "codeinjection_head": null,
        "codeinjection_foot": null,
        "og_image": null,
        "og_title": null,
        "og_description": null,
        "twitter_image": null,
        "twitter_title": null,
        "twitter_description": null,
        "custom_template": null,
        "tags": [
            {
                "id": "59799bbd6ebb2f00243a33db",
                "name": "Getting Started",
                "slug": "getting-started",
                "description": null,
                "feature_image": null,
                "visibility": "public",
                "meta_title": null,
                "meta_description": null,
                "created_at": "2017-07-27T07:52:29.000Z",
                "updated_at": "2017-07-27T07:52:29.000Z",
                "url": "https://demo.ghost.io/tag/getting-started/"
            }
        ],
        "authors": [
            {
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
                "accessibility": null,
                "status": "active",
                "meta_title": null,
                "meta_description": null,
                "tour": null,
                "last_seen": null,
                "created_at": "2017-07-27T07:52:31.000Z",
                "updated_at": "2017-07-27T08:28:34.000Z",
                "roles": [
                    {
                        "id": "59799bbe6ebb2f00243a33e1",
                        "name": "Author",
                        "description": "Authors",
                        "created_at": "2017-07-27T07:52:30.000Z",
                        "updated_at": "2017-07-27T07:52:30.000Z"
                    }
                ],
                "url": "https://demo.ghost.io/author/ghost/"
            }
        ],
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
            "accessibility": null,
            "status": "active",
            "meta_title": null,
            "meta_description": null,
            "tour": null,
            "last_seen": null,
            "created_at": "2017-07-27T07:52:31.000Z",
            "updated_at": "2017-07-27T08:28:34.000Z",
            "roles": [
                {
                    "id": "59799bbe6ebb2f00243a33e1",
                    "name": "Author",
                    "description": "Authors",
                    "created_at": "2017-07-27T07:52:30.000Z",
                    "updated_at": "2017-07-27T07:52:30.000Z"
                }
            ],
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
            "created_at": "2017-07-27T07:52:29.000Z",
            "updated_at": "2017-07-27T07:52:29.000Z",
            "url": "https://demo.ghost.io/tag/getting-started/"
        },
        "url": "https://demo.ghost.io/welcome/",
        "canonical_url": null,
        "excerpt": "Welcome, it's great to have you here."
    }
]}
```

#### Parameters

When retrieving posts from the Admin API, it is possible to use the `include`, `formats`, `filter`, `limit`, `page` and `order` parameters as documented for the [Content API](/api/content/#parameters).
Some defaults are different between the two APIs, however the behaviour and availabililty of the parameters remains the same.

### Creating a Post

To create a new post, the only required field is `title`. All other fields can either be empty, or have a default. A post must always have at least one author. This will default to the staff user with the owner role. Below is a minimal example for creating a published post with content:


```json:title=POST /admin/posts/
{
    "post": [{
        "title": "My test post",
        "mobiledoc": "{\"version\":\"0.3.1\",\"atoms\":[],\"cards\":[],\"markups\":[],\"sections\":[[1,\"p\",[[0,[],0,\"My post content. Work in progress...\"]]]]}",
        "status": "published"
    }]
}
```

#### Source HTML

The post creation endpoint is also able to convert HTML into mobiledoc. The conversion generates the best available mobiledoc representation, meaning this operation is lossy and the HTML rendered by Ghost may be different from the source HTML.

For lossless HTML conversion, you can wrap your HTML in a single mobiledoc card, [as demonstrated here](/api/migration/#mobiledoc-html-card).

To use HTML as the source for your content instead of mobiledoc, use the `source` parameter:


```json:title=POST /admin/posts/?source%3Dhtml
{
    "post": [{
        "title": "My test post",
        "html": "<p>My post content. Work in progress...</p>",
        "status": "published"
    }]
}
```

#### Tags and Authors

You can link tags and authors to any post you create in the same request body, using either short or long form to identify linked resources.

Short form uses a single string to identify a tag or author resource. Tags are identified by name and authors are identified by email address:

```json:title=POST /admin/posts/
{
    "post": [{
        "title": "My test post",
        "tags": ["Getting Started", "Tag Example"],
        "authors" ["example@ghost.org", "test@ghost.org"],
        "mobiledoc": "{\"version\":\"0.3.1\",\"atoms\":[],\"cards\":[],\"markups\":[],\"sections\":[[1,\"p\",[[0,[],0,\"My post content. Work in progress...\"]]]]}",
        "status": "published"
    }]
}
```

Long form requires an object with at least one identifying key-value pair:

```json:title=POST /admin/posts/
{
    "post": [{
        "title": "My test post",
        "tags": [{"name": "my tag", "description": "a very useful tag"}, {"name": "#hidden}],
        "authors": [{"id": "5c739b7c8a59a6c8ddc164a1"}, {"id": "5c739b7c8a59a6c8ddc162c5"}, {"id": "5c739b7c8a59a6c8ddc167d9"}]        
    }]
}
```

Tags that cannot be matched are automatically created. If no author can be matched, Ghost will fallback to using the staff user with the owner role.

## Pages

Pages are [static resources](/concepts/pages/) that are not included in channels or collections on the Ghost front-end. They are identical to posts in terms of request and response structure when working with the APIs.

```JavaScript
GET /admin/pages/
GET /admin/pages/{id}/
GET /admin/pages/slug/{slug}/
POST /admin/pages/
PUT /admin/pages/{id}/
DELETE /admin/pages/{id}/
```

## Images

Sending images to Ghost via the API allows you to upload images one at a time, and store them with a [storage adapter](/concepts/storage-adapters/). The default adapter stores files locally in /content/images/ without making any modifications, except for sanitising the filename.


```JavaScript
POST /admin/images/upload/
```

### The Image Object

Images can be uploaded to, and fetched from storage. 
When an image is uploaded, the response is an image object that contains the new URL for the image - the location from which the image can be fetched.

`url`: _URI_  
The newly created URL for the image. 

`ref`: _String (optional)_  
The reference for the image, if one was provided with the upload.

```json:title=POST /admin/images/upload/
{ 
    images: [{ 
      url: "https://demo.ghost.io/content/images/2019/02/ghost-logo.png", 
      ref: "ghost-logo.png"  
    }] 
}
```

### Uploading an Image

To upload an image, send a multipart formdata request by providing the `'Content-Type': 'multipart/form-data;'` header, along with the following fields encoded as [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData):

`file`: _[Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) or [File](https://developer.mozilla.org/en-US/docs/Web/API/File)_  
The image data that you want to upload.

`purpose`: _String (default: `image`)_  
Intended use for the image, changes the validations performed. Can be one of `image` , `profile_image` or `icon`.  The supported formats for `image` and `profile_image` are JPEG, GIF, PNG and SVG. Supported formats for `icon` are ICO and PNG. `profile_image` must be square.

`ref`: _String (optional)_      
A reference or identifier for the image, e.g. the original filename and path. Will be returned as-is in the API response, making it useful for finding & replacing local image paths after uploads.

```bash
curl -X POST -F 'file=@/path/to/images/my-image.jpg' -F 'ref=path/to/images/my-image.jpg' -H "Authorization: 'Ghost $token'" https://{admin_domain}/ghost/api/{version}/admin/images/upload/
```


## Site

Site is a special unauthenticated, read-only endpoint for retrieving basic information about a site. 
This information is useful for integrations and clients that need to show some details of a site before providing authentication.

```JavaScript
GET /admin/site/
```
### The Site Object

The site endpoint returns a single object, rather than an array.

`title`: _String_  
The title of the site, same as the title returned from the `settings` endpoint.

`url`: _URI_  
The frontend URL for the site, which can be different to the admin / api URL. This comes from the configuration JSON file.

`version`: _Semver String (major.minor)_  
The current version of the Ghost site. Use this to check the minimum version is high enough for compatibility with integrations.


```GET json:title=/admin/site/
{
    "site": {
        "title": "Ghost",
        "url": "https://demo.ghost.io/",
        "version": "2.16"
    }
}
```


## Versioning

The v2 Admin API introduced several **stable** endpoints as of **Ghost 2.16.0**. 
See the [endpoints table](#endpoints) for details of which endpoints are considered stable.
See the [stability index](/faq/api-versioning/) for full details of the API versions.
