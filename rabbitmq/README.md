# RabbitMQ Highlander Demo

MVP RabbitMQ environment for use with shipping log entries to ELK.

If you do not have an existing OSS ELK installed and running,
please go up a directory and stand one up using the scripts in
`elk/`.

## MVP RabbitMQ Installation

To get a running RabbitMQ installation on CentOS 7, after standing
up a CentOS 7 droplet / instance / VM, please run the
`rabbitmq-install` script. Included in the setup script is adding
the repo file for Erlang, since in order to run RabbitMQ on CentOS 7
you'll need to have [Erlang version 21.3+ or 22.0+ installed](https://www.rabbitmq.com/which-erlang.html).

## Using the Demo App

If you do not have an existing app to use the service, please run 
the `rabbitmq-demo-app` script to download a demo Java app. Requires maven, 
which will install if not present.

If you are shipping to Logzio, make sure to include your token in
the logstash configuration file. To find the line run a grep in
shell:

```
grep -rn FIXME
```

This will show the other, optional, FIXMEs as well.

Locations for files:
- Filebeat configuration file should be in: `/etc/filebeat/filebeat.yml`
- Logstash configuration file should be in: `/etc/logstash/conf.d/rabbitmq.conf`

## How to Use the Configuration Files

There are a few methods for shipping the RabbitMQ logs to an ELK
stack, in this case they are as follows:
- Filebeat, Metricbeat -> Logzio platform
- Filebeat, Metricbeat -> OSS Elasticsearch
- Filebeat, Metricbeat -> Logstash -> Logzio platform
- Filebeat, Metricbeat -> Logstash -> OSS Elasticsearch

In both the Filebeat and Logstash configuration files you'll
notice some outputs are commented out. This is so that you have
all the necessary configuration lines you need - simply comment /
uncomment the matching lines for your desired configuratin. e.g.
if you are shipping direct to Logzio from Filebeat, use the
"Shipping direct to Logzio" section under Outputs and do not
configure the Logstash configuration. If you wish to use Logstash,
which I recommend since shipping from Filebeat won't parse your
logs at all, then in the Filebeat configuration you'll need to use
the "Shipping to local Logstash section" and in the Logstash
configuration file you'll need to uncomment the appropriate
output, which is either Elasticsearch or Logzio.

## Some things of note for Metricbeat

In earlier versions of the setup, I ran into a couple of errors:

```
$ ./metricbeat modules list
Error initializing beat: error loading config file: stat
metricbeat.yml: no such file or directory
```

This can be solved a couple of ways. One is by using `-c` to
explicitly supply the path to the configuration file, like so:

```
$ ./metricbeat -c /etc/metricbeat/metricbeat.yml modules list
Error initializing beat: error loading config file: open
/etc/metricbeat/metricbeat.yml: permission denied
```

Note that `metricbeat` needs to be run via `sudo`, so:

```
$ sudo ./metricbeat -c /etc/metricbeat/metricbeat.yml modules list
Enabled:

Disabled:
```

You can see here that simply supplying the configuration path
wasn't enough to fix the issue, as the RabbitMQ module (and
others) would not be recognized:

```
$ sudo ./metricbeat -c /etc/metricbeat/metricbeat.yml modules enable rabbitmq
Module rabbitmq doesn't exist!
```

At this point I (and you) should verify that the modules are in
fact in the `modules.d` directory (they are in this case):

```
ls /etc/metricbeat/modules.d/
aerospike.yml.disabled  dropwizard.yml.disabled
jolokia.yml.disabled       mssql.yml.disabled
redis.yml.disabled
apache.yml.disabled     elasticsearch-xpack.yml.disabled
kafka.yml.disabled         munin.yml.disabled       system.yml
{{ output truncated }}
```

So what's going on? In this case supplying the path to _only_ the
configuration file doesn't tell `metricbeat` where the rest of
it's relevant components are - including the modules. To fix that,
use `--path.home`:

```
$ sudo ./metricbeat --path.home /etc/metricbeat/ modules list
Enabled:
rabbitmq
system

Disabled:
aerospike
apache
aws
ceph
{{ output truncated }}
```

At this point you should be able to view, enable, and disable all
the plugins available to Metricbeat.

**Note:** You should only run the Metricbeat command as
`./metricbeat` if `metricbeat` doesn't work (e.g. there's nothing
in `which metricbeat`). In my case, the binary was in
`/usr/share/meticbeat/bin`, but your mileage may vary.

## Addendum: Installing RabbitMQ Configuration
Some additional notes about this setup that I ran into:

<u>For Erlang</u>: there is no default package for Erlang in
CentOS, but there is in the EPEL (Extra Packages for Enterprise Linux) repository. The
current version of Erlang in EPEL is
[R16B03-1](https://www.erlang.org/downloads/R16B03-1), and isn't
compatible with the current releases of RabbitMQ, which is why
it's still important to add the repo file. When you try to install
RabbitMQ using the version of Erlang with EPEL, you'll see a
message like this:

```
$ sudo rpm -Uvh rabbitmq-server-3.7.9-1.el7.noarch.rpm
warning: rabbitmq-server-3.7.9-1.el7.noarch.rpm: Header V4
RSA/SHA1 Signature, key ID 6026dfca: NOKEY
error: Failed dependencies:
    erlang >= 19.3 is needed by rabbitmq-server-3.7.9-1.el7.noarch
```

If you see the following error when you tried to install RabbitMQ,
you'll need to uninstall Erlang, add the repo file from earlier,
and reinstall. Note that the error message tells you what version
of Erlang you need - for the version of RabbitMQ I am using here,
I'll need 19.3+. For the most current releases of RabbitMQ you'll
need 22.0+. If you want to check the version of Erlang, you need
to look at its `releases` directory:

```
$ readlink -f $(which erl)
/usr/lib64/erlang/erts-10.4.4/bin/erl

$ ls /usr/lib64/erlang/releases/
22  RELEASES  RELEASES.src  start_erl.data

$ cat /usr/lib64/erlang/releases/RELEASES
%% coding: utf-8
[{release,"Erlang/OTP","22","10.4.4",
          [{kernel,"6.4.1","/usr/lib64/erlang/lib/kernel-6.4.1"},
           {stdlib,"3.9.2","/usr/lib64/erlang/lib/stdlib-3.9.2"},
           {sasl,"3.4","/usr/lib64/erlang/lib/sasl-3.4"}],
          permanent}].
```

I ran the above for my installation of Erlang, which is 22.0. You
can see the major release version as a directory in `releases` and
can `cat` the `RELEASES` file for more granular information.

<u>For RabbitMQ</u>: If you are using an existing RabbitMQ setup
you can check its version with the `rabbitmqctl` command. Since the
command has a lot of output I recommend using `grep` and look for
the `{rabbit, "RabbitMQ", "${VERSION}"}` line:

```
$  sudo rabbitmqctl status | grep rabbit
Status of node rabbit@ip-xxx-xxx-xxx-xxx ...
     [{rabbitmq_jms_topic_exchange,
      {rabbitmq_management,"RabbitMQ Management Console","3.7.9"},
      {rabbitmq_management_agent,"RabbitMQ Management Agent","3.7.9"},
      {rabbitmq_web_dispatch,"RabbitMQ Web Dispatcher","3.7.9"},
      {rabbit,"RabbitMQ","3.7.9"},
      {rabbit_common,
          "Modules shared by rabbitmq-server and rabbitmq-erlang-client",
```
