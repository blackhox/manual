# Setting up Validator Monitoring for Cosmos #
This is a detailed guide on how to set up validator monitoring for Cosmos-based blockchains using Prometheus and Grafana.
This guide is for validators who need to set up basic monitoring of their nodes. This manual does not cover all possible monitoring options. 
To set up deeper monitoring, it is recommended to refer to the official documentation.
# Set up a Prometheus server #
First, we need to set up the Prometheus server. 
It will collect all the node monitoring parameters. Prometheus is a database that feeds various diagnostic parameters in near real time.
To install the Prometheus server, it is strongly recommended to allocate a separate server, of course, you can run it along with other programs, but the performance of such a system may be impaired. It is strongly not recommended to run Prometheus on the same server as a sentry or validator, as this will lead to unnecessary load on the server, and the lack of system resources in turn will lead to block skipping.
First of all, let's start the installation by updating the OS and installing security software: **fail2ban**
It won't solve every possible security problem, but it's better than doing nothing at all.
```
sudo apt-get update -y && sudo apt-get upgrade -y && sudo apt install fail2ban -y
```
During the installation of updates to these packages, you may occasionally see a purple screen. Just press the ENTER button.
After updating the Linux OS, create the prometheus user that will be used to run Prometheus.
```
sudo groupadd --system prometheus

sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
Let's remove everything unnecessary, and then download and install Prometheus.
```
sudo mkdir /var/lib/prometheus
for i in rules rules.d files_sd; do sudo mkdir -p /etc/prometheus/${i}; done
mkdir -p /tmp/prometheus && cd /tmp/prometheus
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
//or if the download fails
wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-amd64.tar.gz

tar xvf prometheus*.tar.gz
cd prometheus*/
sudo mv prometheus promtool /usr/local/bin/
```
If you have successfully completed all the previous steps,
you will see the version numbers of installed programs: Prometheus and Promtool
```
prometheus --version
promtool --version
```
Let's move some files like this:

While in the tmp folder, run
```
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
```
Now that all the preparatory work is done, you can set up Prometheus as a service so that it runs all the time!
```
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=
SyslogIdentifier=prometheus
Restart=always
[Install]
WantedBy=multi-user.target
EOF
```
Arranging access rights:
```
for i in rules rules.d files_sd; do sudo chown -R prometheus:prometheus /etc/prometheus/${i}; done
for i in rules rules.d files_sd; do sudo chmod -R 775 /etc/prometheus/${i}; done
sudo chown -R prometheus:prometheus /var/lib/prometheus/
```
Tell systemctl that we created the Prometheus service, add a symbolic link, run it and check the logs
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```
If Prometheus is running successfully, you should see a status screen that looks like this. Press CTL+C to exit the systemctl status screen.
![Prometeus logs](https://miro.medium.com/max/700/1*TYE67j8UZfUIl1OtpBDIIg.png "Log")

Let's increase the server's security level a little more and turn on the firewall.
If you are using the standard port 22 for SSH then it will be available and will be able to connect. If you are using a non-standard port for SSH, then change the command below to include the port, otherwise it will block access to the server.
```
sudo ufw allow proto tcp from any to any port 22
sudo ufw allow proto tcp from any to any port 9090
sudo ufw enable
```
If you have successfully reached this point, then you have a working Prometheus server. We will return later in the manual to its additional configuration. In the meantime, install the Grafana server

# Set up a Grafana Server #
The next step is to launch a Grafana instance which will allow you to visualize data in Prometheus from your computer and more importantly from your smartphone! Now you have a chance not to sit constantly at the computer terminal.

It is recommended to run a separate server for Grafana. Ultimately, Grafana will provide a web server to the public internet, and you are unlikely to want to pair it with your Prometheus server. If it is possible to install these two servers on the same local network, then this solution would be preferable. In this case, you can connect them privately without having to use the public Internet.

Once we set up the Grafana server, we will also update it and install ```fail2ban```. Again, this server is open to the internet, so consider additional security measures such as ```multi-factor authentication``` and ```no root login```.
```
sudo apt install fail2ban -y
```
**Now install some dependencies for Grafana**
```
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
**Now update your package repos**
```
echo "deb https://packages.grafana.com/enterprise/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
**Install Grafana and upgrade your packages**
```
sudo apt-get update -y && sudo apt-get install grafana-enterprise -y && sudo apt-get upgrade -y
```
Now let's start Grafana as a service:
```
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```
