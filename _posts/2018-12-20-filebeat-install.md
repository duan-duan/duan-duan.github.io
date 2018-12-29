```sh
[root@Apache ~]# cd /usr/local/filebeat/
[root@Apache filebeat]# ls
data  fields.yml  filebeat  filebeat.reference.yml  filebeat.yml  kibana  LICENSE.txt  module  modules.d  NOTICE.txt  README.md
[root@Apache filebeat]# vim filebeat.yml(只修改如下几段）
#=========================== Filebeat inputs =============================
filebeat.inputs:
- type: log
  paths:
    - "/var/log/httpd/*" 此处日志位置要注意 我使用的是yum安装的apache
  fields:
    apache: true
  fields_under_root: true
#============================== Kibana =====================================
 
# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:
 
  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "192.168.8.104:5601"
#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
 hosts: ["192.168.8.103:5044"]
 
  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
 
  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"
 
  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"
[root@Apache filebeat]./filebeat -e -c filebeat.yml

#后台运行 
nohup ./filebeat -e -c filebeat.yml &
```

