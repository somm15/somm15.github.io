---
layout: single
title:  "SSH PIV authentication on MacOS without errors"
date:   2018-11-20 21:51:16 +0100
categories: yubikey macos ssh
---
My Yubikey has become increasingly useful over the past month. So far my main usage was 2 factor authentication using U2F for Facebook, Gitbhub and Archlinux laptop. I also use it to authenticate to my mac.

I used it as well to store my ssh keys (see: [yubikey-SSH_PIV]  and [yubiket-SSH_PIV2]). However, I always faced the following problem:
{% highlight bash %}
bash:~ somos$ ssh-add -s /usr/local/lib/libykcs11.dylib
Enter passphrase for PKCS#11:
Could not add card "/usr/local/lib/libykcs11.dylib": agent refused operation
bash:~ somos$ launchctl list|grep ssh
698	0	com.openssh.ssh-agent
{% endhighlight %}

I found so many bad suggestions online to solve this problem. I found posts suggesting to disable the "System Integrity Protection" or recommending to delete symlink and copy files. This decreases the security of your Mac. SIP is useful and copying the file could cause your next update to fail or be inefficient.

Let's try to understand the problem.

MacOS Sierra introduced restriction to the ssh-agent restricting folder from which PKCS#11 can be loaded. The default folders are `/usr/lib/*,/usr/local/lib/*`. None of them are stored in these directories.
{% highlight bash %}
bash:~ somos$ ls -l /usr/local/lib/libykcs11.dylib
lrwxr-xr-x  1 somos  admin  51 13 oct 17:15 /usr/local/lib/libykcs11.dylib -> ../Cellar/yubico-piv-tool/1.6.2/lib/libykcs11.dylib
bash:~ somos$ ls -l /usr/local/lib/opensc-pkcs11.so
lrwxr-xr-x  1 somos  admin  44 13 oct 21:02 /usr/local/lib/opensc-pkcs11.so -> ../Cellar/opensc/0.19.0/lib/opensc-pkcs11.so
{% endhighlight %}
Why would you delete the symlinks and copy the files to `/usr/local/lib/`? The ssh-agent has an option for that.
Make sure to always launch your ssh-agent, before using ssh-add, using the options -P:
{% highlight bash %}
eval  `ssh-agent -s  -P /usr/local/lib/*,/usr/local/Cellar/opensc/*/lib/*.so,/usr/local/opt/opensc/lib/*.so,/usr/local/Cellar/yubico-piv-tool/*/lib/*.dylib`
{% endhighlight %}

To make it easier, I added the following lines to my .bash_profile:
{% highlight bash %}
export SSH_AUTH_SOCK=$SSH_AUTH_SOCK_LOCAL
alias ssh-agent='eval `/usr/local/bin/ssh-agent -s  -P /usr/local/lib/*,/usr/local/Cellar/opensc/*/lib/*.so,/usr/local/opt/opensc/lib/*.so,/usr/local/Cellar/yubico-piv-tool/*/lib/*.dylib`'
alias yubinit='eval `/usr/local/bin/ssh-agent -s  -P /usr/local/lib/*,/usr/local/Cellar/yubico-piv-tool/*/lib/*.dylib`;ssh-add -s /usr/local/lib/libykcs11.dylib'
{% highlight bash %}


[yubikey-SSH_PIV]: https://developers.yubico.com/PIV/Guides/SSH_user_certificates.html
[yubiket-SSH_PIV2]: https://developers.yubico.com/PIV/Guides/SSH_with_PIV_and_PKCS11.html
