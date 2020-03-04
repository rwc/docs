# Remediating Chef Client Deprecations in Chef Cookbooks

## Intro

It's that time of year when the new [Chef Client](https://github.com/chef/chef), [ChefDK](https://github.com/chef/chef-dk), and [Chef Workstation](https://github.com/chef/chef-workstation) releases are hitting the internet. The Security and Compliance teams are knocking on your door as they've noticed new software has landed on your company-issued laptop. The Windows team is excited as there's no doubt new resources introduced to the core that will enable them to _finally_ start managing their platform.

But you, you're the linux/unix admin and you've been doing this for _years_. You have hundreds of cookbooks. Your cookbooks have dependencies. Their dependencies have dependencies. And now there's dreaded deprecations that have been introduced that need to be fixed up to upgrade to the latest kit.

As I see in most enterprises, the questions that need answering are:

1. Why should we move to the latest kit?
2. How do we know which deprecations we're affected by?
3. Which deprecations do I fix first?
4. Will fixing these deprecations break our current chef-client runs and pipeline build runners?
5. How should I release this change in a controlled fashion?
6. How can I handle this in the future?

Let's get started...

## Prerequisites

1. A workstation of some sort
2. Local copy of the cookbooks that need evaluating
3. The latest version of [Chef Workstation](https://downloads.chef.io/chef-workstation) for your OS.
4. (Optional) [Docker Desktop](https://www.docker.com/products/docker-desktop) for your workstation

## Q&A
### How do we know which deprecations we're affected by?

Great question! In the past you needed to download multiple versions of ChefDK that mapped to the Chef Infra Client version you were going from, to the version you were going to. Then you'd run foodcritic against each cookbook and start traipsing through the output.

With the introduction of [Chef Cookstyle](https://github.com/chef/cookstyle), that's not a thing anymore! We can simply set a target version (in this example we'll be going from 12 to 15) and get rolling! Please check out the [Chef Cookstyle documentation](https://docs.chef.io/cookstyle/) for further questions!

Ok, now what?

At this point you need to have the cookbooks to be evaluated in a local directory, or you can perform a `knife download /cookbooks` against your Chef Server to pull all of them down. You also need the latest version of Chef Workstation installed on the host or running in Docker Desktop.

Using Docker to launch ChefDK to your local cookbook directory:
```bash
$ docker run -v ~/local/path/to/cookbooks:/tmp -it chef/chefworkstation
```

Now perform the following in the docker container or your local shell:
```bash
$ chef env --chef-license accept
$ cd ~/path/to/your/cookbooks (/tmp if using docker)
```

If you want a per-cookbook, verbose line by line breakdown of your deprecations, run:
```bash
$ for cookbook in ./*; do echo "==> Cookbook: $cookbook" >> /tmp/deprecations.txt && chef exec cookstyle -D  "$cookbook" >> /tmp/deprecations.txt; done
```

If you want a frequency analysis of the deprecations in your estate, do the following:

Let's clean up some of the info in the text file by clearing empty lines and lines containing "--"
```bash
$ for cookbook in ./*; do chef exec cookstyle -D --format offenses "$cookbook" >> /tmp/sanitized-deprecations.txt; done
$ sed -i '/^$/d' /tmp/sanitized-deprecations.txt
$ sed -i '/--$/d' /tmp/sanitized-deprecations.txt
```

Now create a quick ruby script with the following: `vim /tmp/frequency.rb`
```ruby
collector = {}
File.open('sanitized-deprecations.txt').read().each_line do |line|
  split = line.split(' ')
  offense = split[1]
  count = Integer(split[0])
  if collector[offense].nil?
    collector[offense] = count
  else
    collector[offense] += count
  end
end
collector = collector.sort_by{|offense, count| [-count, offense]}.to_h
puts collector
```

Now run the script with `ruby frequency.rb >> deprecations-frequency.txt` and observe the output. What you'll see is a hash of the deprecations and their prevalence in your estate. This is the scope of what needs to be fixed.


### Which deprecations do I fix first?

Now you have your text file filled with the deprecations that need fixin' (`deprecations.txt`) and the frequency with which they occur in your codebase (`deprecations-frequency.txt`). If you're curious, you can begin looking at the [Cookstyle Cops Documentation](https://github.com/chef/cookstyle/blob/master/docs/cops.md) and determine which offenses can be auto-corrected. Yes, I said auto-corrected.

If you'd like to proceed with said auto-correction, perform:

```bash
$ cookstyle -a cookbook_name
```

Once this is finished you'll receive output indicating the number of offenses found and the number corrected. If everything is corrected, **please perform the necessary integration testing before releasing the cookbook to your environment.** If there are offenses left over, consult the documentation and proceed with the manual correction, then integrate as previously stated.

### Credit

Thank you to [Nolan Davidson](https://github.com/nsdavidson) for the frequency ruby script!