.... ELK (Elasticsearch, Kibana, Logstash, Filebeat and nginx)
# 403 !!!

rm /etc/resolv.conf

echo "nameserver 10.202.10.202 " | sudo tee -a /etc/resolv.conf

echo "nameserver 10.202.10.102 " | sudo tee -a /etc/resolv.conf


apt update

curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch |sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

sudo apt update

sudo apt install elasticsearch -y

sudo systemctl start elasticsearch

sudo systemctl enable elasticsearch
#curl -X GET "localhost:9200"

apt install kibana -y

systemctl start kibana

systemctl enable kibana

apt install -y nginx

systemctl start nginx

systemctl enable nginx

apt install apache2-utils -y

htpasswd -c /etc/nginx/.htpasswd hkh

cp /etc/nginx/nginx.conf nginx.conf.back

#vi /etc/nginx/nginx.conf

nginx -t

systemctl reload nginx

sudo apt install filebeat -y

#vi /etc/filebeat/filebeat.yml

sudo filebeat modules enable system

sudo filebeat setup --pipelines --modules system

sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'

sudo filebeat setup -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601

sudo systemctl start filebeat

sudo systemctl enable filebeat
