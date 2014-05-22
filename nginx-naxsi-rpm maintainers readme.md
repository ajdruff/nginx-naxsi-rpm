##nginx-naxsi rpm Maintainer Notes

These notes are intended for simpliwp contributors that build new rpms for the simpliwp/rpm-repos repository using the nginx-naxsi rpmbuild files to do so. They are *not* intended for developers who download the nginx-naxsi rpmbuild files to build their own custom rpm.




The nginx-naxsi rpm is contained on a GitHub project located at [https://github.com/simpliwp/nginx-naxsi-rpm](). Each time a new rpm is built, you should release the files you used to create the rpm, as well as releasing the rpm to the [simpliwp/rpm-repos/centos6](https://github.com/simpliwp/rpm-repos) repository.


Disclaimer: This is a quick and dirty brain dump of what you have to do to update the rpm. It will likely have errors, so think about what you are doing!



#####How to Release a new nginx-naxsi RPM



When you release a new RPM, you'll be editing the ~/rpmbuild files 

1. start an instance of the build server
2. ssh ec2-user, then su - builder
3. cd ~/rpmbuild
4. git branch to confirm you are on the dev branch. if not on dev branch then checkout branch dev
5. make your changes
6. Edit SPECS/nginx-naxsi.spec
7. if updating naxsi: 
8. update the %naxsi-version macro
9. download the new naxsi tarball and add it to the nginx-naxsi sources. Rename it using the following procedure

> note: For links to the latest release, see [https://github.com/nbs-system/naxsi/releases]()  
> 
		#this downloads and moves nginx to a new file in the SOURCEs directory, right where we need it
		wget -O /home/builder/rpmbuild/SOURCES/naxsi-0.53-2.tar.gz https://github.com/nbs-system/naxsi/archive/0.53-2.tar.gz


8. Update the sources and files list if needed to match the new name
9. bump the release number if the nginx version hasn't changed. if it has, then leave the release at whatever the nginx tarball has.
10. if updating nginx tarball:

	The safest thing to do is to start from scratch to build a new rpmproject ( see 'How to Build Your Own Nginx RPM from Source') . You can then download the last release, and compare. Once you've followed that procedure, then do a line by line comparison of the nginx-naxsi spec from the previous release, and edit as necessary. 


11. When you are done and ready to release:
12. su ec2-user
13. cd /opt/repo
14. git branch
15. #confirm you are on dev branch, if not git checkout dev
16. su - builder
17. add2repo #this will move all the rpms to the local repo
18. If you get permission denied errors, chown /opt/repo to builder:builder. 
13. exit out from the builder user
14. #commit the updated repo
15. git commit -a -m 'added updated nginx-naxsi vX.X.X rpm'
15. git push origin centos6

Note: 'builder' user cannot communicate with GitHub so will get errors if tries to push, so always use the ec2-user. The add2repo is a builder alias so you need to be builder to use it.



About Versioning: 

* The spec file's 'Version' number will always be identical to the nginx version number that you are packaging.
* Bump the release number each time you make a change to the package that contains a previously released nginx version
* There are separate macro variables, nginx-version and naxsi-version . always update these for your latest release, and they should always include the full version number, including release.
* the version of the included nginx and naxsi packages are included in the 'description' , so if you need to know, you can query the rpm using yum info nginx-naxsi and read the description






####How to Update the Nginx-Naxsi-Rpm Project

#####Outline

- merge changes from last update with master
- merge changes from master with dev branch
- work in dev branch until you are satisfied with changes
- create a branch based on dev with new version, named after nginx version + the release number

git Commands

		git commit -a -m 'completed dev, ready for release'
		git checkout master
		git merge --no-ff dev #merges release into the master
		git tag -a XXX
		git branch



dev 
master 


simplified:

####Simplified Git Branch Management

* 2 branches, Dev and Master
* Dev is always a work in progress and is where you develop from
* Master is always the latest release

Assumptions: 
* You will always be working in dev.
* Creating a new branch will rarely be done, because you always working on the main dev branch. When you do create a new branch, it will be basically a fork of dev, and will be remerged into dev when you are ready to release it. 

Most Frequent workflow:

1. work in dev
2. when you are ready to release, you commit to dev,merge it into master and then tag it with the release number:

		#commit dev changes
		git commit -a -m 'completed dev, ready for release x.x.x'
		#merge dev into master:
		git checkout master
		git merge --no-ff dev
		git -a -m 'merged dev into master for release x.x.x'
		#tag master
		git tag -a x.x.x
		#push to remote
		git push origin dev
		git push origin master	
		#push your tags to remote
		git push --tags


Workflow #2 - Sometimes you may have created a branch from dev to work on a new feature.

		#merge changes from branch into dev
		git checkout dev
		git merge --no-ff branchname
		#Now follow Workflow #1

		#Now delete branch so as to prevent any more changes. You can always recreate it using the tag
		git branch -d
		


tip: to convert branch to tag where branchname is the branch you want to turn into tag and then delete

	git checkout branchname
	git status (to make sure all changes have been committed)
	#create the tag
	git tag -a tagname
	git checkout master ( if it doesn't let you , then may have to commit again)
	#delete branch
	git branch -D branchname



