---
title:  "Host your own npm packages in Nexus"
date:   2015-07-07 18:37:00
categories: nexus
---

As you may notice this is the first post of hopefully many to come on this freshly established blog. 

Without further introduction, let's get things rolling by setting the scene. I've recently joined the ranks of Sonatype, a company that helps understand and manage Software Supply Chains. I was excited to do so, as I've been using nexus for quite some time.

In my role of a Solutions Architect, I'm constantly meeting with new clients to illustrate how Nexus can solve their problems. During these meetings I often encounter very insightful questions about features and functionality. I thought it beneficial to keep track of these questions on this site, and to answer them in more detail.

## Nexus & NPM

I'd like to kickstart this with a simple, and hopefully useful little tutorial about how to start hosting your npm packages with the Nexus repository. 

I can already imagine the reaction:

> "Nah brah, I've got heroku/S3/git"
>
> -*every artisanal code crafter ever*

Fair, if you just want to push, test and deliver your singleton application, this is a good approach. 

However, if you do anything more advanced, say a set of immutable microservices with interdependent and coupled APIs, there are a few advantages to using an Artifact repository. 

First of all, I would like to point out having a artifact repository is now a widely-acknowledged part of a normal software supply chain [Devops Reference Architectures #1](www.slideshare.net/SonatypeCorp/nexus-and-continuous-delivery) [Devops Reference architectures #2](www.slideshare.net/SonatypeCorp/devops-and-continuous-delivery-reference-architectures).

A key aspect of artifact repositories is to proxy, or cache, components you download from 3rd party sources, like npmjs.org. Instead of pointing your package.json directly at npmjs as is standard behaviour, you can configure npm to use a private repository instead. If the repository has a local copy of the package, it can be fetched directly from this local repository. If it does not exist, the artifact repository will automatically ingest new components from npmjs.org. 

Why go to such lengths and add complexity? I present to you ***#npmgate***: [The Register: How one developer just broke Node, Babel and thousands of projects in 11 lines of JavaScript](www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/) . This kind of situation, where builds break when components are removed can be avoided in its entirety by having an artifact repository in place. Additional benefits include consistent builds, the ability to catalog what is being downloaded across many teams, ***the ability to health check packages*** among of course being able to also host and distribute your own packages.

Most companies and teams are also not an exclusively single-language environment either. Nexus 3 can help you also host your bower packages, or packages from other languages, consolidating all your access points to a single base url. Pretty handy!

## Enough talk, let's do this!

Ok, there's basically three things we can do: a hosted repository and a proxied repository. The proxy repository is used to make sure the components you download are cached in a repository that you control, and the hosted repository will be used to publish your own components. The third type is a group repository that we'll explore at the very end.

### Overall architecture

The two come together to form an image like this:
![My helpful screenshot](/assets/images/proxy-repos.png)

**Step 0: Install Nexus 3**

You can grab the latest copy of Nexus 3 here:
[Download Nexus 3](http://www.sonatype.com/get-nexus-sonatype)

**Step 1: Setting up a proxy repository**

Proxy repositories are used to cache components you download from NPM. Handy when you want to retain copies and be independent from any connectivity issues or sure of consistency of the downloaded artifacts. What happens under the hood is that the repository attempts to serve local copies of artifacts. If the artifact is not found locally, the server then goes to the remote repository to fetch the upstream copy, caching files in the process for future usage.

The first step in this is is to create a new proxy repository. This can be done in the repositories view by clicking new and choosing npm proxy repository.

![My helpful screenshot](/assets/images/proxy-2.png)

There are a few setting that appear at the bottom of the page. You can fill in the following:
 
* ***Repository name***: a friendly name for the repo. This name is referenced in the UI and the Logs.
* ***Blobstore***: Just choose the Default for now. In future releases you'll be able to configure different locations to store this repository.

Open the remote repository access dropdown and fill in the following:

 * ***Remote Storage Location***: https://registry.npmjs.org - This is the uri of the proxied npm repository.

That's it! Click save.

Your proxy repository is now in action! You can now configure npm to start downloading packages from your local repo instead.

**Step 2: Configure npm to use the proxied repo**

To start proxying components we need to configure npm to use the new repository.

find the *~/.npmrc* file in your local file system. If it does not exist, go ahead and create it. Add the following line:

{% highlight bash %}

registry=http://NEXUS-URI/nexus/content/repositories/REPOSITORY-NAME/ #You get this from the Nexus UI

{% endhighlight %}

You can also find the full repository URL on the nexus UI, on the right end of the repository listing.

![My helpful screenshot](/assets/images/url-1.png)

**Step 3: Try proxying out!**

Now you should be configured! Try it out by going in to any folder and running the following:

{% highlight bash %}

mkdir demo && cd demo
npm install bower

{% endhighlight %}

Everything should install smoothly and found under ./node_modules. If you go to Nexus and use the Search view to observe your proxy repository, you'll see a full rendering of the repository that should now have all the files you downloaded.

**Step 4: Create a hosted NPM Repository**

Ok, we've downloaded some dependencies. Let's now create a hosted repository to host whatever it is you'll end up building. Creating the hosted repository is much the same as last time - go to the repositories view in Nexus, and add a new ***npm (hosted)*** repository. Again, fill in the details, giving the repository a memorable name. Click save. That's it, you're now ready to publish your own components.

![My helpful screenshot](/assets/images/hosted-1.png)


**Step 5: Publish your package**

Publishing is easy, you can either just add your target URI directly to your npm publish command, or you can edit your package.json [as per the Nexus 3 documentation](https://books.sonatype.com/nexus-book/3.0/reference/npm.html#npm-private-registries)

Direct parameter approach example:

{% highlight bash %}

npm publish --registry http://NEXUS-URI/repository/npm-hosted-repo-id/

{% endhighlight %}

OR editing the package.json:

{% highlight bash %}
 
  "publishConfig" : {
    "registry" : "http://localhost:8081/repository/npm-internal/"
  },

{% endhighlight %}

**Aaaand you're done**

Wicked work my friend! You're done and your new npm repo is in action. You can now host and download components from the same URI. You can also use all the latest tips and trics in npms books, like namespacing.

**Extra Credit: Group both repositories to a single URI**

The final trick in the basic repo management book is a handy one. Normally, to access the packages hosted in your hosted registry you'd have to change the configuration of the .npmrc registry value like we did in Step 2 to point to the hosted repository or specify locations for each package in your package.json. This obviously is a lot of hassle and work to maintain, so Nexus includes a handy way of grouping both the hosted and proxy repository together via a **Group Repository**. This group repo will allow you to download contents of all repositories belonging in said group with accessing only one URL. Pretty cool, eh?

To do this, we'll create a new repo as before, but this time choose ***npm (proxy)***. Again, create a memorable name and id, like npm-group. The big step here is deciding which repositories should be included in your group - this time let's just all bunch them together. In real life situations and larger teams you might consider what groups to establish based on access control etc.

![My helpful screenshot](/assets/images/group-1.png)

Now that the group is created, we only need to update our ***.npmrc*** to point to the group - as we did before. Fetch the URL from the right hand side, and add it to the file... and you're done. After this you can now download packages from all repositories in the group by default. Pretty wicked, eh?

That's it for this post. Happy developing!
