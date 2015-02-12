# docker-image-strip
Experimental utility to "strip" docker images down to the bare minimum required to run an application.

The main idea behind this is based on the proposal [here](https://lists.projectatomic.io/projectatomic-archives/atomic-devel/2015-February/msg00016.html)

Right now things are just being prototyped as a proof of concept. If things materialize, more documentation will be written.

## Example
The proof of concept works for binary executables that are dynamically linked (relying on ldd for now).

NOTE: This needs to be run as root or as sudo (at least for now) becuase it runs `chroot` and some of the layered tarballs from docker images that get extracted often try to set perms that normal users can’t.

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
