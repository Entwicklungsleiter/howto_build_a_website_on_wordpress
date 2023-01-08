# DNS and server hosting

Note: This document is part of the documentation "[Howto build a website on wordpress](./README.md)".
You might want to start from the very beginning of it.

## Hosting and setup overview

There are hosting providers out there who do all the following things for You. Some readers might want to just book their services. If You're technically experienced and love "to have it in Your hand" ... go on reading here to:

- setup DNS
- setup a cloud webspace

## Setup DNS

For any DNS related topics I use provider [Schlundtech](https://cloud.schlundtech.com). The user interface is quite difficult and very often I need help to configure my DNS properly. However all You need here is a domain.

- **Define a domain for Your website.** Have in mind, that the name should tell the user what Your website will offer him. Also have in mind, that top level domains TLD (like ".com" or ".shop") have dramatically differnet costs. Check this too! Use a DNS provider website to check, if Your favourite domain is still available.
- **Register Yourself at the DNS provider.** Don't worry they want You to be a company ... there is nothing illegal in owning domains as a private person.
- **Configure payment options and register the domain.** It might take some time until You can configure Your webservers IP address.
- **Configure the domain A and AAAA record** to send http(s) traffic to Your new IP address (we'll get later). I recommend setting a short "TTL" ("time to live", the time it takes until the world fetches changes You make at DNS records) like 1 hour (3600s).
- **I also configured MX records** (email related stuff) to send and get emails for my new domain. This top is so complex it will be another book ... that's why I leave out the entire topic email here. It includes 
	- setup MX record
	- setup SPF record
	- setup DMARC and DKIM records,
	- whitelisting Your service at hundrets of blacklist pages and
	- many many test emails from all the email providers You know.
- I also got myself a Google account including the [Google search console](https://search.google.com/u/1/search-console) for later search engine optimizing of my website. For that one requires first proving Google to own the website ... this can be made through a DNS record called "google-site-verification". [Here's how this works.](https://support.google.com/webmasters/answer/9008080#domain_name_verification&zippy=%2Cdomainnamen-anbieter)

## Get a webserver

For any topics about webhosting / server / virtual machines I am a huge fan of [Hetzner](https://hetzner.cloud/?ref=FKwSTIu2wZ0L) (If You use this link You'll get 20€ coupon for free). It is easy to configure and runs stable. The costs are transparent and calculated daily. The documentation and support are really well.

- At [Hetzner](https://hetzner.cloud/?ref=FKwSTIu2wZ0L) I registered an account (that's free) and created a "project" ... what is just an option to group my services.
- WARNING: Before You book the following services, think about the location where Your website will be available later. Don't book a server in the USA if Your website users will be located in Europe. And make sure the webserver and disk volume are at the same Hetzner location (else the volume will not be visible for the server)!
- Inside the project I added a very small server "CX11" (quite small with just 2GB RAM, 20GB disc, 1CPU) and 
- additionally a volume (40GB).

Oh, good question! Thanks for asking why I have a separate volume!

All my services (docker containers, configurations, applications, database data) are stored on the 40GB volume and not in the servers 20GB disk. One day the servers operating system will be overaged and needs to be replaced. The separate data storage makes it very easy to configure a new server machine, mount the 40GB volume at the new server and I'll be up and running again without any data migration trouble. Minimal downtime.

- As operating system I select Ubuntu 22.04 in the Hetzner interface. Ubuntu seems to have the best documentation in the web (almost every Linux doc or question online is about Ubuntu), so that I feel optimal supported using this distribution. At the time of writing this documentation (2023/01) version 22.04 is the newest version with [long term support "LTS"](https://en.wikipedia.org/wiki/Ubuntu_version_history#Version_timeline). This delays my scenario (described above) about renewing the servers operating system into the future of 2027.
- At my local PC I [generated SSH keys](https://www.heise.de/tipps-tricks/SSH-Key-erstellen-so-geht-s-4400280.html) to enter my server through ssh later. The public key can be uploaded on the server configuration web interface (only the public key please).
- I did not book additional services like network, load balancing or firewalls in Hetzners web interface. After booking the machine it is available within seconds.

## Hosting finals

Now You might want to do these final steps before entering the server and do all the configurations:

- Through the Hetzner web interface the volume needs to be attached to the server: This is still not "mounting" the volume, but making the volume visible and mountable at the server. The Hetzner web interface also offers registering the volume into /etc/fstab of the server ... what I thankful used.
- In the servers overview page You now have the IPv4 and IPv6 adresses of the new server. Now You can go back to Your DNS hosting and add A record (IPv4) and AAAA (for IPv6) there.
