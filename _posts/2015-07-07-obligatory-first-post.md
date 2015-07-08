---
title:  "Host your own npm packages in Nexus"
date:   2015-07-07 18:37:00
categories: nexus
---

**Hello there**

As you may notice this is the first post of hopefully many to come on this freshly established blog. 

Without further introduction, let's get things rolling by setting the scene. I've recently joined the ranks of Sonatype, a company that helps understand and manage Software Supply Chains. I was excited to do so, as I've been using nexus for quite some time.

In my role of a Solutions Architect, I'm constantly meeting with new clients to illustrate how Nexus can solve their problems. During these meetings I often encounter very insightful questions about features and functionality. I thought it beneficial to keep track of these questions on this site, and to answer them in more detail.

**Nexus & NPM**

I'd like to kickstart this with a simple, and hopefully useful little tutorial about how to start hosting your npm packages with the Nexus repository. 

I can already imagine the reaction:

> "Nah brah, I've got heroku"
>
> -*every artisanal code crafter ever*

Fair, if you just want to push, test and deliver your singleton application, this is a good approach. 

However, if you do anything more advanced, say a set of immutable microservices with interdependent and coupled APIs, there are a few advantages to using an Artifact repository. ***Read my post about this here.***

***Consistent deployments***

One of the issues in relying on services outside your own controls, such as npm, is that the service providers are in no obligation to either guarantee the uptime of the service ( they're usually run entirely on good will ) or the consistency of the served packages ( if the author decides after two days of publishing a package to add a small patch but not increment the version number. You catch my drift ). 

Hosting all of the packages, proprietary and public, can help alleviate this issue. By retaining a copy of every package in an artifact repository

***Consistent testing***

A cool side effect of being able to do consistent deployments is that you can now run a on-demand copy of your entire service and expose it to automated tests, not only on single components, but also on the holistic system scale. This'll help ensure your services talk to one another without hassle.

***Enterprise restrictions***

From an enteprise point of view it is rarely desireable to allow any more connections to the outside world than is absolutely necessary. Sometimes complex firewalling strategies can even stop this altogether.

Also sometimes things like security & standards compliance can put restrictions on these connections.

***Audit***

A curse word for many a developer. Sometimes due to compliance it is necessary to retain or limit the packages used in your system.


***Knowledge sharing***

A positive in hosting packages is that if you have multiple developer teams, you can share patterns and libraries that have been found to work well across other teams. By maintaining a standard toolset and actively encouraging the usage of it, we can ensure that developers understand what happens in other teams' products with much less effort.

**Enough talk, let's do this!**

Ok, there's basically two things we can do: a hosted repository and a proxied repository. The proxy repository is used to make sure the components you download are consistent and the hosted repository will be used to publish your own components.

*Overall architecture*

The two come together to form an image like this:
![My helpful screenshot](/assets/images/proxy-repos.png)

*1. Setting up a proxy repository*

Proxy repositories are used to cache components you download from NPM. Handy when you want to retain copies and be independent from any connectivity issues or sure of consistency of the downloaded artifacts.

Choose https://registry.npmjs.org



Neque porro *quisquam* est, qui **dolorem** ipsum, quia ***dolor*** sit, amet, [consectetur](http://cjdns.info/), adipisci velit.

 * lorem
 * ipsum

1. dolor
2. sit

| First Header | Second Header |
|--------------|---------------|
| Table Cell   | Table Cell    |

**Blockquote**

> They who can give up essential liberty to obtain a little temporary safety, deserve neither liberty nor safety.
> 
> _Benjamin Franklin_

**Code**

{% highlight c %}

static void asyncEnabled(Dict* args, void* vAdmin, String* txid, struct Allocator* requestAlloc)
{
    struct Admin* admin = Identity_check((struct Admin*) vAdmin);
    int64_t enabled = admin->asyncEnabled;
    Dict d = Dict_CONST(String_CONST("asyncEnabled"), Int_OBJ(enabled), NULL);
    Admin_sendMessage(&d, txid, admin);
}

{% endhighlight %}