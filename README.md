.... ELK (Elasticsearch, Kibana, Logstash, Filebeat and nginx)
# 403 !!!

rm /etc/resolv.conf

echo "nameserver 10.202.10.202 " | sudo tee -a /etc/resolv.conf

echo "nameserver 10.202.10.102 " | sudo tee -a /etc/resolv.conf


apt update

# Install Elasticsearch

curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch |sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

sudo apt update

sudo apt install elasticsearch -y

sudo systemctl start elasticsearch

sudo systemctl enable elasticsearch
#curl -X GET "localhost:9200"

# Logstash

sudo apt install -y logstash

systemctl start logstash

systemctl enable logstash

# Edit "/etc/logstash/conf.d/02-beats-input.conf" and customize it based on your environment.

touch  /etc/logstash/conf.d/02-beats-input.conf

# Also Edit "/etc/logstash/conf.d/30-elasticsearch-output.conf" and customize it based on your environment.

touch /etc/logstash/conf.d/30-elasticsearch-output.conf

sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t


# Install Kibana

apt install kibana -y

systemctl start kibana

systemctl enable kibana

# Install nginx

apt install -y nginx

systemctl start nginx

systemctl enable nginx

# Install apache2-utils 

apt install apache2-utils -y

htpasswd -c /etc/nginx/.htpasswd hkh

cp /etc/nginx/nginx.conf nginx.conf.back

#vi /etc/nginx/nginx.conf

nginx -t

systemctl reload nginx

# Install Filebeat

sudo apt install filebeat -y

#vi /etc/filebeat/filebeat.yml

sudo filebeat modules enable system

sudo filebeat setup --pipelines --modules system

sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'

sudo filebeat setup -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601

sudo systemctl start filebeat

sudo systemctl enable filebeat

# iptables (Hardening)

iptables -A INPUT -p tcp --dport 5601 -j DROP

iptables -A INPUT -p tcp --dport 9600 -j DROP

iptables -A INPUT -p tcp --dport 9200 -j DROP

iptables -A INPUT -p tcp --dport 5044 -j DROP

iptables -A INPUT -p tcp --dport 9300 -j DROP

iptables -A INPUT -p tcp --dport 80 -j DROP

iptables -I INPUT 1 -s 127.0.0.1 -j ACCEPT

sudo apt-get install -y iptables-persistent

sudo netfilter-persistent save
