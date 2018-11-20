---
layout: single
title:  "SSH PIV authentication on MacOS without errors"
date:   2018-11-20 21:51:16 +0100
categories: yubikey macos ssh
---
My Yubikey has become increasingly useful over the past month. So far my main usage was 2 factor authentication using U2F for Facebook, Gitbhub and Archlinux laptop. I also use it to authenticate to my mac.

I used it as well to store my ssh keys (see: https://developers.yubico.com/PIV/Guides/SSH_user_certificates.html and https://developers.yubico.com/PIV/Guides/SSH_with_PIV_and_PKCS11.html). However, I always faced the following problem:
{% highlight bash %}
bash:~ somos$ ssh-add -s /usr/local/lib/libykcs11.dylib
Enter passphrase for PKCS#11:
Could not add card "/usr/local/lib/libykcs11.dylib": agent refused operation
bash:~ somos$ launchctl list|grep ssh
698	0	com.openssh.ssh-agent
{% endhighlight %}


You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight bash %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
