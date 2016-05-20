# Dependencies

Add these via apt-get... move them to Ansible management later

+ git
+ bundler
+ nginx
+ docker.io

## Non-apt

### Ruby :.(

Ruby... ugh. Using RVM:

	gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
	\curl -sSL https://get.rvm.io | bash -s stable

Then add the users to the rvm group:

	usermod -a -G rvm username

Then upgrade to beyond 2.0

	rvm install 2.2.0
	rvm use --default 2.2.0

*(see <https://rvm.io> and <http://cheat.errtheblog.com/s/rvm>)*

### Ruby Jekyll

gem install jekyll
gem install jekyll-paginate

### Fix the Jekyll _config.yml

See this post: <https://github.com/poole/lanyon/issues/124>

Specifically, do this:

	## find and comment the following line:
	# relative_permalinks: true

	## then make sure the rest is there
	gems: [jekyll-paginate]
	paginate: 5
	paginate_path: "page:num"


## Basic instructions

See here: <http://habd.as/simple-websites-jekyll-docker/>

And then the bottom of this post:
<http://gon.to/2015/03/03/f-percent-number-ck-github-pages-for-jekyll-why-i-decided-to-use-digital-ocean/>

And then finally, the official jekyll stuff:

<https://jekyllrb.com/docs/posts/>
<https://jekyllrb.com/docs/frontmatter/>

	# create a temp folder
	jekyll new blogs
	cd blogs
	cp -r * ../blog

	# fix the config file
	# see above

	cd ~/src/blog
	jekyll build -d ~/nginx-webroot		# my symlink to the html folder



## Final touches

I ran:

	jekyll build --incremental -d ~/nginx-webroot/

I then ran it with the --watch command, which makes it watch for changes. But, it also
didn't run in a daemon, and I'd have to run that on startup. So the post-commit hook would
be better.
