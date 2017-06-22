---
title: Hosting a Yum Repository on Github
subsection: desktop
order: 4
---

# {{page.title}}

#### Hosting your Yum repository on Github

Github isn't really set up for hosting Yum repositories, but here is a method that currently works. So once you created a local copy of your repository create a new project on github. Then use the follow commands to import your repository into github.

	cd ~/src/myrepository
	git init
	git add -A
	git commit -a -m "first commit"
	git remote add origin git@github.com:yourgitaccount/myrepo.git
	git push -u origin master

Once everything is important go into the github web interface and drill down in the file tree until you find the file called 'repomd.xml' and click on it. You should now see a button the github interface called 'Raw'. Once you click that you get the raw version of the XML file and in the URL bar of your browser you should see a URL looking something like this:

	https://raw.githubusercontent.com/cschalle/hubyum/master/noarch/repodata/repomd.xml

Copy that URL as you will need the information from it to create your .repo file which is what distributions and users want in order to reach you new repository. To create your .repo file copy this example and edit it to match your data:

	[remarkable]
	name=Remarkable Markdown editor software and updates
	baseurl=https://raw.githubusercontent.com/cschalle/hubyum/master/noarch
	gpgcheck=0
	enabled=1
	enabled_metadata=1

So on top is your Repo shortname inside the brackets, then a name field with a more extensive name. For the baseurl paste the URL you copied earlier and remove the last bits until you are left with either the 'norach' directory or your platform directory for instance x86_64.

Once you have that file completed put it into /etc/yum.repos.d on your computer and load up GNOME Software. Click on the 'Updates' button in GNOME Software and then on the refresh button in the top left corner to ensure your database is up to date. If everything works as expected you should then be able to do a search in GNOME software and find your new application showing up.
![Example GNOME Software listing](/content/deployment/desktop/example-gnome-software-listing.png  "Example GNOME Software listing")

