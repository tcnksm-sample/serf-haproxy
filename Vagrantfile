$script = <<SCRIPT

sudo apt-get install -y unzip
cd /tmp/
wget https://dl.bintray.com/mitchellh/serf/0.5.0_linux_amd64.zip -O serf.zip

unzip serf.zip
chmod +x serf
mv serf /usr/bin/serf

SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  config.vm.provision :shell, inline: $script

  config.vm.define :proxy do |proxy|
    proxy.vm.network "private_network", ip: "172.20.20.10"
    proxy.vm.provision :shell, inline: <<SCRIPT

apt-get -y install haproxy
sed -i -e 's/ENABLED=0/ENABLED=1/' /etc/default/haproxy

cat <<EOF >/tmp/haproxy.cfg
global
    daemon
    maxconn 256

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

listen stats
    bind *:9999
    mode http
    stats enable
    stats uri /
    stats refresh 1s

listen http-in
    bind *:80
    balance roundrobin
    option http-server-close
EOF

mv /tmp/haproxy.cfg /etc/haproxy/haproxy.cfg

/etc/init.d/haproxy start

SCRIPT
  end

  config.vm.define :web1 do |web|
    web.vm.network "private_network", ip: "172.20.20.111"
    web.vm.provision :shell, inline: <<SCRIPT
apt-get -y update
apt-get -y install nginx
echo '<h1>web1</h1>' > /usr/share/nginx/www/index.html
/etc/init.d/nginx start
SCRIPT
  end

  config.vm.define :web2 do |web|
    web.vm.network "private_network", ip: "172.20.20.112"
    web.vm.provision :shell, inline: <<SCRIPT
apt-get -y update
apt-get -y install nginx
echo '<h1>web2</h1>' > /usr/share/nginx/www/index.html
/etc/init.d/nginx start
SCRIPT
  end
end
