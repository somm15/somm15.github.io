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

I found so many bad suggestions online to solve this problem. I found posts suggesting to disable the "System Integrity Protection" or  

[yubikey-SSH_PIV]: https://developers.yubico.com/PIV/Guides/SSH_user_certificates.html
[yubiket-SSH_PIV2]: https://developers.yubico.com/PIV/Guides/SSH_with_PIV_and_PKCS11.html
