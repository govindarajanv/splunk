# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define :splunk do |splunk|
    splunk.vm.box = "ubuntu/xenial64"
    splunk.vm.network "private_network", ip: "192.168.33.10"
    splunk.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
    end
	splunk.vm.hostname = "splunk"
	splunk.vm.provision "shell", inline: <<-URLS
	    echo http://localhost > /usr/share/urls.txt
        echo http://localhost/fake >> /usr/share/urls.txt
	    echo http://localhost/wrong >> /usr/share/urls.txt
        echo http://localhost/bad_url >> /usr/share/urls.txt
        echo http://localhost/404.html >> /usr/share/urls.txt
	  URLS
	
    splunk.vm.provision "shell", inline: <<-SHELL
      sysctl -w kernel.hostname="splunk"
  	  apt-get update
      apt-get install -y wget nginx siege sysstat
	  systemctl enable nginx
	  systemctl start nginx
	  (crontab -l 2>/dev/null; echo "*/5 * * * * /usr/bin/siege -t 10s -i -f /usr/share/urls.txt") | crontab -
	  shutdown -r
      SHELL
  end
  config.vm.define :gitea do |gitea|
    gitea.vm.box = "ubuntu/xenial64"
    gitea.vm.network "private_network", ip: "192.168.33.11"
    gitea.vm.provider "virtualbox" do |vb|
     vb.memory = "256"
    end
	gitea.vm.hostname = "gitea"
    gitea.vm.provision "shell", inline: <<-SHELL
	 sysctl -w kernel.hostname="gitea"
     apt-get update
     apt-get install -y git sysstat
     wget -O gitea https://dl.gitea.io/gitea/1.5.0/gitea-1.5.0-linux-amd64
     chmod +x gitea
	 mv gitea /usr/local/bin/
	 adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/git git
	 mkdir -p /var/lib/gitea/{custom,data,indexers,public,log}
     chown git:git /var/lib/gitea/{data,indexers,log}
     chmod 750 /var/lib/gitea/{data,indexers,log}
     mkdir /etc/gitea
     chown root:git /etc/gitea
     chmod 770 /etc/gitea
	 curl https://raw.githubusercontent.com/go-gitea/gitea/master/contrib/systemd/gitea.service > /etc/systemd/system/gitea.service
	 systemctl enable gitea
	 systemctl start gitea
	 shutdown -r
	SHELL
  end
  config.vm.define :nginx do |nginx|
    nginx.vm.box = "ubuntu/xenial64"
    nginx.vm.network "private_network", ip: "192.168.33.12"
    nginx.vm.provider "virtualbox" do |vb|
     vb.memory = "256"
    end
	nginx.vm.hostname = "nginx"
    nginx.vm.provision "shell", inline: <<-SHELL
	 sysctl -w kernel.hostname="nginx"
     apt-get update
     apt-get install -y nginx sysstat
	 echo "
	 server {
      listen       80;
      location / {
        proxy_pass http://192.168.33.11:3000;
      }
     }" > /etc/nginx/sites-enabled/default
	 systemctl restart nginx
	 systemctl enable nginx
	 shutdown -r
	SHELL
  end
    
end
