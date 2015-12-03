### To set up machine to build OMS

There are two ways to build OMS.

1. As an RPM or DEB package that can be installed on the local system,
assuming all dependencies are met (most easily met if a shell bundle
was previously installed to "seed" the system), or
2. As a shell bundle.

Building a shell bundle is a superset of of setting up a system to
build a local RPM, so we will cover that first.

#### To set up system to build a native package

Note that it's very nice to be able to use the [updatedns]
(https://github.com/jeffaco/msft-updatedns) project to
use host names rather than IP numbers. On CentOS systems, this requires
the bind-utils package (updatedns requires the 'dig' program). The
bind-utils package isn't otherwise necessary.

- On CentOS 7.x
```
 sudo yum install git bind-utils ruby bison gcc-c++ rpm-devel pam-devel openssl-devel rpm-build
```
- On Ubuntu 14.04
```
 sudo apt-get install git pkg-config make ruby bison g++ rpm librpm-dev libpam0g-dev libssl-dev
```

- Notes on other platforms

 When building a machine for ULINUX builds (such as CentOS 5.x), we suggest
 using the O/S distribution CD to install the packages. It's not as easy,
 but that's the only way to guarentee that packages aren't updated such that
 generated binaries are not backwards compatible. (See notes on building a
 shell bundle, elsewhere in this document.)

 Also note that since you won't use 'yum', you must also handle the dependent
 packages manually (keep adding lines to the 'rpm install' command line until
 all dependencies are satisfied).

 Similar methods would be utilized if building a Redhat system that is not
 registered for use for up2date.

##### Notes on sudoers configuragion

Two changes should be made to your sudoers configuration for omsagent to
build properly:

1. Configure sudoers to not require a TTY.

 Some platforms require a TTY be default, and this can be problematic for
 background builds. If you have a line like:

 ```Defaults    requiretty```

 then comment it out (like this):

 ```#Defaults    requiretty```

2. Configure your account to not require a password by adding the NOPASSWD:
qualifier to the appropriate line that affects your account. After the correct
changes are applied to /etc/sudoers, test under your personal account with the
following sequence:

 ```shell
 sudo -k
 sudo ls
 ```

 If there is no password prompt, then /etc/sudoers was correctly modified.

#### To set up system to build a shell bundle

To build a shell bundle, we need and older Linux system (we typically use
CentOS 5.0 for this), as binary images created with older Linux systems
are generally upwards compatible when installed on newer Linux systems.

A notable exception: We use the OpenSSL package, and we can't tell if
we need OpenSSL v0.9.8 or OpenSSL v1.0.x. As a result, we have a [special
process] (OPENSSL.md)  to build both both versions of OpenSSL that we can
link against.

Once OpenSSL is set up, you need to configure omsagent to include the
```--enable-ulinux``` qualifier, like this:<br>```./configure --enable-ulinux``` 

### To check out source code repository

To check out the source code repository, you need a command like:

```git clone --recursive git@github.com:MSFTOSSMgmt/bld-omsagent.git```

Note that there are several subprojects, and authentication is a hassle
unless you set up an SSH key via your github account. We would strongly
suggest setting up an SSH key with a passphrase, adding the public key
to your github account, and then add the private key to your SSH program
to act as an SSH agent. The exact procedure differs based on the SSH
program that you use, so ask a fellow developer how to set this up if
you're not sure.

The end result of this mechanism: You specify the password once when you
start your SSH program, and then you never type the passphrase again.

### To build omsagent

From the bld-omsagent directory (created above from 'git clone', do the
following:

```
cd omsagent/build
./configure
make
```

Note that the ```configure``` script takes a variety of options. You can
use ```configure --help``` to see the options available.

When this completes you should have a native package that you can install
on your system. The native package should be in a subdirectory off of
bld-omsagent/omsagent/target (the directory name varies based on debug vs.
release builds).

As mentioned above, this form a build requires a shell bundle to be installed
to "seed" your system with other required dependencies.

Note that the build will build multiple versions of Ruby (one for unit test,
only built once, even after "make distclean"), and then one or two versions
depending if you're building for a shell bundle or not. Bottom line: The
first build is always slower than subsequent builds.
