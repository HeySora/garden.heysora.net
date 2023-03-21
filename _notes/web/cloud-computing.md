---
title: "Cloud Computing: An extensive guide"
short-title: Cloud Computing
subtitle: Why should you stop having everything in your single VPS from OVH and calling it a day
tags: web
---

#### Alternate subtitle: How computers, despite being nasty creatures working against humanity (sometimes), can be really pleasant to work with

This note is intended to explain Clouds from scratch. If you don't have much experience with those, you're at the right place!

---

## What are clouds, anyway? I'm happy having my single machine

Today’s definition of cloud basically started thanks to Amazon, with their *Amazon Web Services*, or AWS for short. They’ve been around since almost 20 years already.

Maintaining a bare-metal machine yourself is a tedious task: you need to monitor  potential hardware failures, you have to dust it out, electricity bills can be quite expensive, and you likely won't have a network suiting your needs, since being able to serve websites to many users (and eventually downloads) requires a TON of network capacity.  
Similarly, if you're renting a dedicated server somewhere, it will still be prone to hardware failure such as failing drives or motherboard, or electricity/network outages within a rack. Available bandwidth also is quite commonly unattractive (often limited to 250-500Mbps) when renting such machines. Also, for simple tasks such as storing data or running some simple code, maintaining a whole machine doesn’t really seem worth it. Then you have all of the other goodness like Load Balancers...

That’s what cloud services are there for! 😎  
You don’t have to think about the specific hardware, about any external failure (most of the time), about network equipment, about disk replication… and the list goes on.  
You typically pay for what you actually consume: Virtual machines are typically billed per-hour, and storage solutions will bill you depending on the actual space and bandwidth you're consuming.
Another important aspect of cloud services is the fact that literally everything can be controlled via your terminal or via scripts, making everything automatable and monitorable.

As alternatives to Amazon’s AWS, the big ones are Google’s GCP, and Microsoft’s Azure. DigitalOcean also has a bunch of cloud offers, and so does CloudFlare. In place of proprietary clouds like this, a really great alternative is [OpenStack](https://www.openstack.org) (which is used by [many providers](https://www.openstack.org/marketplace/public-clouds), I've had lots of fun using it).

## So are clouds just VMs? What kind of services are there?

Amazon, Google and Microsoft all have more than 200 different kinds of services. 😵‍💫  
Aside from Virtual machines, there are many other kinds of services, for example specialized in networking, database management, machine learning, E-mail/SMS/Push delivery, etc.

It would be crazy to detail all of these (and someone claiming to know everything would be a pure liar 😅) so I’m going to focus on three big ones that you’re going to find almost everywhere.

### Virtual machines

This is exactly what it sounds like! A Virtual Machine is nothing but some CPU/RAM/Disk that you have access to, running some OS (mostly Linux).

As explained above, VMs allow you to focus on your system without having to think about hardware maintenance, or outages outside of your reach. VMs also have the ability to be deployed by automatically running a custom install script, or even from a pre-made *image*.  
This might look unimpressive since this is common for dedicated machines, but keep in mind that everything can be automated with your CLI.

VMs, in clouds, also have the ability to be in several networks. Just make as many networks as you want, with whatever IP ranges you'd like, and attach NICs to each of these networks. Everything will just work as you'd expect, similarly to doing this yourself with network switches and routers.  
And of course, a VM can also be *resized* (increase/decrease its hardware), cloned in minutes, etc.

Splitting up a machine into several VMs can be useful for a lot of reasons:

First, splitting things up and delegating tasks to other servers has a lot of advantages like exercising [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns), being able to independently take down and maintain independant elements of your infrastructure, and being able to deploy multiple machines doing the same thing (behind a *Load Balancer*)... All of that leads to making things easier to service for the long-term.

Second, this could allow having separate environments for production and testing; the former being what’s deemed stable and accessible by everyone, and the latter being used for new features or enhancements. While this is not the most attractive argument if you're building a personal website, or basically any project where you're the only developer, it's still something important to think about when working in a team.  
It's easy to have things being set up differently between your local environment and a proper [staging environment](https://en.wikipedia.org/wiki/Deployment_environment). Making use of a standalone server with an identical configuration to the production machine makes potential interruptions of service (due to a faulty code change or deployment) catchable early, before interrupting your service.

Furthermore, a testing environment can be freely reinstalled if things break really badly (faulty operating system update, or accidental `rm -rf`).  
Finally, all of this can be efficiently managed with CI/CD systems. Deploying on separate environments shouldn't be a pain and while it is for sure a lot of effort for the short-term, it's an investment that is really worth it for the future, as long as you've done things properly.

*If you wish to know more about this online, this service is known as these names:*

- *“OpenStack Nova”*
- *“AWS EC2”*
- *“Google Compute Engine”*
- *“Azure Virtual Machines”*
- *“DigitalOcean Droplets”*

### Object Storage

This service can be thought of as a big drive that is easy to interface with, especially from an application or a website.  
This shouldn't be seen like a raw partition (this would be *Block Storage*, which is separate). While you can actually mount an Object Storage endpoint as a partition with [s3fs](https://github.com/s3fs-fuse/s3fs-fuse), you should see this service similarly to a Google Drive or Dropbox account, except that it has very powerful APIs.

One of its main uses is to store big files, as they’re the ones which generate the biggest amount of data transfer, and thus are more likely to slow down your VMs and make everything sluggish for the end-users. They're also convenient for generated data (like daily reports).  

Of course, they also offer the advantage of not having to maintain a webserver for this purpose.  
You don't have to tune nginx anymore to enable `aio threads`, `directio`, and mess with `sendfile` to then realize that PHP-FPM broke in the way and made half of your website unresponsive, only for ending up tuning `sysctl` values in a random fashion and hoping for the best. It's okay, we've all been there. 🙂

The industry standard for Object Storage is “AWS S3”, which is a really, *really* big technology Amazon brought into this world. You will hear of many Object Storage providers as being “S3-compatible” or “S3-like”, meaning that any software already supporting AWS S3’s API could support their own without any code change, despite the underlying system being different.

<div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 8px;">
    <img src="/assets/cloud-computing/s3-cloudflare.png" alt="Cloudflare qualifying their object storage as being &quot;S3-compatible&quot;" class="tone-down"
    style="margin: 0px; flex-grow :1; width: 260px;" />
    <img src="/assets/cloud-computing/s3-digitalocean.png" alt="DigitalOcean qualifying their object storage as being &quot;S3-compatible&quot;" class="tone-down"
    style="margin: 0px; flex-grow :1; width: 260px;" />
</div>

This means that DigitalOcean storage could be operated like a real S3 storage from an app’s point of view, without needing a “DigitalOcean” plugin. Just use any library supporting S3, just swap Amazon’s URL endpoint to DO’s one, and you’re good to go!  
For some quick use-case context, Discord uses GCP for all file attachments.

*If you wish to know more about this online, this service is known as these names:*

- *“OpenStack Swift”*
- *“AWS S3”*
- *“Google Cloud Storage”*
- *“Azure Cloud Storage”*
- *“DigitalOcean Spaces”*
- *“Cloudflare R2”*

### Workers / Serverless code

Imagine being able to write some concise code that is good at doing a (few) specific thing(s). Now imagine being able to *magically* deploy it over the whole world, without needing to setup a single server, nor even deal with replication. Without having to pay for a proper machine, without having to deal with updates and stuff, and while being always close to the requester… Sounds utopian, kind of.

This is called “Workers”, "Lambdas", or “Serverless architecture”. Of course servers are still being involved behind the scenes, but you have no control/access over these simply because you don’t need to. Your code will run everywhere, and quickly, period.

This is exactly what [fxtwitter.com](https://fxtwitter.com) is, for example! This one is actually running on Cloudflare's Serverless infrastructure. Its role is to improve Twitter’s embeds on services such as Discord and Telegram. Its role is quite concise:
- First, a request is made, with the usual Twitter URL scheme
    (e.g. `https://fxtwitter.com/HeeySora/status/1495046593971638280`)
- The code will then act differently whether it’s from a genuine browser (e.g. if you were to click the link) or a fetching bot (Telegram/Discord generating the link preview)
    If you clicked the link directly, the code will simply redirect you to Twitter and bail out
- For bots, the code will fetch the actual tweet, and apply its transformations to make the twitter media actually readable (whether it is multi-pictures, videos, polls, etc.)
- Finally, the response is crafted in a special way for these bots, and returned, so Discord embeds and Telegram previews are all shiny. ✨

Obviously, there are other things going on (for example, caching is done in order to not do the job twice for a single tweet; error handling is being set-up, etc.), but this is the gist of it. It's made in mind to do specific things, and it does it well. Besides for the caching, this worker doesn’t even use any kind of persistent storage or filesystem access. That’s the philosophy of workers.
*(By the way, it is [open-source](https://github.com/FixTweet/FixTweet), if you're curious about how it works!)*

Another use-case for serverless code could be asset generation (e.g. for an e-commerce website, generating PDF invoices or coupons from input data) or serving cached API data (caching it on the worker’s side rather than having unnecessary load on the origin server). All of those examples are easy ways to remove a lot of load off your other machines!

*If you wish to know more about this online, this service is known as these names:*

- *“OpenStack Qinling”*
- *“AWS Lambda”*
- *“Google Cloud Functions”*
- *“Azure Serverless”*
- *“DigitalOcean Functions”*
- *“Cloudflare Workers”*

## Conclusion

Hopefully this note made the obscure concept of cloud computing much clearer. While it might not be your cup of tea, it's always useful to know about this kind of things, in case you'll be working on such an environment.

It's fun for me to write about this, and all of this served me well: when [NotITG was at AGDQ2022](https://youtu.be/sBP8MxQhEVM?t=14), being able to support the load was going to be a serious challenge.

In two hours, over 50K game downloads had been served (which was over 30TB of bandwidth, representing an average of ~33Gbps during these two hours!), and another 30K downloads in the next 24 hours. Everything went flawlessly, and was being managed by several VMs located on different datacenters and being located behind a doubly-replicated Load Balancer, all of this running on OpenStack.  
Evidently, as you might expect, OpenStack is [open-source too](https://opendev.org/openstack).

!["Always has been" meme template, with one of the astronauts saying "Wait, it's all open-source?"](/assets/cloud-computing/open-source.png){: .tone-down}

Every small effort matters, so please also make sure to [[web-optimisations|decently optimize your websites]], and properly bundle/minify your assets if it's not yet the case!