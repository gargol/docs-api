---
title: "reading_time"
date: "2018-10-01"
meta_title: "Handlebars Theme Helpers: reading_time"
meta_description: "Render the estimated reading time of a post in your Ghost publication with this handlebars helper ⚡️ Read more about Ghost themes!"
keywords:
    - api
    - handlebars
    - themes
    - helpers
sidebar: "handlebars"
---

Usage: `{{reading_time}}`

### Description

`{{reading_time}}` renders the estimated reading time for a post.

The helper counts the words in the post and calculates an average reading time of 275 words per minute. For the first image present, 12s is added, for the second 11s is added, for the third 10, and so on. From the tenth image onwards every image adds 3s.

By *default* the helper will render a text like this:
- `x min read` for estimated reading time longer than one minute
- `1 min read` for estimated reading time shorter than or equal to one minute

You can override the default text.

### Example Code

```handlebars
{{#post}}
    {{reading_time}}
{{/post}}
```

