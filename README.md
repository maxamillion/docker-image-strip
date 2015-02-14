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
    * This unfortunately leaves RHEL/CentOS 7 out of the mix as a host for performing the stripping on until python3 makes it into EPEL7[[0]](https://lists.fedoraproject.org/pipermail/epel-devel/2015-January/010700.html)[[1]](https://fedoraproject.org/wiki/User:Bkabrda/EPEL7_Python3)
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
