# Guide: Remediating Chef Client Deprecations in Chef Cookbooks

It's that time of year when the new [Chef Client](https://github.com/chef/chef), [ChefDK](https://github.com/chef/chef-dk), and [Chef Workstation](https://github.com/chef/chef-workstation) releases are hitting the internets. The Security and Compliance teams are knocking on your door as they've noticed new software has landed on your company-issued laptop. The Windows team is excited as there's no doubt new resources introduced to the core that will enable them to _finally_ start managing their platform.

But you, you're the linux/unix admin and you've been doing this for _years_. You have hundreds of cookbooks. Your cookbooks have dependencies. Their dependencies have dependencies. And now there's dreaded deprecations that have been introduced that need to be fixed up to upgrade to the latest kit.

As I see in most enterprises, the questions that need answering are:

- Why should we move to the latest kit?
- How do we know what deprecations we're effected by?
- Will fixing these deprecations break our current chef-client runs and pipeline build runners?
- How should I release this change in a controlled fashion?