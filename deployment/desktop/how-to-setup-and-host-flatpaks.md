---
title: Set Up and Host Flatpaks
subsection: desktop
order: 5
---

# {{page.title}}

### Flatpak hosting and Metadata

The *flatpak-builder* binary generates AppStream metadata automatically when building applications if the *appstream-compose* tool is installed on the flatpak build machine. Flatpak[1] repositories are exported with a separate *appstream* branch which is automatically downloaded by GNOME Software and no additional work is required when building your application or updating the remote. Adding the remote is enough to add the application to the software center, on the assumption the AppData file is valid.

Extensive information on building flatpaks and on hosting and signing flatpak repostories can be found elsewhere[2].

In summary, to create an empty repository, you use:

        ostree init --mode=archive-z2 --repo=repo

To tell flatpak-builder to import the end result of a build into this repository, you pass *--repo=repo*:
        
        flatpak-builder --verbose --force-clean \
            --repo=repo --gpg-homedir=gpg --gpg-sign=$GPG_KEY \
            recipes flatpak/org.gnome.Recipes.json

To generate appstream branches and static deltas in this repository, you use:

        flatpak build-update-repo --generate-static-deltas --gpg-homedir=gpg --gpg-sign=$GPG_KEY repo

Note that both of these commands take a *--gpg-sign* argument. Flatpak uses GPG as a means to ensure that the repository
can be trusted, so you should sign your public repositories.

The best way to make your application and its Flatpak repository available to users is to publish a flatpakref file for it:
	
	[Flatpak Ref]
	Title=GNOME Recipes
	Name=org.gnome.Recipes
	Url=https://raw.githubusercontent.com/matthiasclasen/recipes-releases/master/repo/
	Branch=1.0
	IsRuntime=False
	GPGKey=...
	RuntimeRepo=https://sdk.gnome.org/gnome.flatpakrepo
	Comment=GNOME loves to cook

### Hosting a Flatpak repository on Github

Github isn't really set up for hosting Flatpak repositories, so we can't guarantee that this will keep working in the future. So once you created a local copy of your repository create a new project on Github, enable Github pages for the project and point it at the master branch.

Then use the follow commands to import your repository into Github.

	cd ~/src/myrepository
	git init
	git add -A
	git commit -a -m "first commit"
	git remote add origin git@github.com:yourgitaccount/myrepo.git
	git push -u origin master

Now you should be able to refer to your repo with a *raw.githubusercontent.com/* URL like the one shown in the flatpakrepo example above.

1. [Flatpak Homepage: 17th October 2016](http://flatpak.org/)

2. [Maintaining a Flatpak repository](https://blogs.gnome.org/alexl/2017/02/10/maintaining-a-flatpak-repository/)

