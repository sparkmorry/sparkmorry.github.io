---
layout: post
title: "Octopress Journey"
author: Mengli Xia
date: 2014-02-12 16:42:24 +0800
comments: true
categories: [octopress, essay]
---

&nbsp;&nbsp;&nbsp;&nbsp;After one-day searching, I finally set up my own blog by octopress. It looks so amazing and I just fall in love with the green markdown at the first glance. I will just imigrate the previous blog <http://www.cnblogs.com/sparkmorry/> @cnblog here.

&nbsp;&nbsp;&nbsp;&nbsp;This is just a testing post where I learn how to write an article with markdown and post it under a certain page. According to the documentation of Octopress, to write a new post and create a new page:

    rake new_page[super-awesome]
	# creates /source/super-awesome/index.markdown
 
	rake new_page[super-awesome/page.html]
	# creates /source/super-awesome/page.html
	
to publish:

	rake generate   # Generates posts and pages into the public directory
	rake watch      # Watches source/ and sass/ for changes and regenerates
	rake preview    # Watches, and mounts a webserver at http://localhost:4000
	rake deploy
	rake gen_deploy #combination

&nbsp;&nbsp;&nbsp;&nbsp;I have come across a problem of the blog configuration, the source code should be under the root directory for preview, or maybe by changing somewhere else. I am still in my process of learning this new awsome tool.

&nbsp;&nbsp;&nbsp;&nbsp;Here in my blog, the posts are under [Articles](http://sparkmorry.github.io/). The articles are mainly about the reflections on front-end development and my daily life. Also, you may see some recent work on drawings or photographies under [Gallery](http://sparkmorry.github.io/painting/) as I really like both of them. For more infomation, plz see [About page](http://sparkmorry.github.io/about/). Thanks to github and octopress that I can set up the blog for free. 

&nbsp;&nbsp;&nbsp;&nbsp; If you have any issues or questions on the code or posts, please contact me. Lastly, I'm so pleasant of having your visit on this blog.
