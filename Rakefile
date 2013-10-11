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