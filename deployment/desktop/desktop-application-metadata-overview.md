[Back to index](how-to-package-a-desktop-application.md) 

###System Architecture
####Linux File Hierarchy
Applications on Linux are expected to install binary files to /usr/bin, the install architecture independent data files to /usr/share/ and configuration files to /etc. Small temporary files can be stored in /tmp and much larger files in /var/tmp. Per-user configuration is either stored in the users home directory (in ~/.config) or stored in a binary settings store such as dconf.
See the File Hierarchy Standard[[1](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)] for more information.

###Desktop files
Desktop files have been around for a long while now and are used by almost all Linux desktops to provide the basic description of a desktop application that your desktop environment will display. Like a human readable name and an icon. 

So the creation of a desktop file on Linux allows a program to be visible to the graphical environment, e.g. KDE or GNOME Shell. If applications do not have a desktop file they must be manually launched using a terminal emulator. Desktop files must adhere to the Desktop File Specification[[2](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#introduction) ] and provide metadata in an ini-style format such as:

- Binary type, typically ‘Application’
- Program name (optionally localized)
- Icon to use in the desktop shell
- Program binary name to use for launching
- Any mime types that can be opened by the applications (optional)
- The standard categories the application should be included in (optional)
- Keywords (optional, and optionally localized)
- Short one-line summary (optional, and optionally localized)

The desktop file should be installed into /usr/share/applications for applications that are installed system wide. An example desktop file provided below:

	[Desktop Entry]
	Type=Application
	Name=OpenSCAD
	Icon=openscad
	Exec=openscad %f
	MimeType=application/x-openscad;
	Categories=Graphics;3DGraphics;Engineering;
	Keywords=3d;solid;geometry;csg;model;stl;
	Keywords[de]=3D;solid;Geometrie;csg;Modell;stl;



*Text 1:  example desktop file for the OpenScad project*

The desktop files are used when creating the software center metadata, and so you should verify that you ship a .desktop file for each built application, and that these keys exist: Name, Comment, Icon, Categories, Keywords and Exec and that desktop-file-validate correctly validates the file. There should also be only one desktop file for each application.

The application icon should be in the PNG format with a transparent background and installed in /usr/share/icons, /usr/share/icons/hicolor/*/apps/*, or /usr/share/${app_name}/icons/*. The icon should be at least 64×64 in size.

The file name of the desktop file is also very important, as this is the assigned ‘application ID’. New applications typically use a reverse-DNS style, e.g. org.gnome.Nautilus.desktop but older programs may just use a short name, e.g. gimp.desktop. It is important to note that the file extension is also included as part of the desktop ID.

You can verify your desktop file using the command 'desktop-file-validate'.  You just run it like this:

	desktop-file-validate myapp.desktop

This tools is available through the desktop-file-utils package, which you can install on Fedora Workstation using this command

	dnf install desktop-file-utils

###AppData Files
At least one valid AppData file with the suffix .appdata.xml file should be installed into /usr/share/appdata with a name that matches the name of the .desktop file, e.g. gimp.desktop & gimp.appdata.xml or org.gnome.Nautilus.desktop & org.gnome.Nautilus.appdata.xml.

In the AppData file you should include several 16:9 aspect screenshots along with a compelling translated description made up of multiple paragraphs. 

In order to make it easier for you to do screenshots in 16:9 format we created a small GNOME Shell extension called 'Screenshot Window Sizer'. You can install it on Fedora Workstation through the Software tool.
Once it is installed you can resize the window of your application to 16:9 format by focusing it and pressing 'ctrl+alt+s'. It should resize your application window to a perfect 16:9 aspect ratio and let you screenshot it.

Make sure you follow the style guide, which can be tested using the appstream-util command line tool. appstream-util is part of the 'appstream' package in Fedora Workstation.:

	appstream-util validate foo.appdata.xml
	
If you don't already have the appstream-util installed it can be installed using this command on Fedora Workstation:

	dnf install appstream

What is allowed in an AppData file is defined in the AppStream specification[[3](https://www.freedesktop.org/software/appstream/docs/)] but common items typical applications add is:

- License of the upstream project in SPDX identifier format [[4](https://spdx.org/licenses/)], or ‘Proprietary’
- A translated name and short description to show in the software center search results
- A translated long description, consisting of multiple paragraphs, itemized and ordered lists.
- A number of screenshots, with localized captions, typically in 16:9 aspect ratio
- An optional list of releases with the update details and release information.
- An optional list of kudos which tells the software center about the integration level of the application
- A set of URLs that allow the software center to provide links to help or bug information
- An optional gettext or QT translation domain which allows the AppStream generator to collect statistics on shipped application translations.

A typical (albeit somewhat truncated) AppData file is shown below:

	<?xml version="1.0" encoding="UTF-8"?>
	<component type="desktop">
	  <id>org.gnome.MultiWriter.desktop</id>
	  <metadata_license>CC0-1.0</metadata_license>
	  <project_license>GPL-2.0+</project_license>
	  <name>MultiWriter</name>
	  <name xml:lang="bs">Grupni pisač</name>
	  <summary>Write an ISO file to multiple USB devices at once</summary>
	  <summary xml:lang="bs">Zapišite ISO datoteku na nekoliko USB uređaja odjednom</summary>
	  <description>
	    <p>GNOME MultiWriter can be used to write an ISO file to multiple USB devices at once. Supported drive sizes are between 1GB and 32GB.</p>
	    <p xml:lang="bs">Gnomov Grupni pisač može da se koristi za zapisivanje ISO datoteke na nekoliko USB uređaja odjednom. Podržane veličine diskova su od 1GB do 32GB.</p>
	  </description>
	  <screenshots>
	    <screenshot type="default">
	      <image>https://git.gnome.org/browse/gnome-multi-writer/plain/data/appdata/gmw-startup.png</image>
	      <caption>Initial screen for the application</caption>
	      <caption xml:lang="bs">Početni ekran programa</caption>
	    </screenshot>
	  </screenshots>
	  <releases>
	    <release version="3.20.0" date="2016-03-21">
	      <description>
	        <p>This is the first stable release for GNOME 3.20</p>
	      </description>
	    </release>
	  </releases>
	  <kudos>
	    <kudo>AppMenu</kudo>
	    <kudo>HiDpiIcon</kudo>
	    <kudo>ModernToolkit</kudo>
	  </kudos>
	  <url type="homepage">https://wiki.gnome.org/Apps/MultiWriter</url>
	  <url type="bugtracker">https://bugzilla.gnome.org/browse.cgi?product=gnome-multi-writer</url>
	  <translation type="gettext">gnome-multi-writer</translation>
	</component>

####Some Appstream background

The AppStream specification is a mature and evolving standard that allows upstream applications to provide metadata such as localized descriptions, screenshots, extra keywords and content ratings for parental control. The basic concept is that the upstream project ships one extra AppData XML file which is used to build a global application catalog called an AppStream file. Over 1000 open source projects now include AppData files, and the software center shipped in Fedora, Ubuntu and OpenSuse is an easy-to-use application filled with useful application metadata. Applications without AppData files are no longer shown which provides quite some incentive to upstream projects wanting visibility in popular desktop environments.

The AppStream specification is an mature and evolving standard that allows upstream applications to provide metadata such as localized descriptions, screenshots, extra keywords and content ratings for parental control. The basic concept is that the upstream project ships one extra AppData XML file which is used to build a global application catalog called an AppStream file. Over 1000 open source projects now include AppData files, and the software center shipped in Fedora, Ubuntu and OpenSuse is now an easy to use application filled with useful application metadata. Applications without AppData files are no longer shown which provides quite some incentive to upstream projects wanting visibility in popular desktop environments.

AppStream[[3](https://www.freedesktop.org/software/appstream/docs/) ] was first introduced in 2008 and since then many people have contributed to the specification. It is being used primarily for application metadata but also now is used for drivers, firmware, input methods and fonts. There are multiple projects producing AppStream metadata and also a number of projects consuming the final XML metadata.

When applications are being built as packages by a distribution then the AppStream generation is done automatically, and you do not need to do anything other than installing a .desktop file and an appdata.xml file in the upstream tarball or zip file.

If the application is being built on your own machines or cloud instance then the distributor will need to generate the AppStream metadata manually. This would for example be the case when internal-only or closed source software is being either used or produced. This document assumes you are currently building RPM packages and exporting yum-style repository metadata for Fedora or RHEL although the concepts are the same for rpm-on-OpenSuse or deb-on-Ubuntu.

NOTE: If you are building packages, make sure that there are not two applications installed with one single package. If this is currently the case split up the package so that there are multiple subpackages or mark one of the .desktop files as NoDisplay=true. Make sure the application-subpackages depend on any -common subpackage and deal with upgrades (perhaps using a metapackage) if you’ve shipped the application before.

###Summary of Package building
The steps outlined above explain the extra metadata that you need for your application to show up in GNOME Software. This tutorial does not cover how to set up your build system to build these, but both for  Meson and autotools you should be able to find a plethora of examples online. 
And there are also major resources available to explain how to create a [Fedora RPM](https://fedoraproject.org/wiki/How_to_create_an_RPM_package) or [how to build a Flatpak](http://docs.flatpak.org/en/latest/). You probably also want to tie both the Desktop file and the AppData file into your i18n system so the metadata in them can be translated.

It is worth nothing here that while this document explains how you can do everything yourself, we do generally recommend relying on existing community infrastructure for hosting source code and packages if you can (for instance if your application is open source), as they will save you work and effort over time. For instance putting your source code on GNOME's git will make it easy for the GNOME translation teams to work on your application and thus increase the chance that it gets translated in many languages. And by building your package in Fedora you can get peer review of your package and free hosting of the resulting package. We are also working on a service that will automatically generate a Flatpack bundle of your application in Fedora, meaning you get a Flatpak version of your application with no extra effort.

So the steps outlined above explains the extra metadata you need to have your application show up in GNOME Software. This tutorial does not cover how to set up your build system to build these, but both for  Meson and autotools you should be able to find a long range of examples online. 
And there are also major resources available to explain how to create a [Fedora RPM](https://fedoraproject.org/wiki/How_to_create_an_RPM_package) or [how to build a Flatpak](http://docs.flatpak.org/en/latest/). You probably also want to tie both the Desktop file and the AppData file into your i18n system so the metadata in them can be translated.

It is worth nothing here that while this document explains how you can do everything yourself we do generally recommend relying on existing community infrastructure for hosting source code and packages if you can (for instance if your application is open source), as they will save you work and effort over time. For instance putting your sourcecode into the GNOME git will give you free access to the translator community in GNOME and thus increase the chance your application is internationalized significantly. And by building your package in Fedora you can get peer review of your package and free hosting of the resulting package. We are also working on a service that will automatically generate a Flatpack bundle of your application in Fedora, meaning you get a Flatpak version of your application for free as a result.

###Citations

1. [Filesystem Hierarchy Standard: 17th October 2016](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) 
2. [Desktop Specification: 17th October 2016](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#introduction) 
3. [AppStream Specification 0.10: 17th October 2016](https://www.freedesktop.org/software/appstream/docs/) 
4. [SPDX license list](https://spdx.org/licenses/)

[Back to index](how-to-package-a-desktop-application.md) 