# Running Chef and OSS ELK on Single Server for Demo

##Requirements

- CentOS 7
- For AWS, minimum functional instance is type `t3.large`
- Chef + ELK ports need to be open
  - For Chef: 80 & 443 (HTTP and HTTPS), 9683
  - For ELK: 9200, 9300, 5601, 5044
- Run `grep -rn "FIXME"` to see the lines where you should personalize your demo
  - Chef + ELK installation: change the variables for customizing name, org, and whether you wish to install version 7.0.0 or 6.5.4.
  - `filebeat-demo.yml` : When shipping to the Logzio platform you'll need to supply the appropriate account token
  - `elasticsearch-demo.yml` : You'll need to supply your demo server / instance's IP

# The installation

Once you've made the optional edits to the Chef install and/or the
mandatory edit in the Elasticsearch configuration (by finding the
FIXMEs), the full demo can be installed with:

```
./chef-highlander-demo
```

If you already have ELK running elsewhere and just need the Chef
demo:

```
./chef-install-only
```

# The Results

If you are using this I recommend following along the intended
setup as outlined in the corresponding blog post (TBD). The
`*.final.*` files are what your resulting files should look like
for that specific demo / tutorial.

# Troubleshooting

## Filebeat, Why?

The most common issue I encountered with Filebeat was the process
failing to start / restart due to a misconfigured YAML file. This
is usually immediately visible by checking the process status and
then digging into `/var/log/messages`:

```bash
sudo service filebeat status

sudo cat /var/log/messages | grep filebeat | less
```

## Why, Logstash, Why?

Because building `grok` patterns can be hazardous. Typically
errors would appear during an attempt at a restart. This is
visible by checking the status, similar to Filebeat, and then
looking at either the `logstash-plain.log` or `/var/log/messages`
as applies. If you're making several, successive, changes it's 
probably worth just tmux'ing and tailing the logs in separate
windows:

```bash
sudo tail -f /var/log/logstash/logstash-plain.log

sudo tail -f /var/log/messages | grep logstash
```

## Chef feels left out

The only issue I encountered with Chef during this whole process
was that the default user's password doesn't seem to always set
properly. Probably because of how quickly all the setup is done.
If that happens, just set the user's password from the command
line:

```bash
sudo chef-server-ctl password ${CHEF_USERNAME}
```

When troubleshooting Logstash, typically grok patterns, it's
helpful to have a window open that tails the relevant Chef logs.
This way, if the logs don't make it into Logstash or Elasticsearch
you can see that they are actually being generated. If you want to
see all the logs:

```bash
sudo chef-server-ctl tail
```

Or for a specific service:

```bash
sudo chef-server-ctl tail nginx
```

Simple regex works too, e.g. a wildcard:

```bash
sudo chef-server-ctl tail opscode-*
```
