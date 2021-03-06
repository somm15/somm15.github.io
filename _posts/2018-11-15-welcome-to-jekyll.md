---
layout: single
title:  "SSH PIV authentication on MacOS without errors"
date:   2018-11-20 21:51:16 +0100
categories: yubikey macos ssh
---
Tired of getting the `Could not add card "/usr/local/lib/libykcs11.dylib": agent refused operation` error when trying to use your yubikey with SSH?

My Yubikey became increasingly useful over the past months. I mainly use it for 2 factor authentication using U2F for Facebook, Gitbhub and Archlinux laptop. I also use it to authenticate to my mac.

I also use it to store my ssh keys (see: [SSH-PIV]  and [SSH-PIV-CERT]). I used a mix of both guides to have ssh certificates but use the slot a (with a pin rather than a touch). I created the keys and certificate in slot A using the GUI rather than the command line.

**Warning!** Make sure to generate RSA2048, ECC doesn't work with ssh.
{: .notice--danger}

I always faced the following problem:
{% highlight bash %}
bash:~ somos$ ssh-add -s /usr/local/lib/libykcs11.dylib
Enter passphrase for PKCS#11:
Could not add card "/usr/local/lib/libykcs11.dylib": agent refused operation
bash:~ somos$ launchctl list|grep ssh
698	0	com.openssh.ssh-agent
{% endhighlight %}

I found so many bad suggestions online to solve this problem. I read posts suggesting to disable the "System Integrity Protection" or recommending to delete symlink and copy files. This decreases the security of your Mac. SIP is useful and copying the file could cause your next update to fail or be inefficient.

Let's try to understand the problem.

MacOS Sierra introduced restriction to the ssh-agent restricting folder from which PKCS#11 can be loaded. The default folders are `/usr/lib/*,/usr/local/lib/*`. None of them are stored in these directories.
{% highlight bash %}
bash:~ somos$ ls -l /usr/local/lib/libykcs11.dylib
lrwxr-xr-x  1 somos  admin  51 13 oct 17:15 /usr/local/lib/libykcs11.dylib -> ../Cellar/yubico-piv-tool/1.6.2/lib/libykcs11.dylib
bash:~ somos$ ls -l /usr/local/lib/opensc-pkcs11.so
lrwxr-xr-x  1 somos  admin  44 13 oct 21:02 /usr/local/lib/opensc-pkcs11.so -> ../Cellar/opensc/0.19.0/lib/opensc-pkcs11.so
{% endhighlight %}

Why would you delete the symlinks and copy the files to `/usr/local/lib/`? The ssh-agent has an option for that.
Make sure to always launch your ssh-agent using the options -P:
{% highlight bash %}
eval  `ssh-agent -s  -P /usr/local/lib/*,/usr/local/Cellar/opensc/*/lib/*.so,/usr/local/opt/opensc/lib/*.so,/usr/local/Cellar/yubico-piv-tool/*/lib/*.dylib`
{% endhighlight %}

**Warning!** Launch the ssh-agent before using ssh-add otherwise, MacOS will start a default ssh-agent automatically
{: .notice--danger}

**Beta** I wrote a plist file [Gist-plist] to launch the homebrew ssh-agent with the proper options rather than the default one. However, I can execute it only once, the second time will fail. I didn't solve it yet.
{: .notice--success}

To make it easier, I added the following lines to my .bash_profile:
{% highlight bash %}
export SSH_AUTH_SOCK=$SSH_AUTH_SOCK_LOCAL
alias ssh-agent='eval `/usr/local/bin/ssh-agent -s  -P /usr/local/lib/*,/usr/local/Cellar/opensc/*/lib/*.so,/usr/local/opt/opensc/lib/*.so,/usr/local/Cellar/yubico-piv-tool/*/lib/*.dylib`'
alias yubinit='eval `/usr/local/bin/ssh-agent -s  -P /usr/local/lib/*,/usr/local/Cellar/yubico-piv-tool/*/lib/*.dylib`;ssh-add -s /usr/local/lib/libykcs11.dylib'
{% endhighlight %}

The yubinit alias launches ssh-agent and adds the yubikey PKCS11 provider.

**Tip** The ssh version shipped with MacOS is fine. However, I would suggest installing the [Homebrew] version.
{: .notice--success}

[SSH-PIV]: https://developers.yubico.com/PIV/Guides/SSH_with_PIV_and_PKCS11.html
[SSH-PIV-CERT]: https://developers.yubico.com/PIV/Guides/SSH_user_certificates.html
[Gist-plist]: https://gist.github.com/somm15/cf92d1ce5708a0a153b4157d6813053b
[Homebrew]: https://brew.sh/
