---
title: Testing Jekyll!
layout: post
date: '2018-08-09 01:18:06 +0530'
categories: jekyll update tests
---

**A**t a glance, Jekyll seems to be a great `method` for serving static content. And it probably is. I'll keeping probing this and tweaking that till I get a pretty good understanding of the markdown type site.

## Update

Since I already had a personal blog on wordpress.com, I wanted to migrate my content off wordpress to github. Manually doing such a task would've been a nightmare, but, Jekyll already has an importer plugin for wordpress.com. (And many other services/cms as well).

There were other alternatives that I could've used for the migration, but, the prior seemed easy.

I downloaded an `xml` file that contained entries of my posts and associated assets (images, urls etc) from the export option at `https://YOUR_BLOG.wordpress.com/wp-admin/export.php`.

<img src="/images/testing-jekyll/export.png" alt="export.php" width="600px" class="center"/>

Then used the following script to automate the task of exporting from the wordpress site.
```ruby
require "jekyll-import";
    JekyllImport::Importers::WordpressDotCom.run({
      "source" => "wordpress.xml",
      "no_fetch_images" => false,
      "assets_folder" => "assets"
    })
```

Simply calling the script from the command window is sufficent for the script to execute.
```bash
$ ruby import.rb
```

After the script completes, I had to do some housekeeping stuff to clear the markup artifacts that were showin in the posts. I also changed from the default Minima theme to the [Tale theme](http://github.com/chesterhow/tale) since it looked quite simplistic.

The fact that a simple commit on the repo would update the whole site is very handy. Writing a blog post and simply committing would update the blog post as well. This, all in all, is quite a nice way to manage 'pseudo-static' site.