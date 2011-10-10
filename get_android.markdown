How to get android source code from github mirror
==========

Get android source code using repo
----------
* `Since the android.git.kernel.org is down now, so we can't get android source code from there now. But AOSP has a mirror on github, so we can try to get android source code there.`

# Get repo #

* `#cd ~/your_android_dir`
* `#curl https://github.com/android/tools_repo/blob/master/repo > ~/bin/repo`
* `#chmod a+x ~/bin/repo`

# Init repo #

* `#repo init --repo-url=git://github.com/android/tools_repo.git -u git://github.com/android/platform_manifest.git`
* `NOTE:repo-url refers to repo tool url, and -u specifies android manifests`

# Sync repo #

* `#repo sync -j16`

# Congratulations #

* `Now you can work with AOSP`
