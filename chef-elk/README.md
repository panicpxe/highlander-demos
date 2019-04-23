# Running Chef and OSS ELK on Single Server for Demo

Requirements:
- CentOS 7
- Chef + ELK ports need to be open
  - For Chef: 80 & 443 (HTTP and HTTPS), 9683
  - For ELK: 9200, 9300, 5601, 5044
- Run `grep -rn "FIXME"` to see the lines where you should personalize your demo
  - Chef + ELK installation: change the variables for customizing name, org, and whether you wish to install version 7.0.0 or 6.5.4.
  - `filebeat-demo.yml` : When shipping to the Logzio platform you'll need to supply the appropriate account token
  - `elasticsearch-demo.yml` : You'll need to supply your demo server / instance's IP
