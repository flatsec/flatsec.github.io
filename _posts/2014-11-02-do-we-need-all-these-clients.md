---
layout:     post
title:      Do we really need all these clients running?
date:       2014-11-02 14:00:00
categories:
  - devops

---

Part of DevOps is tools. Not all of it, but some. Tools like Puppet, Chef, Sensu, Logstash...tools that provide valuable services in terms of configuration management, monitoring, and logging and metrics. But many of them require running a full-fledged client on the server. These clients take up resources, require maintenance, and increase the attack surface. Do we really need them?

<!-- more -->

### Overuse of client based solutions

What I would like to discuss in this post is what I think is an overuse of client based solutions, ie. clients running on a server providing some part of a system, in order to enable typically DevOps associated tools and solutions. Such as:

1. Configuration management
2. Monitoring ("if it's not monitored it's not in production")
3. Metrics gathering

All three of these components typically require clients to run on the server. So if we have an N-tier system of of some kind, we end up with these clients running on every server, on every tier.

### Web application example

Let's take a simple multi-tier web application as an example. So we have the below tiers:

1. Load balancer (Haproxy)
2. Cache servers (Varnish)
3. Application servers (Apache)
4. Database server (MySQL)

In what is getting to be a pretty standard DevOpsy setup, every server will run:

1. A Chef or Puppet client (for config management)
2. Sensu monitoring client (for monitoring)
3. Logstash client (for metrics and logging)

If we're running in some kind of high availability mode, ie. two servers for each tier, then we have eight servers, each running these clients, so 8*3=24 clients, plus the applications required to actually provide the server. This seems too heavy for me, and while I haven't yet looked at the memory and CPU usage for all of these clients (fodder for a future post), I have a feeling that they will, depending on load, outweigh the actual systems running.

### Other concerns

* Least amount of code, attack surface size

Not only are these clients taking up memory and CPU resources, but they also increase the attack surface of each system. Each of these clients is made up of thousands of lines of code, and, if there is any guarantee in life, it's that more code means more bugs, and more bugs means more security issues.

* Often run as root

Many of these clients expect to be run as root. For example, by default the Chef client runs as root. Haproxy, Varnish, Apache, MySQL--these systems can all run as their own user and yet the Chef client is running as root. These systems are mature and drop privileges where they can. Presumably we're running each main service on a separate virtual machine in part for security reasons, and yet here is Chef or Puppet running as root on every node.

* May be listening on the network

Some of these clients may have an open port. Most connect back to their server in a pull based model, but some will be listening on the network.

* SSL

Chef and Puppet clients typically use SSL for authentication. So does Sensu. Given the recent security issues with OpenSSL (heartbleed, poodle, etc) we know that there is a lot of work that needs to be done with OpenSSL before we can go back to trusting it. Eventually OpenSSL will be more trustworthy, especially given the Internet community at-large realized that we need to fund its maintenance, but for the time being maybe it's something to be avoided, if possible. Even Netflix has recently discussed their concerns with SSL and how they have implemented their own [secure messaging system](http://techblog.netflix.com/2014/10/message-security-layer-modern-take-on.html).

* More work

If we are automating this system then each of these clients will have to have their installation and configuration automated. That is extra work that may not be necessary. Certainly once the clients are automated there isn't all that much more work to do, but at least initially there is some Chef or Puppet code that needs to be written.

Further, if there are updates (security or otherwise) to these clients they will also need to be maintained.

### This isn't going to work in containers

I do think containers are important. They aren't going to be used by all organizations to solve all problems, but I do think that they provide useful ideas and workflows, and certainly point out issues in typical systems deployment methodologies.

If we deployed this example web application using containers we would not want to run the additional services in each container. Using Docker as an example of container technology, it quickly becomes evident that it's preferable to run as few services in each container as possible, even going so far as to run only one main service (_if_ we can get away with it). I don't think it would make sense to run Varnish plus a Chef client, plus a Sensu client, plus a Logstash client in each individual container.

Also, I should mention that some containers don't even run ssh.

### Potential solutions

* Agent-less configuration management

Of the four major configuration management systems, Puppet, Chef, Ansible, and SaltStack, Ansible is the only one that is typically deployed without agents. I am a big fan of Ansible, so perhaps I am biased here, but I certainly like the fact that Ansible is agent-less and uses ssh as its transport mechanism.

The other three all support agentless workflows in some fashion (eg. [Puppet](http://bitfieldconsulting.com/scaling-puppet-with-distributed-version-control)) usually by pulling code from a repository and running the agent from cron.

* Virtual machine introspection

This is less common but is actually recommended by NIST in a hypervisor environment. Essentially the idea is that the hypervisors can provide an API that can allow a security system (or any system, really) to peer into the virtual machines. Through this API it's conceivable that monitoring, logging, and even configuration management systems could be run. Certainly this could have security implications as well.

(I believe, for example, Rackspace runs a client on each of their virtual machines that provides a similar concept, though I can't find a link right now.)

* Service discovery and configuration

I don't really know what to call this yet, but basically I mean systems like Consul, which is a datacenter technology that also includes [health checks](http://www.consul.io/intro/getting-started/checks.html), ie. monitoring, among other features.

Jeff Lindsay writes about this on his [blog](http://progrium.com/blog/2014/08/20/consul-service-discovery-with-docker/):

>Specialized health checks are exactly what monitoring systems give us, and Consul gives us a distributed monitoring system. Then it lets us choose if we want to want to associate a check with a service, while also supporting the simpler TTL heartbeat model as an alternative. Either way, if a service is detected as not healthy, it's hidden from queries for active services.

Consul may require running a client on the node, and if so might just be exchanging one client for another, though my feeling is that Consul is meant to be a lightweight system that performs multiple functions--configuration management, leader election, monitoring, and service discovery. When looking at new technology like Consul it becomes evident that these processes are all much more closely related than, we, as in IT workers, are willing to admit. If monitoring, logging, metrics, and configuration managmeent are needed for every system, then perhaps we are going about it the wrong way.

* Use existing systems

To be honest I'm not quite sure what a logstash client provides. Why not just use rsyslog to send log messages to a central server? Presumably that central log server can then import logs into whatever system you have there. Or perhaps in a container environment the system managing the containers takes care of centralizing logs and monitoring services.

### Conclusion

This post isn't meant to provide any answers. It's really about exploring some ideas I've been thinking about lately, especially around complexity, and also, frankly, I'm annoyed with having to install and maintain all these clients (that's the grumpy sysadmin side of me). Plus it's a lot of code, sometimes running as root.

In future posts I'd like to explore:

* How much code is actually in these clients?
* What kind of memory footprint do they have?
* What do attack vectors look like for these clients?
* What are other alternatives?
* What does virtual machine introspection look like?
* How can we avoid SSL but still pass messages securely?
* Do technologies like Consul wrap all these requirements up into one nice, relatively simple multi-datacenter ready system?
* Will container management systems such as Docker eventually take care of things like monitoring and logging?
