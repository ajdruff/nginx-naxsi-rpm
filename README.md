GitHub Project: [nginx-naxsi-rpm](https://github.com/simpliwp/nginx-naxsi-rpmbuild-centos6)
============================

this is a test change

rpm build files used to create a custom Nginx rpm that includes the Naxsi WAF. 


#About

This project provides the rpmbuild files necessary to build a custom rpm for Nginx on CentOS, adding the naxsi Web Application Firewall. 

***Do you just want to install a nginx-naxsi rpm for CentOS?** If all you want to do is download an already compiled rpm for CentOS, you can do so using [simpliwp/rpm-repos](https://github.com/simpliwp/rpm-repos). Save yourself some headaches and don't compile and run this project since I've already built the rpm for you. This project is only for those who want to be able to customize the rpm or use it as a template to create other customized rpms*

References:   

* Nginx - is a free, open-source, high-performance HTTP server and reverse proxy [http://wiki.nginx.org/Main]()
* Naxsi  -  NAXSI is an open-source, high performance, low rules maintenance WAF (Web Application Firewall) for NGINX 
[https://github.com/nbs-system/naxsi/]()
* Naxsi Basic Setup [https://github.com/nbs-system/naxsi/wiki/basicsetup]()
* How to Create an RPM Package [https://fedoraproject.org/wiki/How_to_create_an_RPM_package]()
* FOSS Rules blog post about creating a nginx-naxsi rpm [http://www.prateekn.com/2012/06/re-building-rpm-of-nginx-with-added.html]()


######How to Use this Project

Overview:

These files are intended to be built on a CentOS 6.5 build server with all the development tools installed. It is highly recommended that you do not attempt this on a production server and that you create a non-sudo user specifically for building the rpm.

**Prep Your Build Server** 

	sudo yum  install --enablerepo=epel yum-utils rpmdevtools

(**OR**, since Development Tools includes rpmdevtools)

	sudo yum groupinstall --enablerepo=epel  'Development Tools'
	sudo yum  install --enablerepo=epel yum-utils





**Create Build User**

	sudo useradd builder
	sudo passwd builder 
	su - builder

**Create rpmbuild directory**

If you don't already have a rpmbuild directory in your builder user's root, create one now, or backup your old one:

to create a new rpmbuild directory, login as the 'builder' user created above and run the following:

	rpmdev-setuptree

This will create a rpmbuild directory in the user's home directory, and a .rpmmacros file.


**Download this project**

Download and untar this project

	wget https://github.com/simpliwp/nginx-naxsi-rpmbuild-centos6/archive/master.tar.gz
	tar -xzvf master.tar.gz

now overwrite your rpmbuild directory with its contents

	mv nginx-naxsi-rpmbuild-centos6-master/* ~/rpmbuild/

**Customize the spec**

(optional) 

	nano ~/rpmbuild/SPECS/nginx-naxsi.spec


**Build the rpm**


	rpmbuild -ba ~/rpmbuild/SPECS/nginx-naxsi.spec

Your rpm will be located in	`~/rpmbuild/RPMS/x86_64`

(**OR** if your rpmbuild directory is somewhere other than your user's home directory, try this :

rpmbuild --define "_topdir $HOME/path/to/top/level/directory" -ba /path/to/top/level/directory/SPECS/nginx-naxsi.spec ) 




**Install the rpm**

	sudo yum install ~/rpmbuild/RPMS/x86_64/nginx-naxsi-1.6.0-1.el6.ngx.x86_64.rpm

be sure to match the path and name of the rpm with the file that was produced.

For additional information on how to configure, see the readme for my [nginx-naxsi project]()


**Contributors:**

Andrew Druffner andrew at nomstock.com
