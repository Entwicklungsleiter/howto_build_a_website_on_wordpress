# DNS and server hosting

Note: This document is part of the documentation "[Howto build a website on wordpress](../README.md)".
You might want to start from the very beginning of it.

## Hosting and setup overview

For my website I required a domain and a webserver with public IP addresses. Regarding Wordpress hosting, there are providers out there who do all the hosting things for You. Some readers might want to just book their services and skip all the "technical" chapters and jump into [Wordpress work topics](./04_wordpress_basic_configuration.md). If You ... like me ... love "to have it in Your hand" ... go on reading here for:

- setup DNS and
- setup a cloud webspace

and in the next chapters

- [configure the server](./02_server_configuration.md) and
- [configure the webserver and install Wordpress](./03_webserver_configuration.md).

## Setup DNS

For any DNS related topics I use provider [Schlundtech](https://cloud.schlundtech.com). The user interface is somehow difficult and very often I needed help to configure my DNS properly. However all I need here is a domain.

- **Define a domain for the website.** I have in mind, that the name will tell the user what my website will offer. My domain should NOT express my inner feelings, but tell someone else very short, what to get here. I also have in mind, that top level domains TLD (like ".com" or ".shop") have dramatically differnet costs. I use the DNS provider website to check, if my favourite domain is still available and how much annual costs it will be.
- **Register at the DNS provider.** The register form wants me to be a company ... but there is nothing illegal in owning domains as a private person.
- **Configure payment options and register the domain.** After that it takes 2 days until I "have" the domain and can configure the IP addresses the domain will point to. But I still have no server and no IP ... this will be in next chapter.
- **(as soon as I have the domain:) Configure the domain A and AAAA record** to send http(s) traffic to my new IPv4 (A record) and IPv6 (AAAA record) address (we'll get later). I set a short "TTL" ("time to live", the time it takes until the world fetches changes I make at DNS records) to something like 1 hour (3600s).
- **I also configured MX records** (email related stuff) to send and get emails for my new domain. This top is so complex, it will be another book ... that's why I leave out the entire topic email here. It includes 
	- setup MX record
	- setup SPF record
	- setup DMARC and DKIM records,
	- whitelisting my IP at many blacklist pages and
	- many many test emails from all the email providers I know.
- I also got myself a Google account including the [Google search console](https://search.google.com/u/1/search-console) for later search engine optimizing of my website. For that one requires first proving Google that I own the domain ... this can be made through a txt DNS record called "google-site-verification". [Here's how this works.](https://support.google.com/webmasters/answer/9008080#domain_name_verification&zippy=%2Cdomainnamen-anbieter)

## Get a webserver

For any topics about webhosting / server / virtual machines I am a huge fan of [Hetzner](https://hetzner.cloud/?ref=FKwSTIu2wZ0L) (If You use this link You'll get 20€ coupon for free). It is easy to configure and runs stable. The costs are transparent and calculated daily. The user interface and support are really well.

- At [Hetzner](https://hetzner.cloud/?ref=FKwSTIu2wZ0L) I register an account (that's free) and create a "project" ... what is just an option to group my services.
- WARNING: Before booking the following services, I have to think about the location, where the website will be available later. One should not book a server / machine in the United States, if the website audience will be located in Europe. This would make the website slow for the audience.
- Inside the project I add a very small server "CX11" (quite small with just 2GB RAM, 20GB disc, 1CPU) and 
- additionally a volume (40GB). Here I make sure the webserver and disk volume are at the same Hetzner location (else the volume will not be visible for the server)!

**Oh, good question! Thanks for asking why I have a separate volume!**

All my services (docker containers, configurations, applications, database data) are not stored on the servers 20GB disk, but on the 40GB volume. One day the servers operating system will be overaged and needs to be replaced. The separate data storage makes it very easy to (one day) configure a new server machine, re-mount the 40GB volume (with all my data and services) at the new server and I'll be up and running again without any data migration trouble. That's why I split the server machine operating system and my services hard disk from the very beginning.

- As operating system I select Ubuntu 22.04 in the Hetzner interface. Ubuntu seems to have the biggest documentation in the web (almost every Linux doc or question online is about Ubuntu), so that I feel optimal supported using this distribution. At the time of writing this documentation (2023) version 22.04 is the newest version with [long term support "LTS"](https://en.wikipedia.org/wiki/Ubuntu_version_history#Version_timeline). This delays my scenario (described above) about renewing the servers operating system into the future of 2027.
- At my local PC I [generated SSH keys](https://www.heise.de/tipps-tricks/SSH-Key-erstellen-so-geht-s-4400280.html) to enter my server through ssh later. The public key I upload on the server configuration web interface (only the public key please).
- I do (for now) not book additional services like network, load balancing or firewalls in Hetzners web interface. After booking the machine it is available within seconds.

## Hosting finals

Now I do these final steps before entering the server and do all the configurations:

- Through the Hetzner web interface the volume needs to be "attached" to the server: This is still not "mounting" the volume, but making the volume visible and mountable at the server (like plugin an USB stick). Additionally the Hetzner web interface also offers registering the volume into /etc/fstab of the server ... what I thankful use. My path for mounting the volume is /mnt/volume-\<volume-name\>.
- After rebooting the Hetzner virtual machine the volume will be mounted. This can be checked with 
```shell
# show all storage partitions mounted and their size in human readable format
df -h
```
- In the Hetzner servers overview page I find the IPv4 and IPv6 adresses of the new server. So I can go back to DNS hosting and add A record (IPv4) and AAAA (for IPv6) there.

Well, now we have an URL / domain and a server. See [next chapter](./02_server_configuration.md) how I configured the server to be a server, docker host and available via ssh.