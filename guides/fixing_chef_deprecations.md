# Remediating Chef Client Deprecations in Chef Cookbooks

##Intro
It's that time of year when the new [Chef Client](https://github.com/chef/chef), [ChefDK](https://github.com/chef/chef-dk), and [Chef Workstation](https://github.com/chef/chef-workstation) releases are hitting the internets. The Security and Compliance teams are knocking on your door as they've noticed new software has landed on your company-issued laptop. The Windows team is excited as there's no doubt new resources introduced to the core that will enable them to _finally_ start managing their platform.

But you, you're the linux/unix admin and you've been doing this for _years_. You have hundreds of cookbooks. Your cookbooks have dependencies. Their dependencies have dependencies. And now there's dreaded deprecations that have been introduced that need to be fixed up to upgrade to the latest kit.

As I see in most enterprises, the questions that need answering are:

1. Why should we move to the latest kit?
2. How do we know what deprecations we're effected by?
3. Which deprecations do I fix first?
4. Will fixing these deprecations break our current chef-client runs and pipeline build runners?
5. How should I release this change in a controlled fashion?
6. How can I handle this in the future?

Let's get started...

## Prerequisites

1. A workstation of some sort
2. Local copy of the cookbooks that need evaluating
3. The latest version of [ChefDK](https://downloads.chef.io/chefdk) or [Chef Workstation](https://downloads.chef.io/chef-workstation)

## Q&A
### How do we know what deprecations we're effected by?

Good question - I'm going to make an assumption that you either have all of the cookbooks you need evaluated in a local directory already, or you can perform a `knife download /cookbooks` against your Chef Server to pull them down. Then perform the following in a shell:

```bash
$ cd /cookbooks
$ for cookbook in ./*; do echo "==> Cookbook: $cookbook" >> /tmp/all-cookbook-deprecations.txt && chef exec foodcritic -t deprecated "$cookbook" >> /tmp/all-cookbook-deprecations.txt; done
```
The above command will begin iterating over the cookbooks with foodcritic and outputting the deprecations to a text file in `/tmp/all-cookbook-deprecations.txt`.

In this text file you'll have a per-cookbook, line-by-line breakdown of the exact code in your cookbooks that will need changing.

### Which deprecations do I fix first?

So now you have your text file filled with the deprecations that need fixin'. I like to perform a frequency analysis to see just how effected my estate is. To do so:

```bash
$ grep -e "^FC*" /tmp/all-cookbook-deprecations.txt | cut -d " " -f 1,1 | sort | uniq -c
```

You'll now receive output that looks like this:

```
1 FC080: http://www.foodcritic.io/#FC080 User resource uses deprecated supports property
6 FC082: http://www.foodcritic.io/#FC082 node.set or node.set_unless used to set node attributes
81 FC083: http://www.foodcritic.io/#FC083 Execute resource using deprecated 'path' property
          This warning is shown if an execute resource includes the deprecated path property, which was removed in Chef 12. You should instead using the environment property to set the PATH variable.
2 FC084: http://www.foodcritic.io/#FC084 Deprecated Chef::REST class used
         This warning is shown if the deprecated Chef::REST class used in a cookbook. Chef::REST has been replaced by Chef::ServerAPI. See Chef Deprecation CHEF-9
41 FC113: http://www.foodcritic.io/#FC113 Resource declares deprecated use_inline_resources
2 FC116: http://www.foodcritic.io/#FC116 Cookbook depends on the deprecated compat_resource cookbook
```