---
layout: post
title: "create new jekyll posts rake task "
date: 2013-09-30 16:03
comments: true
categories: ruby
---
Jekyll, a static site generator, is blog-aware. That means it tracks posts and publication dates. Working with such files can be tricky, but here is one way of using Rake to ease the pain.

####What we need
Jekyll requires blog posts to be saved in files with a specific filename. It should include the title of the post and the publication date. Example: 2009-06-15-hello-world.markdown. The extension tells Jekyll what text filter to use (Markdown in this case).

####What we want
Manually typing filenames in the form yyyy-mm-dd-title.filter is a pain. I want to be able to create a post, given a title, and let it figure out the date itself.

####What we get
I use the following Rake task to create post files:
{% highlight ruby %}
desc 'create a new draft post'
task :post, :title do |t, args|
  title = args.title
  slug = "#{Date.today}-#{title.downcase.gsub(/[^\w]+/, '-')}"

  file = File.join(
    File.dirname(__FILE__),
    '_posts',
    slug + '.markdown'
  )

  if File.exist?(file)
    abort("rake aborted!\n #{file} already exists.")
  end

  File.open(file, "w") do |f|
    f << <<-EOS
---
layout: post
title: "#{title}"
date: #{Time.now.strftime('%Y-%m-%d %H:%M')}
comments: true
categories:
---
EOS
  end
  puts "Creating new page: #{file}"
end
{% endhighlight %}
Now I can type rake post TITLE='hello, world' to create a post. Here’s what it does:

At lines 3 and 2 it finds the TITLE argument and converts it to a suitable filename with the current date prepended.
At line 6 the filename is expanded to a full path to the file to create.
The following block (8–18) writes a post template to that file. This is some YAML front matter, with the published flag down by default.
Finally, at line 20, it launches the file in my default text editor, which in my case in TextMate.

####Draft posts
With the published flag set to false I can keep my drafts in my Git repository without actually publishing them. I can find all draft posts using rake drafts:
{% highlight ruby %}
desc 'List all draft posts'
task :drafts do
  puts `find ./_posts -type f -exec grep -H 'published: false' {} \\;`
end
{% endhighlight %}

####Renaming files
One slight problem occurs when I want to rename the file to publish it on a different date. I don’t have a task yet that can easily rename a file. It would also be useful to be able to override the default extension.

There’s still some work to do, but I guess I’ve got 80% of the scenarios covered.