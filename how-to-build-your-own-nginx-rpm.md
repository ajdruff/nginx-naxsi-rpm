

##How to Build Your Own Nginx RPM from Source



###Use Scenario:

So you need an Nginx rpm but there is no current package that installs all the options that you need. Your options are either to use make and build from source, or to re-create the nginx package from source. In this tutorial, we'll show you how to rebuild the rpm from source, to create a version of nginx that has the naxsi WAF server integrated into it.




###Overview/Outline



> **This overview is a quick rundown of the major steps involved. It is not intended as a thorough step by step guide. For more detailed steps, scroll down to the next section.**



* Preqs: Create User and Install Dev and Build Tools  

	Install Build Tools

		sudo yum  install --enablerepo=epel yum-utils rpmdevtools

	alternately install the full set of dev tools:  
	
		sudo yum groupinstall --enablerepo=epel  'Development Tools'

	* yum utils ( installs downloader which automatically installs srpms repos, allowing you to download source rpms)
	
	
* Create User

	A non-root user is needed to do the build. This is a precautionary measure that Fedora and others strongly recommend because of the danger of building packages with root/sudo privileges.

* Edit Your User's ~./rpmmacros file. 

	This is a configuration file for your user that provides the rpm dev tools information about you and your environment so it can build you package


	Example:
	
			%_topdir %(echo $HOME)/rpmbuild
			%_smp_mflags -j2
			%__arch_install_post /usr/lib/rpm/check-rpaths /usr/lib/rpm/check-buildroot
			%_query_all_fmt %%{name}-%%{version}-%%{release}.%%{arch}
	


* Configure your repo config files to point to the source repos (only necessary if you didn't install yum downloader)

* Create the RPM Build Directory Structure
	* rpmdev-setuptree
* Download RPM Source and Spec File 
	* This places a .spec file in the /spec directory
	* and a number of .bz2 (compressed source), .patch (patch file), and .sign files in the /source directory
* Build dependencies
	* sudo yum-builddep <package>.src.rpm 
	* ref:http://repoforge.org/package/rebuild.html
* Build and Prep (Integrate the Patches into the Source)
	* Next, we want to create source files that include all the latest patches. We do this by using the 'build and prep' options against the spec file
	* What we are left with are patched source files within the build/<package name> directory
* OR Convert SRPMS to original sources
	* use rpm2cpio (rpm to cp) which creates a spec and tar.gz file
	* untar the tar.gz file to get original cp sources
	* 
* Finally, rebuild:
	* rpmbuild --rebuild <package>.src.rpm 

Tips:  

* Never include a binary with your package - always the original sources ('pristine source')
* always install dependencies before you do the make 

Resources

[https://fedoraproject.org/wiki/How_to_create_an_RPM_package]()
[http://www.crashcourse.ca/content/working-source-rpms-under-centos]()
[http://wiki.centos.org/HowTos/SetupRpmBuildEnvironment]()
[http://hacktux.com/fedora/source/rpm]()


Repack an RPM without Sources
[http://stackoverflow.com/a/6461281/3306354]()

Use FPM
[https://github.com/jordansissel/fpm]()

[www.openlogic.com/wazi/bid/296143/Quickly-create-custom-software-packages-with-FPM]()


References:  

* Main reference used for this tutuorial: [http://hacktux.com/fedora/source/rpm]()

####Step By Step Example of Compiling an Nginx RPM from Source


#####Pre-reqs: Create User and Install Dev and Build Tools  


**Build Server**

Setting up a build server is outside the scope of this tutorial, but we strongly recommend it. A build server is identical to your production server with the exception that it has all your build tools and build users installed.

If you happen to be running on AWs, creating a build server is easy. Simply create an image of your current running instance,  then launch a new instance from that image. Just update a local hosts file to point to your build server, and configure the remote /etc/hosts file to also point to the server.

Here are the basic steps:
* create an ami of the running instance
* launch a new instance from the ami you created
* edit /etc/hosts and add an entry for the public IP and mydomain-build.com or whatever you want to call it.
* edit hosts file on your local workstation for the public IP and mydomain-build.com.
* thats it - now you don't have to worry about messing up your prod server with build tools.






**Install Build Tools**

	sudo yum  install --enablerepo=epel yum-utils rpmdevtools

This is what it installed:


	 rpmdevtools              noarch            7.5-2.el6                   base               109 k
	 yum-utils                noarch            1.1.30-17.el6_5             updates            102 k
	Installing for dependencies:
	 elfutils                 x86_64            0.152-1.el6                 base               201 k
	 elfutils-libs            x86_64            0.152-1.el6                 base               181 k
	 fakeroot                 x86_64            1.12.2-22.2.el6             base                73 k
	 fakeroot-libs            x86_64            1.12.2-22.2.el6             base                23 k
	 gdb                      x86_64            7.2-60.el6_4.1              base               2.3 M
	 patch                    x86_64            2.6-6.el6                   base                90 k
	 rpm-build                x86_64            4.8.0-37.el6                base               127 k
	 unzip                    x86_64            6.0-1.el6                   base               149 k


**Create User**

	sudo useradd builder
	sudo passwd builder 
	su builder
	cd ~


**Configure build ~/.rpmmacros preferences**

The dev tools look at a special file ~/.rpmmacros for any user specific 'macro' settings. A macro is simply a variable that the build tools reference as they build your package. Each user can specify their own macros in the .rpmmacros file.

	nano ~/.rpmmacros

add the following lines and save

	%_topdir %(echo $HOME)/rpmbuild
	%__arch_install_post /usr/lib/rpm/check-rpaths /usr/lib/rpm/check-buildroot
	%_query_all_fmt %%{name}-%%{version}-%%{release}.%%{arch}


**Setup your rpm build directory**

	rpmdev-setuptree

Your user's home directory should now contain the following directory and file:

		rpmbuild
		.rpmmacros

within the rpmbuild directory, you should have the following directories:
	
	rpmbuild/BUILD:
	
	rpmbuild/RPMS:
	
	rpmbuild/SOURCES:
	
	rpmbuild/SPECS:
	
	rpmbuild/SRPMS:


**Download nginx source**

	cd ~/rpmbuild/SRPMS/
	yumdownloader --source nginx


when we do this, only the 1.0.15 source package is available from the epel repo. 

If you cant find the srpms you need, review the 'where to get srpm packages' information at http://wiki.centos.org/HowTos/RebuildSRPM . There you'll see where else you can find rpms. For example, it says that for Centos 6, you should be able to use packages from fedora 14 and above
	

we do a bit of searching and find one we can use at nginx.org:

		wget http://nginx.org/packages/rhel/6/SRPMS/nginx-1.6.0-1.el6.ngx.src.rpm

Now build it with dependencies


		sudo yum-builddep --enablerepo=epel --nogpgcheck nginx-1.6.0-1.el6.ngx.src.rpm


Now expand the source rpm into our build directories

	rpm -ivh --nosignature  nginx-1.6.0-1.el6.ngx.src.rpm


Now lets take a look at whats contained in our build directories. To find out, do a find ~/rpmbuild

this is what we see


	
	/home/builder/rpmbuild
	/home/builder/rpmbuild/SPECS
	/home/builder/rpmbuild/SPECS/nginx.spec
	/home/builder/rpmbuild/RPMS
	/home/builder/rpmbuild/SOURCES
	/home/builder/rpmbuild/SOURCES/nginx.vh.example_ssl.conf
	/home/builder/rpmbuild/SOURCES/logrotate
	/home/builder/rpmbuild/SOURCES/nginx.vh.default.conf
	/home/builder/rpmbuild/SOURCES/nginx.service
	/home/builder/rpmbuild/SOURCES/nginx.upgrade.sh
	/home/builder/rpmbuild/SOURCES/nginx.sysconf
	/home/builder/rpmbuild/SOURCES/nginx-1.6.0.tar.gz
	/home/builder/rpmbuild/SOURCES/nginx.init
	/home/builder/rpmbuild/SOURCES/nginx.suse.init
	/home/builder/rpmbuild/SOURCES/nginx.conf
	/home/builder/rpmbuild/BUILD
	/home/builder/rpmbuild/SRPMS
	/home/builder/rpmbuild/SRPMS/nginx-1.6.0-1.el6.ngx.src.rpm
	/home/builder/rpmbuild/SRPMS/nginx-1.6.0-1.fc21.src.rpm
	/home/builder/rpmbuild/SRPMS/nginx-1.0.15-5.el6.src.rpm



Make a copy of the spec file

cp nginx.spec nginx-naxsi.spec

Edit the spec file:

Change the name to nginx-naxsi


High performance web server

Summary: High performance web server
Name: nginx
Version: 1.6.0
Release: 1%{?dist}.ngx
Vendor: nginx inc.
URL: http://nginx.org/



Change
Summary: High performance web server 
to:
Summary: Nginx High performance web server with Naxsi Web Application Firewall 

Remove Vendor tag

Edit the description to add a line describing Naxsi:

	Naxsi is a Web Application Firewall (WAF).


Change the name of the tar file and retar

cd /home/builder/rpmbuild/SOURCES/
cp nginx-*.gz temp/
cd temp
tar xzf *.gz
mv nginx-1.6.0 nginx-naxsi-1.6.0
tar -zcvf nginx-naxsi-1.6.0.tar.gz nginx-naxsi-1.6.0
mv nginx-naxsi*.gz ../
cd ..
rm temp

you should have the following .tar.gz files in your SOURCES directory, the original one, and the new renamed one:
	* nginx-1.6.0.tar.gz
	* nginx-naxsi-1.6.0.tar.gz





Add the naxsi module

ref:[http://www.prateekn.com/2012/06/re-building-rpm-of-nginx-with-added.html]()

download the module	

mkdir ~/downloads
cd ~/downloads

note: For links to the latest release, see [https://github.com/nbs-system/naxsi/releases]()  

		wget https://github.com/nbs-system/naxsi/archive/0.53-2.tar.gz

Modify the tar.gz to include the package's name 'naxsi':  

		#untar
		tar -xzf  0.53-2.tar.gz
		#delete old archive
		rm 0.53-2.tar.gz
		#tar up with new name
		tar -zcvf  naxsi-0.53-2.tar.gz naxsi-0.53-2
		#remove expanded directory
		rm -R naxsi-0.53-2



Or Simply:  


	wget https://github.com/nbs-system/naxsi/archive/0.53-2.tar.gz
	mv 0.53-2.tar.gz /home/builder/rpmbuild/SOURCES/naxsi-0.53-2.tar.gz



Or even simpler:  

		#this downloads and moves nginx to a new file in the SOURCEs directory, right where we need it
		wget -O /home/builder/rpmbuild/SOURCES/naxsi-0.53-2.tar.gz https://github.com/nbs-system/naxsi/archive/0.53-2.tar.gz



find the Source section, beginning with 'Source0:...', e.g:

		...
		Source5: nginx.vh.default.conf
		Source6: nginx.vh.example_ssl.conf
		Source7: nginx.suse.init
		Source8: nginx.service
		Source9: nginx.upgrade.sh
		...

and add a Source with the name of the tar.gz file you downloaded,e.g:

		Source10: naxsi-0.53-2.tar.gz

look for the following lines

		%prep
		%setup -q

Now add instructions below these lines that tell how to untar the module

		%{__tar} zxvf %{SOURCE10}
		%setup -T -D -a 10

The '10' in the instruction below should match your source number. Your configuration will probably be different.


The above procedure was provided by [prateekn](http://www.prateekn.com/2012/06/re-building-rpm-of-nginx-with-added.html) and here is the explanation given:

> These lines define unpacking of the SOURCE10 file... -T switch disables the automatic unpacking of the archive. -D switch tells the %setup command not to delete the directory before unpacking and -a switch tells the %setup command to unpack only the source directive of the given number after changing directory.


Now add the necessary configure instructions:

To the very first line of the./configure instructions, add this line:

		--add-module=%{_builddir}/%{name}-%{version}/naxsi-0.53-2/naxsi_src \

where 0.53-2 should be changed to match the name of the tar file

Note: Its important that this be added as the first configure instruction  per the nginx 'IMPORTANT' note at the bottom of their wiki page here.[https://github.com/nbs-system/naxsi/wiki/installation]()



In addition to adding a module to nginx, the user will need to configure a basic set of naxsi rules. In order to do this, they will need a file called 'naxsi_core.rules' , located in the naxsi_config directory after the naxsi module has been expanded. It will need to be moved to the /etc/nginx directory for it to be usable.

So lets tell the package manager to install the file to /etc/nginx:

add the following lines:

locate the `%install` section and add the following installation line which will copy the naxsi_core.rules file to the /etc/nginx directory.

		# install naxsi core rules
		%{__install} -m 644 -p %{_builddir}/%{name}-%{version}/naxsi-%{naxsi_version}/naxsi_config/naxsi_core.rules \
		   $RPM_BUILD_ROOT%{_sysconfdir}/nginx/naxsi_core.rules


locate the `%config(noreplace) %{_sysconfdir}/sysconfig/nginx` and add the following line right after it:

		#add core rules file
		%config(noreplace) %{_sysconfdir}/nginx/naxsi_core.rules

* The installation instructions also tell you to create a mysite.rules file, which we'll also include:

	Add the following lines to a file called naxsi-mysite.rules and add it to /home/builder/rpmbuild/SOURCES


			 LearningMode; #Enables learning mode
			 SecRulesEnabled;
			 #SecRulesDisabled;
			 DeniedUrl "/RequestDenied";
			 ## check rules
			 CheckRule "$SQL >= 8" BLOCK;
			 CheckRule "$RFI >= 8" BLOCK;
			 CheckRule "$TRAVERSAL >= 4" BLOCK;
			 CheckRule "$EVADE >= 4" BLOCK;
			 CheckRule "$XSS >= 8" BLOCK;

* Now similar to the naxsi_core.rules, we'll add the instructions to the spec file to tell the rpm where to add it : 

		Source11: naxsi-mysite.rules

		# install naxsi mysite rules 
		%{__install} -m 644 -p %{SOURCE11} \
		$RPM_BUILD_ROOT%{_sysconfdir}/nginx/naxsi-mysite.rules
		#add mysite rules
		%config(noreplace) %{_sysconfdir}/nginx/naxsi-mysite.rules



So when you are done, you should have a file that has instructions similar to the following (only lines changed from the original nginx.spec are listed)

		...
		%define naxsi_version 0.53-2
		...
		
		Summary: Nginx High performance web server with Naxsi Web Application Firewall
		Name: nginx-naxsi

		...


		Source10: naxsi-%{naxsi_version}.tar.gz
		Source11: naxsi-mysite.rules

		...

		%description
		nginx [engine x] is an HTTP and reverse proxy server, as well as
		a mail proxy server. Naxsi is a Web Application Firewall (WAF).

		...

		
		%prep
		%setup -q
		%{__tar} zxvf %{SOURCE10}
		%setup -T -D -a 10
		
		...
		%build
		./configure \
				--add-module=%{_builddir}/%{name}-%{version}/naxsi-%{naxsi_version}/naxsi_src \

		...

		
		./configure \
				--add-module=%{_builddir}/%{name}-%{version}/naxsi-%{naxsi_version}/naxsi_src \
        

		...
		
		# install naxsi core rules
		%{__install} -m 644 -p %{_builddir}/%{name}-%{version}/naxsi-%{naxsi_version}/naxsi_config/naxsi_core.rules \
		   $RPM_BUILD_ROOT%{_sysconfdir}/nginx/naxsi_core.rules
		   
		# install naxsi mysite rules 
		%{__install} -m 644 -p %{SOURCE11} \
		$RPM_BUILD_ROOT%{_sysconfdir}/nginx/naxsi-mysite.rules
		
		...


		#add core rules file
		%config(noreplace) %{_sysconfdir}/nginx/naxsi_core.rules
		
		#add mysite rules
		%config(noreplace) %{_sysconfdir}/nginx/naxsi-mysite.rules
		


		...
		
		%changelog
		
		* Mon May 19 2014 Andrew Druffner <andrew@joelhost.com>
		- 1.6.0
		- added Naxsi web module
		...


**Build the rpm**
		


		rpmbuild -bb ~/rpmbuild/SPECS/nginx-naxsi.spec

Now test the package

cd /home/builder/rpmbuild/RPMS/x86_64

sudo yum remove nginx


#####Create a Local Repository

Once you've created your own custom rpm, you'll want to place it in a repository so you don't have to remember where you last put it, so all you'll have to do is do something like:
yum install enablerepo=myrepo mypackage


1. create the directories that will hold our repo


	sudo mkdir -p /opt/repo/x86_64/debug /opt/repo/x86_64/rpms /opt/repo/x86_64/srpms
	sudo chown -R builder:builder /opt/repo

2. add2repo script

	Create a script that moves the rpms created in the build process to our repo directories.**

 	After moving the rpms, it will re-rerun the createrepo command, which is necessary to keep the repo database up to date.

		mkdir ~/scripts
		nano ~/scripts/add2repo.sh

	add the following lines to the file and save it
	
			#!/bin/bash
		
			###################
			# add2repo.sh
			# Usage: add2repo
			# Adds any repos in build path to their respective repo directories
			###################
			
			
			###################
			#
			# Settings
			###################
			
			repo_path=/opt/repo/x86_64
			build_path=/home/builder/rpmbuild
			#########################
			
			rpm="$@"
			
			## move the debuginfo to the repo
			mv "$build_path"/RPMS/x86_64/*-debug-*.rpm "$repo_path"/debug/
			mv "$build_path"/RPMS/x86_64/*-debuginfo-*.rpm "$repo_path"/debug/
			
			#move the binary to the debug repo
			mv "$build_path"/RPMS/x86_64/*.rpm "$repo_path"/rpms/
			
			#move the source rpm to the source repo
			mv "$build_path"/SRPMS/*.rpm "$repo_path"/srpms/
			
			
			#now rebuild repo so it recognizes new rpms
			createrepo  "$repo_path"/debug/;
			createrepo "$repo_path"/rpms/;
			createrepo "$repo_path"/srpms/;
	
	
	Be sure to edit the repo\_path and build\_path variables to match where you created these directories if the locations are different than shown.
	
			###################
			#
			# Settings
			###################
			
			repo_path=/opt/repo/x86_64
			build_path=/home/builder/rpmbuild
			#########################		
	
	
	Make it executable:
	
			chmod o+x ~/scripts/add2repo.sh

Now when you are satisfied with your build, you can just run the add2repo.sh script and it will 

To make this an alias so it runs without the full path and the sh extension, do this :

echo 'alias add2repo=~/scripts/add2repo.sh' >> ~/.bashrc;source ~/.bashrc;


3. Create a opt.repo file that tells yum where your local repo is: 


		sudo nano /etc/yum.repos.d/opt.repo

add the following lines and save
     
	[opt]
	name=Optional Local Packages - $basearch
	baseurl=file:///opt/repo/x86_64/rpms/
	enabled=1
	gpgcheck=0
	#gpgkey=file:///path/to/you/RPM-GPG-KEY
	
	[opt-debuginfo]
	name=Optional Local Packages - $basearch
	baseurl=file:///opt/repo/x86_64/debug/
	enabled=1
	gpgcheck=0
	#gpgkey=file:///path/to/you/RPM-GPG-KEY
	
	[opt-source]
	name=Optional Local Packages - $basearch
	baseurl=file:///opt/repo/x86_64/srpms/
	enabled=1
	gpgcheck=0
	#gpgkey=file:///path/to/you/RPM-GPG-KEY



Note: Ideally, you'd also add the path to your gpgkey. 




#####How to Add a File






####Problems

**Error: No Package found for systemd**

This can happen when you install a fedora package based on fedora 15 or above, which uses systemd. systemd isn't available for centOS 6.5, so you'll need to find another package. Try redhad....


**Could not open local rpm file:.... RPM Error opening Package:**

This happens because the keycheck fails due to a bug ... just us --nogpgcheck option 


**Installed (but unpackaged) file(s) found: /etc/nginx/naxsi_core.rules**

When you install individual files, you need to list the file and install them. so add thefollowing line:

		%config(noreplace) %{_sysconfdir}/nginx/naxsi_core.rules


**Error Downloading Packages:...[Errno 256] No more mirrors to try*

When you use sudo yum install, it tells you the package is avaiable, but when you say yes to install it, it fails with a message saying that the repo isn't available .

	#sometimes yum wont see the new package because of caching issues.
	#use the following line to clear the cache, replacing 'opt' with the name of your repo
	
	sudo yum clean expire-cache --enablerepo=opt --disablerepo="*"

	sudo yum clean expire-cache 

**You get 404 instead of 500 Server Error**
This is because you aren't providing a 500 server page. Look at your config files and makes sure you have a line like this:

	    error_page   500 502 503 504  /500.html;
	    location = /500.html {
	    }

and that /500.html actually exists



**Tip:** To help Google sources for what you are looking for use similar keywords:

	fedora source rpm for nginx 1.6 src.rpm

or

	nginx-1.6.0-1.el6 SRPMS



**Tip**

An easy way to remember the rebuild options:

	( -bb for 'build binary') 
	( -bs for 'build source')
	( -ba for 'build all')
( -bi to only create the buildroot (no rpms)

-----------------------
 Useful Commands: 

* rpmdev-setuptree
* creates directories BUILD  RPMS  SOURCES  SPECS  SRPMS  under the rpmbuild directory that you specified in your ~/.rpmmacros file.

* rpmdev-wipetree
* deletes all the files and subdirectories from rpmbuild, keeping the directory structure intact. You'll be left with empty rpmbuild/BUILD,RPMS,SOURCES,SPECS,SRPMS directories


#####How to Configure Naxsi

sudo nano /etc/nginx/nginx.conf

add the following line under the http block

		include                        /etc/nginx/naxsi_core.rules;


		sudo nano /etc/nginx/conf.d/joelhost.com.conf

add the following line: 

	#add naxsi rules
	include           /etc/nginx/naxsi-mysite.rules;



reload

test, pass this : 
	
	http://joelhost.com/?a=%3C

now check logs



For logs to work properly.
rm logs
actually have to reset the server so the log can be recreated
then request using http://joelhost.com?a=%3C
