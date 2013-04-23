This repository is a fork of https://github.com/doxavore/warbler144, which was initially created to reproduce the error believed to be
https://github.com/jruby/warbler/issues/144. I'm using the fork to reproduce the issue.

I can demonstrate this issue most effectively by showing how warbler improperly packages the bundler gem for bundler v=1.3.5 while generating the correctly packaged gem directory structure for bundler -v=1.2.4.

Note that this issue only manifests when bundler is run in '--deployment' mode.

Tested Environment
-------------------
Tested with the following configuration:
* OS X 10.8.3
 * `uname -a`: Darwin nonesuch.pop.umn.edu 12.3.0 Darwin Kernel Version 12.3.0: Sun Jan  6 22:37:10 PST 2013; root:xnu-2050.22.13~1/RELEASE_X86_64 x86_64
* rvm 1.19.6 (master) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
* `jruby -v`: jruby 1.7.3 (1.9.3p385) 2013-02-21 dac429b on Java HotSpot(TM) 64-Bit Server VM 1.7.0_17-b02 [darwin-x86_64]
* Bundler 1.2.4 and 1.3.5
* Warbler 1.3.6


Properly Packaged Bundler Gem (bundler -v=1.2.4):
------------
Uninstall existing versions of bundler and executables. Then install bundler -v=1.2.4:
```
gem uninstall -ax bundler && gem install bundler -v=1.2.4
```

cd to the repos root directory and create the jar with:
```
bundle --deployment && bundle exec warble
```

Now examine the structure of the packaged bundler gem:
```
mkdir -p bundler-1.2.4 && mv myapp.jar bundler-1.2.4 && cd bundler-1.2.4 && jar xf myapp.jar

ls -l gems/bundler-1.2.4/
total 144
-rw-rw-r--   1 peterson  staff  40053 Apr 23 08:36 CHANGELOG.md
-rw-rw-r--   1 peterson  staff   2757 Apr 23 08:36 ISSUES.md
-rw-rw-r--   1 peterson  staff   1115 Apr 23 08:36 LICENSE
-rw-rw-r--   1 peterson  staff   1490 Apr 23 08:36 README.md
-rw-rw-r--   1 peterson  staff   6549 Apr 23 08:36 Rakefile
-rw-rw-r--   1 peterson  staff   4175 Apr 23 08:36 UPGRADING.md
drwxrwxr-x   3 peterson  staff    102 Apr 23 08:36 bin
-rw-rw-r--   1 peterson  staff   1243 Apr 23 08:36 bundler.gemspec
drwxrwxr-x   4 peterson  staff    136 Apr 23 08:36 lib
drwxrwxr-x  11 peterson  staff    374 Apr 23 08:36 man
drwxrwxr-x   2 peterson  staff     68 Apr 23 08:36 spec
```

In the above case, the packaged bundler gem has the expected structure (e.g. bin and lib directories are present).

Improperly packaged bundler gem (bundler -v=1.3.5):
------------
Uninstall existing versions of bundler and executables. Then install bundler -v=1.3.5:
```
gem uninstall -ax bundler && gem install bundler -v=1.3.5
```

cd to the root directory and create the jar with:
```
bundle --deployment && bundle exec warble
```

Now examine the structure of the package bundler gem:
```
mkdir -p bundler-1.3.5 && mv myapp.jar bundler-1.3.5 && cd bundler-1.3.5 && jar xf myapp.jar

ls -l gems/bundler-1.3.5/
total 24
drwxrwxr-x  51 peterson  staff   1734 Apr 23 08:55 bundler
-rw-rw-r--   1 peterson  staff  11965 Apr 23 08:55 bundler.rb
```

Note that the packaged bundler gem now has the wrong directory structure (e.g lib and bin directories are missing).

Conclusion:
------------
When a dependency bundle has been installed in '--deployment' mode with bundler -v=1.3.5, warbler incorrectly packages the bundler gem in both jar and war files. In fact, warbler produces this broken directory structure for any version of bundler in the 1.3.x series. Warbler behaves appropriately in this regard when used in conjunction with bundler 1.2.x.

I've been able to show that this mangled directory structure leads directly to the "No such file to load -- bundler/setup" errors we're seeing when attempting to deploy our webapp to Tomcat.