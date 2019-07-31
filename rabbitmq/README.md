# RabbitMQ Highlander Demo

MVP RabbitMQ environment for use with shipping log entries to ELK.

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
