This is my "local port repository" for use with MacPorts, containing among others the first co-installable Qt4 and Qt5 ports and a collection of other ports not (yet) in or overriding those from the main repository.
That includes ports for a growing selection of KF5 ("KDE5") applications and the KF5 Frameworks (see below).

Instructions to use are given below. Make sure to read everything, especially the note about PortGroups. And the note about bug reports and feature requests.

## Installation

*Update 20211129 It has been brought to my attention that Apple have made use of the computers as traditional Unix workstations yet a bit more difficult. See issue #86 (https://github.com/RJVB/macstrop/issues/86) which has an adequate description of a pitfall I wasn't aware of.
I probably shouldn't be giving out advice about disabling SIP and whatever else Apple invented to "protect ourselves against using our computers" ... suffice it to say that it'd be the 1st thing I will disable on systems that have been burdened with it.
*

To "install", clone the repository to a location of choice. My original is in the root of the MacPorts prefix: /opt/local/site-ports; let's say you want to do the same:

%> sudo git clone --depth=1 --no-single-branch http://github.com/RJVB/macstrop.git /opt/local/site-ports
%> sudo chown -R macports /opt/local/site-ports

This makes sure that the resulting tree will be readable by the "macporst" user. (The --depth and --no-single-branch options are optional of course.)
To update the tree later on, use
%> sudo -u macports git -C /opt/local/site-ports pull
(or just use `port selfupdate`).

Then, edit ${prefix}/etc/macports/sources.conf (${prefix} is where MacPorts is installed; by default it's /opt/local). Towards the end of the file, add a line *above* the line that has [default] at the end:

file:///opt/local/site-ports

and save the file. If you do not want this repository to be updated automatically whenever you do port selfupdate or port sync, you'll want to add [nosync] at the end of the line you added.

Now, do

%> (cd /opt/local/site-ports ; sudo -u macports portindex)

and the new ports will be available.

Ports that are in my repository but are also in the default repository will override those in the default repository (and all other repositories listed after site-ports in sources.conf). If you use the aforementioned [nosync] option, you'll have to update the tree yourself. In that case you may have to do `portindex -f` at least periodically, to account for changes like removed variants or subports.

## PortGroups

An important note: this repository also contains PortGroup files. These files that contain definitions that are used by multiple ports that share certain properties (form a group, say all ports that build using CMake, or that use Qt or KDE libraries). PortGroups are like C header files, or Python/Perl/Tcl packages, included via a specific command in a port's "Portfile". Note that PortGroups themselves can include other PortGroups in exactly the same way! An example: the command `PortGroup kf5 1.1` will include the file kf5-1.1.tcl, which will then include other PortGroup files like qt5-1.0.tcl, qt5-kde-1.0.tcl and cmake-1.1.tcl. Each file will be included from either the same _resources directory, or else from the default _resources directory.
These PortGroups apply only to the ports in the local repository. This may lead to surprising behaviour and errors. For instance, if you install my port:qt5-kde or port:qt5-kde-devel, every port (including those in the official port tree) that requires Qt5 will need to use the corresponding Qt5 PortGroup (the qt5-*1.0.tcl files and probably qmake5-1.0.tcl), esp. if you plan to build them from source.
You will thus have to copy the corresponding files from the /opt/local/site-ports/_resources directory to the same directory in the default port tree (defined in sources.conf). You will have to do this again after each selfupdate.
Sadly there isn't much that can be done at the moment to make this more convenient. The best rule of thumb is to check for each port you install from my tree what PortGroups it uses (fgrep PortGroup `port file foo`), identify the corresponding files and files those include, check which ones are present in site-ports/_resources, and copy those files to the default _resources directory.
UPDATE: I am now providing a script which you can install in git's hooks directory, and which will copy all new files from the local _resource directory into the _resources directory (or directories of choice). It comes preconfigured with the location for the rsync-based standard ports. You will still need to do the initial install manually, but afterwards new versions should be installed (copied) automatically by the script.
In order to install this script do:

%> sudo -u macports ln -s ../../Hooks/post-checkout /opt/local/site-ports/.git/hooks/post-checkout
%> sudo -u macports ln -s ../../Hooks/post-checkout /opt/local/site-ports/.git/hooks/post-merge
%> sudo -u macports ln -s ../../Hooks/post-checkout /opt/local/site-ports/.git/hooks/post-rewrite

UPDATE (20180613): I now also provide a patch for your MacPorts "base" install which modifies the PortGroup lookup to use the same hierarchical search as also used for ports. As a result, PortGroup files from MacStrop will also override those from the other port trees lower in the tree list defined in sources.conf, unless you add `own_portgroups_first` to the tree options. In that case, the algorithm searches the port's own ports tree first.
Assuming the usual installation paths, the patch can be applied with

%> (cd /opt/local/libexec/macports/lib ; \
 sudo patch -p2 -i /opt/local/site-ports/sysutils/MacPorts/files/patch-hierarchical-portgroup-search.diff)

Now, for illustrative purposes: you could declare MacStrop at the bottom of sources.conf with

file:///opt/local/site-ports [own_portgroups_first]

and (say) the MacStrop KF5 ports will still use the MacStrop Qt5 PortGroup. Note that options must be separated with commas, and that the own_portgroups_first option is moot for the first tree in the list.

## Upgrades

A note about upgrades: I am not very religous at all in bumping the port revision variable each time I change something minor. That means that even after running portindex those ports won't be updated automatically (marked as outdated). It is thus useful to know that any port can be reinstalled ("upgraded") with the command `port -ns upgrade --force foo` (the -n is strongly advisable here; without it all ports on which `foo` depends would be reinstalled).

## KF5 ("KDE5")

A note about the KF5 ports:
These are designed to install much like the KDE4 ports from the official ports tree: using shared resources and dependencies.
For KDE4 this was the "linuxy but logical" way to do things, and as far as I'm concerned it still is for KF5 software in the context of a package management system like MacPorts. Deploying them as fully standalone "app bundles" is nice but inappropriate for MacPorts because it leads to duplicate installation of dependencies and will inevitably lead to interoperability issues because crucial resources are not installed to the proper central location(s).
This "linuxy" approach does require the use of a patched Qt5, which is provided by port:qt5-kde (or port:qt5-kde-devel) in this repository.
The recommended way of installing KF5 ports is to install port:qt5-kde first (and follow the above instructions about PortGroups!).
It should also be possible to specify the +qt5kde variant in the install command (e.g.

%> port install kf5-kate +qt5kde

I haven't tested this in a clean install; it is NOT necessary to specify the variant once port:qt5-kde is installed.

## Bug reports, upgrade/feature requests, etc.

Feel free to report issues or do upgrade or feature requests via the GitHub issue tracker. However, please consider the fact that many ports
in this tree are versions of ports from the main tree that have received some kind of personal convenience tweak(s). So, in terms of rules of thumb:

- upgrade requests go here
- if an issue you're seeing is clearly related to my tweaks, it will go here
- if a feature (or new port) request isn't compatible with or possible in the main tree (e.g. a new KF5 port), it'll go here
- anything that also applies to the mainstream version of a port would go onto the MacPorts issue tracker (trac.macports.org)
  (example: https://github.com/RJVB/macstrop/pull/64).

Please don't file duplicate issues, but if you do file a Trac ticket for something MacStrop-related please add me ("RJVB") in the 
`Cc:` field on the Trac new ticket form. That will allow me to remain informed about the issue, and see what I can do about it here.

Good MacPorts bug reporting practice has it that you attach the main log (`port logfile foo`). With modern computers having more and more
CPU cores it can be difficult to read that file to understand the error because output from several parallel build processes is intermingled.
So please have a look at the file after attaching it, and, for build failures, consider doing an additional build attempt with
`port -nok build foo [variants-and-options] build.jobs=1`; the`build.jobs` argument will force a serial build and the `main.log` file will
be rewritten with nicely serialised output of only the relevant part of the build procedure.

## Legal disclaimer (FWIW)
Many of the files in this repositories are more or less verbatim copies of files available elsewhere. I have neither removed nor added copyright or license information/claims. The commit history will make it clear which portfiles and patches have been authored by me. Those are explicitly put in the public domain, for use in any way seen fit, including incorporation (of code in the patch files) in commercially available software (like Qt) as long as a reference is made to my original authorship. Contact me in case of doubt or necessity.
