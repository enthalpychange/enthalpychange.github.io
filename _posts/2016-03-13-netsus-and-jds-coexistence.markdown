---
layout: post
title:  "NetSUS and JDS coexistence"
tags:
- casper suite
- apache
---

One of the Casper Suite’s components is the JDS, which hosts packages used by the JSS. They also have a product called NetSUS, which uses several open-source projects to mimic Apple’s NetBoot and SUS. When installing both the JDS and NetSUS on the same RHEL 7 server, you will run into some issues since they both use Apache. In this post, we will take a look at how the Apache config is modified when each one is installed and how we can get them to coexist.


<div class="post-heading1">Installing the JDS</div>

JAMF has a [list of the files][jdscomponents] that are installed with the JDS. As far as Apache is concerned, we can make several observations:


<ul>
<li><span class="path">httpd.conf</span> and <span class="path">ssl.conf</span> are not modified</li>
<li><span class="path">conf.d/jds.conf</span> defines a new named-based virtual host on port 443</li>
<li>ServerName is set to the JDS name you entered during enrollment</li>
<li>The included <span class="path">apache_aliases.conf</span> defines several aliases (e.g. <span class="path">/CasperShare</span>, where packages are located)</li>
<li><span class="path">/jds/api.php</span> exists in the default DocumentRoot <span class="path">/var/www/html</span></li>
</ul>

<div class="post-heading1">Installing the NetSUS</div>

The NetSUS installer and source code is [available here][netsusgithub]. After installation, we see several changes to <span class="path">httpd.conf</span>:
<ul>
<li>Update DocumentRoot to <span class="path">/srv/SUS/html</span></li>
<li>Add an alias for <span class="path">/NetBoot</span></li>
<li>Add several rewrite rules for SUS</li>
</ul>


There are also changes to <span class="path">ssl.conf</span>:
<ul>
<li>Uncomment DocumentRoot <span class="path">var/www/html</span></li>
<li>Disable SSLv3</li>
<li>Modify logging</li>
</ul>

<div class="post-heading1">Conflicts</div>

Setting the global DocumentRoot to <span class="path">/srv/SUS/html</span> affects both the JDS and NetSUS. <span class="path">/jds/api.php</span> and <span class="path">/webadmin</span> are no longer accessible after the change. When you GET these resources, Apache directs your request to the virtual host defined in <span class="path">jds.conf</span>. DocumentRoot is not defined here, so it inherits the global value of <span class="path">/srv/SUS/html</span>. Since those two resources are in <span class="path">/var/www/html</span>, you get a 404 error.

<div class="post-heading1">Resolution</div>

There are many ways to remedy this. The simplest way is to define the DocumentRoot in <span class="path">jds.conf</span>, like so:

{% highlight text %}
{% raw %}
DocumentRoot /var/www/html
{% endraw %}
{% endhighlight %}

Future JDS upgrades might revert this modification, so a better method might be to add a separate config file that defines aliases to <span class="path">/webadmin</span> and <span class="path">/jds</span>:

{% highlight text %}

Alias /webadmin /var/www/html/webadmin
Alias /jds /var/www/html/jds

<Directory /var/www/html>
    Options None
    AllowOverride None
    Require all granted
</Directory>

{% endhighlight %}

Perhaps the best fix is to create a pull request for the [NetSUS project][netsusgithub]. There’s no real reason to change the global DocumentRoot in <span class="path">httpd.conf</span>. The same thing can be accomplished with a separate config file with a virtual host on port 80.

[jdscomponents]: https://jamfnation.jamfsoftware.com/article.html?id=339
[netsusgithub]: https://github.com/jamf/NetSUS