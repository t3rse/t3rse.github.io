---
layout: post
title: "Moving from BlogEngine.net to Octopress"
date: 2014-05-06 22:42:36 -0500
comments: true
categories: 
 - blogging octopress 
---

Although I switched to Octopress via inspiration from [Staxmanade's blog series](http://staxmanade.com/2014/04/migrating-blogspot-to-octopress-part-1-introduction/) on exporting content from Blogger, my current blogging has been with BlogEngine.net. Here is how I migrated the content over. 

****Export to BlogML****

The first task was to export my current blog entries. You can generate a BlogML file with all of your posts with the following steps:

1. Log into the administration portal then choose Settings. 
2. On the subnavigation select the "Import & Export" option.
3. Click the *Export* button.


****Importing to Octopress****

You need to generate files for each of your posts. I found [a lengthy Ruby script](https://github.com/philippkueng/philippkueng.github.com/blob/30ef1570f06d33938b18d5eee7767d6641b9a779/source/_import/blogml.rb) to do this but I just hacked up what I needed in a few lines of powershell: 

	$file_name = "{0}-{1}.html"
	$post_template = 
	@"
	---
	layout: post
	title: `"{0}`"
	comments: true
	---
	
	{1}
	"@
	$xml = [xml](get-content BlogML.xml)
	$xml.blog.posts.post | % { 
	  $name = $file_name -f [DateTime]::Parse($_."date-created").ToString("yyyy-MM-dd"), ($_."post-url" -replace ".aspx", "")
	  $name = $name -replace "/post/\d{4}/\d{2}/\d{2}/", ""
	  $value = $post_template -f $_.title.InnerText, $_.content.InnerText
	
	  New-Item -Path . -Name $name -Value $value -ItemType file
	}

I ran it in the same directory as my BlogML.xml file and it iterates over each post generating a file and using the post name and created date for a unique file name. 

****Fixing Images and Links****

You may need to clean up links to images and other posts. I've done it for images and something similar could be done for posts (I don't link to myself very often though I may update this script when there is time). First, I copied all my images out of the /App_Data/files directory and into /public/images in my octopress folder structure. Next, here is an powershell one liner (where my BlogEngine was "http://www.t3rse.com" you will need to make sure you replace with whatever domain name you were using): 

	ls *.html -rec | %{ $f=$_; (gc $f.PSPath) | %{ $_ -replace "http://www.t3rse.com/image.axd\?picture=", "/images/" } | sc $f.PSPath }

This script, run in the same directory as the generated posts visits each (the .html extension) and does simple replace. 

---

There may be some additional details migrating (for example, I ignored comments, I also didn't pull in the categories for each post) that I add tweaks for in the future. If you have a bright idea comment or modify my gists on github - I'd love to improve them for anyone else to use. 

- [Blog Engine Import Gist](https://gist.github.com/t3rse/11241697)
- [Blog Engine Image Fix](https://gist.github.com/t3rse/11401972)

