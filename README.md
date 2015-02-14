docker-image-strip
==================
Experimental utility to "strip" docker images down to the bare minimum required to run an application.

The main idea behind this is based on the proposal [here](https://lists.projectatomic.io/projectatomic-archives/atomic-devel/2015-February/msg00016.html)

Right now things are just being prototyped as a proof of concept. If things materialize, more documentation will be written.

Example
-------
The proof of concept works for binary executables that are dynamically linked (relying on ldd for now).


```shell
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
fedora              21                  834629358fe2        6 weeks ago         250.2 MB
fedora              latest              834629358fe2        6 weeks ago         250.2 MB

$ ./dis -i fedora:21 -e ‘/bin/bash’

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
fedora              21-stripped         1694d8015aa4        3 seconds ago       3.451 MB
fedora              21                  834629358fe2        6 weeks ago         250.2 MB
fedora              latest              834629358fe2        6 weeks ago         250.2 MB

$ docker run -ti fedora:21-stripped /bin/bash
bash-4.3# ls
bash: ls: command not found
bash-4.3# echo “it worked”
it worked
bash-4.3# exit
exit

```

#### NOTES
  * This needs to be run as root or as sudo (at least for now) becuase it runs `chroot` and some of the layered tarballs from docker images that get extracted often try to set perms that normal users can’t.
  * Requires python3 because of an interesting bug in python2’s tarfile implementation that occurs only on *some* images tarball layers, example:
```
$ ./dis -i centos7:httpd -e '/usr/sbin/httpd'
Traceback (most recent call last):
  File "./dis", line 269, in <module>
    create_chroot(working_dir, from_chroot, image_layers)
  File "./dis", line 145, in create_chroot
    layer_tar.extractall(path=chrootdir)
  File "/usr/lib64/python2.7/tarfile.py", line 2041, in extractall
    for tarinfo in members:
  File "/usr/lib64/python2.7/tarfile.py", line 2471, in next
    tarinfo = self.tarfile.next()
  File "/usr/lib64/python2.7/tarfile.py", line 2319, in next
    tarinfo = self.tarinfo.fromtarfile(self)
  File "/usr/lib64/python2.7/tarfile.py", line 1242, in fromtarfile
    return obj._proc_member(tarfile)
  File "/usr/lib64/python2.7/tarfile.py", line 1264, in _proc_member
    return self._proc_pax(tarfile)
  File "/usr/lib64/python2.7/tarfile.py", line 1394, in _proc_pax
    value = value.decode("utf8")
  File "/usr/lib64/python2.7/encodings/utf_8.py", line 16, in decode
    return codecs.utf_8_decode(input, errors, True)
UnicodeDecodeError: 'utf8' codec can't decode byte 0xc0 in position 4: invalid start byte
[
```
  * This unfortunately leaves RHEL/CentOS 7 out of the mix as a host for performing the stripping on until python3 makes it into EPEL7[[0]](https://lists.fedoraproject.org/pipermail/epel-devel/2015-January/010700.html)[[1]](https://fedoraproject.org/wiki/User:Bkabrda/EPEL7_Python3)
  * The following is a possible work around, but you require access to python33 from [SoftwareCollections.org](https://www.softwarecollections.org/) or [RHSCL](https://access.redhat.com/documentation/en-US/Red_Hat_Software_Collections/)
```
$ scl enable python33 'bash'

$ virtualenv docker-py
Using base prefix '/opt/rh/python33/root/usr'
New python executable in docker-py/bin/python3
Also creating executable in docker-py/bin/python
Installing Setuptools..............................................................................................................................................................................................................................done.
Installing Pip.....................................................................................................................................................................................................................................................................................................................................done.

$ . docker-py/bin/activate

(docker-py)$ pip install docker-py
Downloading/unpacking docker-py
  Downloading docker-py-0.7.2.tar.gz
  Running setup.py egg_info for package docker-py

Downloading/unpacking requests>=2.2.1,<2.5.0 (from docker-py)
  Downloading requests-2.4.3.tar.gz (438kB): 438kB downloaded
  Running setup.py egg_info for package requests

Downloading/unpacking six>=1.3.0 (from docker-py)
  Downloading six-1.9.0.tar.gz
  Running setup.py egg_info for package six

    no previously-included directories found matching 'documentation/_build'
Installing collected packages: docker-py, requests, six
  Running setup.py install for docker-py

  Running setup.py install for requests

  Running setup.py install for six

    no previously-included directories found matching 'documentation/_build'
Successfully installed docker-py requests six
Cleaning up...

(docker-py)$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
centos7                 httpd               4cbfe5b8029c        3 hours ago          283.1 MB
centos7                 nginx               1bc828ee469a        33 hours ago         314.3 MB
centos                  centos7             dade6cb4530a        7 days ago           224 MB

(docker-py)$  cd ~/src/dev/docker-image-strip/ # where ever you have this repo cloned

(docker-py)$ ./dis -i 'centos7:httpd' -e '/usr/sbin/httpd'

(docker-py)$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
centos7                 httpd-stripped      e749b5af69d0        26 seconds ago       6.585 MB
centos7                 httpd               4cbfe5b8029c        3 hours ago          283.1 MB
centos7                 nginx               1bc828ee469a        33 hours ago         314.3 MB
centos                  centos7             dade6cb4530a        7 days ago           224 MB
```
