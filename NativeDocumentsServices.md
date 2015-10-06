# NativeDocumentsServices
## Ubuntu
### Preparation / Setup
<pre><code>sudo apt-get update        # Always update!</code></pre>
### Install NativeDocumentsServices as a Local Service (Recommended)
<pre><code>sudo apt-get install curl --yes && \
curl -O http://nativedocuments.wordsdk.com/downloads/NativeDocumentsServices.initd.x86_64.deb && \
sudo dpkg --install NativeDocumentsServices.initd.x86_64.deb && \
bash -c 'source /opt/NativeDocumentsServices/NativeDocumentsServices.config && \echo Listening on $NDS_SERVICE_ARGS'</code></pre>
### Manually start / stop the service
<pre><code>sudo initctl stop NativeDocumentsServices    # start service manually
sudo initctl start NativeDocumentsServices   # stop service manually</code></pre>
###Expose service externally (Not Recommended; For Debug or Load Balancing)
<pre><code># Expose the service at port 9015
sudo initctl stop NativeDocumentsServices; \
sudo sed 's/^NDS_SERVICE_ARGS=.*$/NDS_SERVICE_ARGS=0.0.0.0:9015/g' -i /opt/NativeDocumentsServices/NativeDocumentsServices.config && \
sudo initctl start NativeDocumentsServices
# Open Firewall
sudo apt-get install iptables-persistent --yes && bash -c 'source /opt/NativeDocumentsServices/NativeDocumentsServices.config && sudo iptables -I INPUT -p tcp -m tcp --dport `echo $NDS_SERVICE_ARGS | sed s/.*://g` -j ACCEPT && sudo service iptables-persistent save'</code></pre>
###Expose service via NGINX (Production System)
<pre><code># Install NGINX
sudo /etc/init.d/nginx stop;
sudo apt-get install nginx --yes && \
sudo /etc/init.d/nginx stop; \
sudo bash -c 'source /opt/NativeDocumentsServices/NativeDocumentsServices.config && echo -e "\
server {\n\
  listen 8080;\n\
  location / {\n\
      proxy_pass http://127.0.0.1:`echo $NDS_SERVICE_ARGS | sed s/.*://g`;\n\
      proxy_read_timeout 60m;\n\
      proxy_set_header Host \$host;
      client_max_body_size 50m;\n\
  }\n\
  error_page   500 502 503 504  /50x.html;\n\
  location = /50x.html {\n\
      root   html;\n\
  }\n\
}\n" > /etc/nginx/conf.d/nds.conf' && sudo /etc/init.d/nginx start
# Open Firewall
sudo apt-get install iptables-persistent --yes && bash -c 'source /opt/NativeDocumentsServices/NativeDocumentsServices.config && sudo iptables -I INPUT -p tcp -m tcp --dport 8080 -j ACCEPT && sudo service iptables-persistent save'</code></pre>
