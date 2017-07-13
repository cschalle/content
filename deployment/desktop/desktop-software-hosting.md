---
title: How to Host Desktop Software
subsection: desktop
order: 3
---

# {{page.title}}

## Setting up hosting infrastructure for your package

We will here explain how you set up a Yum repository for RPM packages that provides the needed metadata. If you are making a Flatpak we recommend skipping ahead to the [Flatpak section](how-to-setup-and-host-flatpaks.html).

### Yum hosting and metadata

When GNOME Software checks for updates it downloads various metadata files from the server describing the packages available in the repository. GNOME Software can also download AppStream metadata at the same time, allowing add-on repositories to include applications that are visible in the software center.

### Yum hosting and Metadata:

When GNOME Software checks for updates it downloads various metadata files from the server describing the packages available in the repository. GNOME Software can also download AppStream metadata at the same time, allowing add-on repositories to include applications that are visible in the the software center.

In most cases distributors are already building binary RPMS and then building metadata as an additional step by running something like this to generate the repomd files on a directory of packages. The tool for creating the repository metadata is called *createrepo_c* and is part of the package *createrepo_c* in Fedora. You can install it by running the command:

	dnf install createrepo_c

Once the tool is installed you can run these commands to generate your metadata: 

	$ createrepo_c --no-database --simple-md-filenames SRPMS/
	$ createrepo_c --no-database --simple-md-filenames x86_64/

This creates the primary and filelist metadata required for updating on the command line. Next to build the metadata required for the software center we we need to actually generate the AppStream XML. The tool you need for this is called *appstream-builder*. This works by decompressing *.rpm* files and merging together the *.desktop* file, the *.appdata.xml* file and preprocessing icons and screenshots. Remember, only applications installing AppData files will be included in the metadata.

You can install appstream-builder in Fedora Workstation by using this command:

	dnf install libappstream-glib-builder

Once it is installed you can run it by using the following syntax:

	$ appstream-builder				\
		--origin=yourcompanyname		\
		--basename=appstream			\
		--cache-dir=/tmp/asb-cache		\
		--enable-hidpi				\
		--max-threads=1				\
		--min-icon-size=32			\
		--output-dir=/tmp/asb-md		\
		--packages-dir=x86_64/			\
		--temp-dir=/tmp/asb-icons

This takes a few minutes and generates some files to the output directory.  Your output should look something like this:

	Scanning packages...
	Processing packages...
	Merging applications...
	Writing /tmp/asb-md/appstream.xml.gz...
	Writing /tmp/asb-md/appstream-icons.tar.gz...
	Writing /tmp/asb-md/appstream-screenshots.tar...
	Done!


The actual build output will depend on your compose server configuration. At this point you can also verify the application is visible in the *yourcompanyname.xml.gz* file.

We then have to take the generated XML and the tarball of icons and add it to the *repomd.xml* master document so that GNOME Software automatically downloads the content for searching. This is as simple as doing:

	modifyrepo_c					\
		--no-compress				\
		--simple-md-filenames			\
		/tmp/asb-md/appstream.xml.gz		\
		x86_64/repodata/
	modifyrepo_c					\
		--no-compress				\
		--simple-md-filenames			\
		/tmp/asb-md/appstream-icons.tar.gz	\
		x86_64/repodata/
		
Deploying this metadata will allow GNOME Software to add the application metadata the next time the repository is refreshed, typically, once per day.
