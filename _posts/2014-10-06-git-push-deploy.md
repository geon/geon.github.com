---
layout: post
category : Programming
tags : [git, deployment, hooks, upstart]
title: Git Push Deploy
---


<style type="text/css">
pre.console {
font-family: menlo, consolas, inconsolata, sans-serif;
background-color: #222;
color: #eee;
line-height: 1.2em;
}
</style>

Disclaimer
----------
I am a Git/shell noob. Everything I write in this article is likely wrong, stupid and/or harmful. That being said…

"git push deploy"?
------------------
Updating files on a server through FTP sucks. You need to get the exactly right files moved, unless you just replace the entire project, clients are buggy and cumbersome, and you can't automate it very much. And God forbid you have more than one person accessing the server.

Just ugh.

You might already use git, and it's pretty easy to set up to work like a deployment tool. You basically add your production/testing servers as git remotes with suitable names and push to them. Post receive hooks on the servers take care of the details, like restarting.

This has been written about by a number of people already, but I'd like to share my experiences with it so far, and some of the cool solutions I used in a project I'm working on.

If you want to try it, read the excellent article [Git is not a deployment tool](http://gitolite.com/deploy.html). It goes through the details of how to use git for deployment. It is kind of ugly too, so you know it's legit.

The code for this article is [available at Github](https://github.com/geon/deploy-demo). It contains a tiny Node.js app and some setup files, just enough to serve as an illustration. Clone it to your local machine.

<small>Folder structure:</small>

	.gitignore
	app.js
	config.js
	package.json
	setup
		deploy-demo.conf
		schema.sql
		git-hooks
			post-receive
			pre-receive
	unversioned-files
		Put user files and caches in here.txt

When you use Git to deploy, any checked in file will be overwritten at each deploy. Therefore, any new files, like caches, uploaded user files, etc. must never be checked in. I like to make that easier by storing them all in a `unversioned-files` folder in the project root. It is checked in, but added to the `.gitignore` file, and any other folder that need to have unversioned files (like the `public` folder in Express) are symbolic links into it.

I won't go through the entire process of installing the server. For this demo, I used a pretty standard install of Ubuntu 14.10 with Node.js, running in VirtualBox. (Note that you still need [Chris Lea's 3:rd party repository](http://www.ubuntuupdates.org/ppa/chris_lea_nodejs) to get Node.js and npm properly working.) The rest of the article assumes you have it set up with a user named `demouser`.

Port 80
-------

You should probably serve HTTP on port 80, or you'll have trouble with firewalls and confused users. But you can't normally use ports below 1024 without being root, which would be dangerous.

If you have a large site, you probably run Nginx as a reverse proxy in front of Node.js. Then you could use any port for Node and configure Nginx accordingly.

For those of us who haven't bothered with that, there is a command to grant low port rights to a specific executable:

	sudo apt-get install libcap2-bin
	sudo setcap cap_net_bind_service=+ep /usr/bin/nodejs

There. Now anyone can start Node on port 80.

Running The App As A Service
----------------------------

The app must be started at boot, and have some kind of mechanism for restarting itself if (when) it crashes.

At first, the project I work on used the shell initialization script of a specific user that was made to auto-login on boot, and the Node.js based Forever. It was a huge cludge, made overcomplicated by some other features the app had to support.

Looking around, I found that running it as an Upstart service seemed to be the best solution. There are a number of advantages to this:

* The app can be started on boot.
* It can be restarted if it crashes.
* It runs in the background by default.
* Starting/stopping is standardized and centralized.

I'll expand on that last one.

It is very important to always start the app...

* ...from the right path - or you won't find files where you expect them.
* ...as the right user - or you won't have the right to read your database and files.
* ...with logging to the right file - or you won't know what went wrong.

With Upstart, you can easily get a small shell script to start the app consistently, and stopping/starting is available to any user with sudo privileges.

I obviously don't want the script to run as root, or have passwords in the scripts, but it turns out there is a way to make sudo run without a password for a specific command.

Entering the command below will open a text editor for you, where you can configure sudo rights.

	sudo visudo

Add this line:

	demouser ALL = (root) NOPASSWD: /sbin/start deploy-demo, /sbin/stop deploy-demo

Now, the exact commands

	sudo stop deploy-demo
	
and

	sudo start deploy-demo

can be run by the user `demouser` without entering the password.

Setting Up The Repo
-------------------

Setting up the repo on the production server is pretty simple. I use separate folders for the script-files and the repo itself, placed right in the user's home folder.

	mkdir ~/deploy-demo
	mkdir ~/deploy-demo.git

The suffix is nothing magical, just a convention. Initialize the folder with the `.git` suffix as "bare". 

	cd ~/deploy-demo.git
	git init --bare

Making it a "bare" repo [is a good idea](http://www.gitguys.com/topics/shared-repositories-should-be-bare-repositories/) since it will be pushed to, and no work should be done in the repo itself.

The repo is now ready to be pushed to. Back on your local machine, add the production server to your local repo:

	git remote add deploy demouser@[PRODUCTION_SERVER_IP]:~/deploy-demo.git/

There is nothing magical with the name `deploy`. In fact, add as many servers with diffrent names as you like. A `test`-server identical to your production environment would be a great idea.

Now, push with
	
	git push deploy

Nothing magical will happen yet. That's what the Git hooks are for.

Setting Up The Hooks
--------------------

Since you have already pushed the demo code to the server, you can just copy the pre-made hooks into the correct location.

On the production server (obviously), check out the files:

	cd ~/deploy-demo.git
	git --work-tree=../deploy-demo checkout -f master
	
Copy them:	

	cp ~/deploy-demo/	git-hooks/* ~/deploy-demo.git/hooks

The magic is all in the hooks. In my pre-receive hook, I check for changes to the config file and the database schema, since these will require manual meddling on the server.

Anything outputted to stdout/stderr will be sent by git to your console as well, so you'll se exactly what's going on remotely. Awesome.

Let's go through the hooks.

<small>Filename: `	git-hooks/pre-receive`</small>

```sh
#!/bin/bash

# Any error should exit.
set -e

# Headline. With Color!
echo -e "\n\033[1;32m-=#=- Deploying! -=#=-\033[0m\n"

# Show the db schema changes. On initial commit, oldrev is all 0s.
read oldrev newrev refname
if [ "0" == $(expr "$oldrev" : "0*$") ]; then

	setupChanges=$(git diff --color $oldrev $newrev -- setup)
	if [[ -n $setupChanges ]]; then
		echo -e "\n\033[1;31m - Changes to setup folder - Schema changes require manual ALTER TABLE! - \033[0m\n"
		echo -e "$setupChanges"
	fi

	configChanges=$(git diff --color $oldrev $newrev -- config.js)
	if [[ -n $schemaChanges ]]; then
		echo -e "\n\033[1;31m - Changes to config.js - The file has not been updated. Update manually! - \033[0m\n"
		echo -e "$schemaChanges" 
	fi
	
fi
```

The first `if`-block ensures the diffs are not run on the first push, since there is nothing to diff against yet.

The variables `setupChanges` and `configChanges` contains the output of diffing some files that requires the server to be manually updated. If there were any differences, they will be printed with a large, red headline.

The *post*-receive hook does the actual update.

<small>Filename: `	git-hooks/post-receive`</small>

```sh
#!/bin/bash

# Stop the service
sudo stop deploy-demo

# Stash the local config, so it won't be overwritten.
mv ../deploy-demo/config.js ../deploy-demo/config.js.local

# Update the files.
git --work-tree=../deploy-demo checkout -f master

# Un-stash the local config, in case it was overwritten.
mv ../deploy-demo/config.js.local ../deploy-demo/config.js

# Install any new dependencies.
cd ../deploy-demo
npm install

# Start the service.
sudo start deploy-demo
```

Unsurprisingly, `sudo stop deploy-demo`
 stops the service.

The config file needs to be stashed away before checking out the files, or it would be overwritten with the version in your git repo. 

After the checkout, I run `npm install`. All dependency errors will be clearly visible, so you notice if anything goes wrong.

At last, the service can be restarted.

Configuring Your App
--------------------

In this demo, it doesn't matter, but in a real life, this is when you would edit `config.js` to fit the server. The deploy hooks will protect this file from being overwritten by changes in the repo.

Suitable things to put here would be anything that changes between your development machine, the testing server ('cause you have one, right?) and the production server. Things like database connection strings, the URL to the website or whether errors should be displayed in the browser.

Changes you do locally to this file should not normally be committed, since they are unique for each machine. You can make git ignore changes to it with the command `git update-index --skip-worktree config.js`.

Setting Up Upstart
------------------

The Upstart file doesn't have to be complex. Actually, the one I use is very simple:

<small>Filename: `	setup/deploy-demo.conf`</small>

```sh
# Respawn up to 10 times in 1 minute.
respawn
respawn limit 10 1

# Start on all runlevels, except recovery (1). Stop on halt and reboot.
start on runlevel [2345]
stop on runlevel [06]

script
	# Run from demouser's ~/deploy-demo.
	cd /home/demouser/deploy-demo

	# Run as demouser. "exec" makes node replace the shell process. No point in leaving it running.
	# Append stdout and stderr to separate log files.
	exec sudo -u demouser node app.js 1>> ../debug-deploy-demo.log 2>> ../error-deploy-demo.log
end script
```

To make your app work with Upstart, the Upstart file needs to be copied to `/etc/init/`.

	sudo cp ~/deploy-demo/deploy-demo.conf /etc/init/

The name of the service is the name of the file without the suffix. Now you can start the service:

	sudo start deploy-demo

Then to see logs (with color!):

	tail -F debug-deploy-demo.log
	tail -F error-deploy-demo.log

Trying It Out
-------------

And that's pretty much it.

Let's try it out. First start up the service...

	sudo start deploy-demo

...and verify that it's running on http://[PRODUCTION\_SERVER\_IP]. You should see a happy cat smiley in your browser.

Now change `app.js` to serve something else:

```js
	res.end('Something else.');
```
	
Commit and push the change:

	git commit -am "Changed the cat smiley."
	git push deploy
	
"Screenshot":

<pre class="console">
<span style='color:#FD0'><b>geon@local-machine</b></span> ~/deploy-demo $ git commit -am "Changed the cat smiley."
[master 0c8c840] Changed the cat smiley.
 1 file changed, 1 insertion(+), 1 deletion(-)
<span style='color:#FD0'><b>geon@local-machine</b></span> ~/deploy-demo $ git push deploy
demouser@192.168.0.5's password: 
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects:  33% (1/3)   
Compressing objects:  66% (2/3)   
Compressing objects: 100% (3/3)   
Compressing objects: 100% (3/3), done.
Writing objects:  33% (1/3)   
Writing objects:  66% (2/3)   
Writing objects: 100% (3/3)   
Writing objects: 100% (3/3), 339 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: 
remote: <b><span style='color:#2F0'>-=#=- Deploying! -=#=-</span></b>
remote: 
remote: deploy-demo stop/waiting
remote: Already on 'master'
remote: deploy-demo start/running, process 3990
To demouser@192.168.0.5:~/deploy-demo.git/
   3f4e374..0c8c840  master -> master
</pre>

Refresh the browser, and you should see your change. That means the app was updated and the service restarted properly.

How about changing the table definition in `	schema.sql`? Add a column for author id, and remove the keywords.

```sql
CREATE TABLE posts (
	"id" SERIAL PRIMARY KEY,
	"authorId" INT NOT NULL,
	"slug" TEXT NOT NULL,
	"title" TEXT NOT NULL,
	"content" TEXT NOT NULL,
	"publishedAt" TIMESTAMPTZ DEFAULT NULL
);
```

That change needs to be done manually on the production server with an `ALTER TABLE` command, or the app will stop working. The pre-receive hook will warn you about that. Just try:

	git commit -am "Changed the db table definition."
	git push deploy

"Screenshot":

<pre class="console">
<span style='color:#FD0'><b>geon@local-machine</b></span> ~/deploy-demo $ git commit -am "Changed the db table definition."
[master a82a353] Changed the db table definition.
 1 file changed, 1 insertion(+), 1 deletion(-)
<span style='color:#FD0'><b>geon@local-machine</b></span> ~/deploy-demo $ git push deploy
demouser@192.168.0.5's password: 
Counting objects: 7, done.
Delta compression using up to 8 threads.
Compressing objects:  25% (1/4)   
Compressing objects:  50% (2/4)   
Compressing objects:  75% (3/4)   
Compressing objects: 100% (4/4)   
Compressing objects: 100% (4/4), done.
Writing objects:  25% (1/4)   
Writing objects:  50% (2/4)   
Writing objects:  75% (3/4)   
Writing objects: 100% (4/4)   
Writing objects: 100% (4/4), 377 bytes | 0 bytes/s, done.
Total 4 (delta 3), reused 0 (delta 0)
remote: 
remote: <b><span style='color:#2F0'>-=#=- Deploying! -=#=-</span></b>
remote: 
remote: 
remote: <b><span style='color:#F03'> - Changes to setup folder - Schema changes require manual ALTER TABLE! - </span></b>
remote: 
remote: <b>diff --git a/	schema.sql b/	schema.sql</b>
remote: <b>index 873c3ef..e2ea75e 100644</b>
remote: <b>--- a/	schema.sql</b>
remote: <b>+++ b/	schema.sql</b>
remote: <span style='color:#0FF'>@@ -1,9 +1,9 @@</span>
remote:  
remote:  CREATE TABLE posts (
remote:  	"id" SERIAL PRIMARY KEY,
remote: <span style='color:#2F0'>+</span>	<span style='color:#2F0'>"authorId" INT NOT NULL,</span>
remote:  	"slug" TEXT NOT NULL,
remote:  	"title" TEXT NOT NULL,
remote:  	"content" TEXT NOT NULL,
remote: <span style='color:#F03'>-	"keywords" TEXT NOT NULL,</span>
remote:  	"publishedAt" TIMESTAMPTZ DEFAULT NULL
remote:  );
remote: deploy-demo stop/waiting
remote: Already on 'master'
remote: deploy-demo start/running, process 4069
To demouser@192.168.0.5:~/deploy-demo.git/
   0c8c840..a82a353  master -> master
</pre>

Aww Yiss
--------

I tend to get too exited about stuff like this (I just wrote 2200 words about it), but it really is that cool. Not only does it save a lot of time when deploying, it also give me a whole new confidence about deploying, so I do it incrementally, all the time. When anything breaks, it is likely something small I can fix quickly, rather that a big mess.
