--- 
layout: post
title:  "Welcome to Jekyll!"
date:   2014-01-15
title: "AWS VPC Buildout With Foreman Hooks for RDNS Creation"
permalink: "posts/foreman_hooks_aws_vpc"
---
Originally posted on [Digital Ducttape](http://digital-ducttape.com/2013/10/23/aws-vpc-buildout-with-foreman_hooks-for-rdns-creation/)

[Foreman](http://theforeman.org/) is a tool I have used for a long time as an external node classifier for Puppet and its smart-proxy for integration with DNS. However when Foreman recently [added support](http://projects.theforeman.org/issues/1871) for building EC2 instances inside a VPC, I thought it was a great opportunity to use the same tool for a new buildout.

Shortly after starting I discovered that reverse DNS is [unfortunately broken](http://projects.theforeman.org/issues/3166) for EC2 VPC builds. Foreman does not map a relationship between VPC subnets and any DNS smart-proxy. Unswayed I took to #foreman and was guided to a potential solution.
<!-- more -->

Enter [foreman_hooks](https://github.com/theforeman/foreman_hooks). This plugin for foreman allows you to subscribe to create/update/destroy events and invoke any executable. Lucky for me this was a great workaround for the functionality lacking in the recent 1.3 release. Initially I wrote a script utilizing the smart-proxy API to create PTR records but it complained that the IP address was already assigned. Fair enough. Without further investigation I wrote instead a short shell script which does the same thing using nsupdate with tsigs and viola! Integrated forward and reverse DNS for EC2 VPC deployments. A few more lines of code added support for the destroy hook and now the feature is complete.

I named this script rdns.sh and placed it in /usr/share/foreman/config/hooks.d and created symlinks to it in both:

/usr/share/foreman/config/hooks/host/managed/destroy/  
/usr/share/foreman/config/hooks/host/managed/create/

[source for rdns.sh](https://gist.github.com/ddoc/8447483)  
<script src="https://gist.github.com/ddoc/8447483.js"></script>
[source for utils.sh](https://gist.github.com/ddoc/8446722)  
[source for hook_functions.sh](https://github.com/theforeman/foreman_hooks/blob/master/examples/hook_functions.sh)

Of course, after implementing this I saw many uses for foreman_hooks including:

* resizing an EC2 root volume at build time
* attaching additional volumes
* EIP associations and DNS
* different hooks for different domains
* adding and removing hosts to an external monitoring system

I will post updates on these implementations at another time.
