:css: hovercraft.css

.. title:: Build Your Own Docker

----

================
Linux Containers
================

----

Process Management
==================

----

Namespaces
==========

----

Chroot
======

----

.. code-block:: console

    # ls -la /proc/self/root
    lrwxrwxrwx 1 root root 0 Jan 19 22:15 /proc/self/root -> /
    # chroot .vagga/build/ /bin/sleep 1000 & pid=$!
    [4] 29988
    # ls -la /proc/$pid/root
    lrwxrwxrwx 1 root root 0 Jan 19 22:15 /proc/29988/root -> /newroot

----

Pivot Root
==========

----

::

    +- /
    |--- /usr
    |--- /dev
    |--- /home
    \-+- /newroot
      |--- /nix
      |--- /dev
      \--- /tmp

----

.. code-block:: console

   # pivot_root /newroot /newroot/tmp
   # umount /tmp

----

::

    +- /  (was /newroot)
    |--- /nix
    |--- /dev
    \-+- /tmp
      |--- /usr
      |--- /dev
      \--- /home

----

.. code-block:: console

    mount --bind /dev /newroot/dev

----

Mount Namespace
===============

----

.. code-block:: console

    # unshare --mount /bin/bash
    # mount --make-rprivate /
    # mount -t tmpfs tmpfs /tmp
    # ls -la /tmp
    total 16
    drwxrwxrwt 2 root root  40 Jan 19 22:32 .
    drwxr-xr-x 1 root root 142 Dec 13 16:14 ..
    #

----

Network Namespace
=================

----

.. code-block:: console

    # ip netns create isolated
    # ip netns exec isolated ip addr
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

----

.. code-block:: console

    # ip netns exec isolated ip link set dev lo up
    # ip netns exec isolated ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever

----

.. code-block:: console

    # ip netns exec isolated wget google.com
    --2015-01-19 22:49:23--  http://google.com/
    Resolving google.com (google.com)... 173.194.113.194, ...
    Connecting to google.com (google.com)|173.194.113.194|:80...
    failed: Network is unreachable.

----

How Wget Resolves IP?
=====================

----

.. code-block:: console

    # ip netns exec isolated \
      strace -f -s 100 wget google.com

----

::

    write(2, "Resolving google.com (google.com)... ", 37Resolving google.com (google.com)... ) = 37
    socket(PF_LOCAL, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 3
    connect(3, {sa_family=AF_LOCAL, sun_path="/var/run/nscd/socket"}, 110) = 0
    sendto(3, "\2\0\0\0\r\0\0\0\6\0\0\0hosts\0", 18, MSG_NOSIGNAL, NULL, 0) = 18
    poll([{fd=3, events=POLLIN|POLLERR|POLLHUP}], 1, 5000) = 1 ([{fd=3, revents=POLLIN}])
    recvmsg(3, {msg_name(0)=NULL, msg_iov(2)=[{"hosts\0", 6}, {"\310O\3\0\0\0\0\0", 8}], msg_controllen=24, {cmsg_len=20, cmsg_level=SOL_SOCKET, cmsg_type=SCM_RIGHTS, {4}}, msg_flags=MSG_CMSG_CLO
    EXEC}, MSG_CMSG_CLOEXEC) = 14

----

Traffic Control
===============

----



