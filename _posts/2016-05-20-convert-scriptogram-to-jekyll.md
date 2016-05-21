---
layout: post
title:  "Moving to Jekyll"
date:   2016-05-20 01:36:41 -0400
categories: jekyll update
---
Wulp... it's that time again. I have this pattern:

1. Find a cool free blog host that supports [Markdown][] *(I started this back in 2011
or so when Wordpress had no clue what [Markdown][] was)*

2. Go all crazy on the publishing

3. Find out down the road the blog host has been bought or is shutting down

4. Stick my head in the sand for a while, hoping it's not true

5. Find a new option, rinse & repeat

I'm hoping to break that cycle. I'm going with running [Jekyll][] on my own droplet and
setting up post-commit hooks. 

So, in the mean time please forgive the clunkiness and non-formatted posts as I migrate.
And I hope to see you soon, and much more often than the last year or so.

## Migration Plan from Scriptogr.am

Scriptogram's headers did not have a block separator like Jekyll's, but, they always
appear at the top of the file. It seems I have some files where "Title" isn't on the
first line, so this is fun.

While I know I can do all of this in one block to make it more performant, or use something
other than [sed][], I chose to do one line at a time because I don't have that many posts and
I'm really only trying to be a little lazy. And I'm using [sed][] because I kinda love it.

So here we go. Here is the format *(order doesn't matter if I remember correctly, but the
headers did have to be at the top)*

	Title: Insert title here, no quotes required
	Date: YYYY-MM-DD
	Tags: comma, separated, tags, again, no, quotes, and, spaces, might, matter
	Published: False

The Published tag wasn't required, and actually seemed like sometimes it wasn't respected.
It was their way of doing a draft. Since I'm switching to `git`, I'm going to handle drafts
through branches, somewhat following gitflow.

+ On `master`, I have everything that I want published.
+ On `develop`, those are my drafts that I'm testing.
+ And for drafts under edit, I'm using `draft/draft-name` *(rather than `feat/foo`)* 

So, I'm just going to drop it. All that to say, my apologies if some random draft makes it
to production as I'm going through this migration. You'll know it either by my non-polished
rant or the subject just stopping part way through.

## Get to the migration!

Here we go.

### 1. Fix the title and add the `---`

Change this:

	Title: blah blah blah

to:
	
	---
	layout: post
	title: "blah blah blah"

**The code:**

	# test it (showing only the replacement)
	find -name '*md' -exec sed -n '1,5 s/Title: \(.*\)$/---\nlayout: post\ntitle: "\1"/p' {} \;

	# do it inline
	find -name '*md' -exec sed -i '1,5 s/Title: \(.*\)$/---\nlayout: post\ntitle: "\1"/' {} \;



**The breakdown:**

	# find all files that end in .md, (ignoring .md~)
	find -name '*md' ...
	# check the first 5 lines for Title
	... -exec sed -i '1,5 s/Title: ...
	# hold the actual title (and end the sed search with / )
	... \(.*\)$/ ...
	# and replace it with ---, lower case title, and the hold wrapped in quotes
	... /---\ntitle: "\1"/' ...
	# tell sed which file to work on (which is coming from find)
	... {} ...
	# and end the find command, which requires an escaped semicolon
	... \;


### 2. Fix the Date

Less explaination here, we're just changing Date to date, using a similar command to above

	# test
	find -name '*md' -exec sed -n '1,5 s/Date: \(2.*\)$/date: \1/p' {} \;

	# do:
	find -name '*md' -exec sed -i '1,5 s/Date: \(2.*\)$/date: \1/' {} \;

### 3. Change Tags to categories & close

We want `Tags: comma, separated, ...` to go to categories and close the headers:

	categories: ...
	---

This is going to be more like the first find / replace, but we also have to replace the
commas. Let's do that first:

	# test it
	find -name '*md' -exec sed -n '/^Tags:.*$/ s/,//pg' {} \;

	# replace it
	find -name '*md' -exec sed -i '/^Tags:.*$/ s/,//g' {} \;

Two new things above; now our range is restricted to lines lines that start with 'Tags'
rather than a numeric range, and we want to globally `g` replace the commas.

Now replace Tags with categories, going back to the line range:


	# test it (showing only the replacement)
	find -name '*md' -exec sed -n '1,5 s/Tags: \(.*\)$/categories: \1\n---/p' {} \;

	# do it inline
	find -name '*md' -exec sed -i '1,5 s/Tags: \(.*\)$/categories: \1\n---/' {} \;
	

### 4. Drop the Published stuff

	# test it (showing only the replacement)
	find -name '*md' -exec sed -n '1,5 s/Published.*$/you-wont-see-this-later/p' {} \;

	# do it inline
	find -name '*md' -exec sed -i '1,5 s/Published.*$//' {} \;
	




[jekyll]: http://jekyllrb.com/docs/home
[markdown]: http://daringfireball.net/projects/markdown/
