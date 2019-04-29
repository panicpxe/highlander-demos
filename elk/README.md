# Running Chef and OSS ELK on Single Server for Demo

## Requirements

- CentOS 7
- For AWS, minimum functional instance is type `t3.large`
- ELK ports need to be open: 9200, 9300, 5601, 5044
- Run `grep -rn "FIXME"` to see the lines where you should personalize your demo
  - Chef + ELK installation: change the variables for customizing name, org, and whether you wish to install version 7.0.0 or 6.5.4.
  - `filebeat-demo.yml` : When shipping to the Logzio platform you'll need to supply the appropriate account token
  - `elasticsearch-demo.yml` : You'll need to supply your demo server / instance's IP

# The installation

Once you've changed the IP in `elasticsearch-demo.yml` to the
private IP of the instance:

```
./elk-highlander
```

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

