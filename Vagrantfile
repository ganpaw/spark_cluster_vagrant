Vagrant.configure("2") do |config|
  N_WORKERS = 1

  config.vm.box = "centos/7"

  # head node:
  config.vm.define "hn0" do |node|
    node.vm.hostname = "hn0"
    node.vm.network "private_network", ip: "172.28.128.150"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "hn0"
      vb.gui = false
      vb.memory = 512
      vb.cpus = 1
    end

    node.vm.provision "shell", inline: <<-SHELL 
      mkdir /home/vagrant/spark
      chmod 755 /vagrant/jdk-7u80-linux-x64.rpm
      rpm -ivh /vagrant/jdk-7u80-linux-x64.rpm
      tar xf /vagrant/spark-1.5.1-bin-hadoop2.6.tgz -C /home/vagrant/spark --strip 1
      sudo chown vagrant:vagrant -R /home/vagrant/spark/
      echo "export JAVA_HOME=/usr/java/jdk1.7.0_80" >> /home/vagrant/.bash_profile
      echo "export SPARK_HOME=/home/vagrant/spark" >> /home/vagrant/.bash_profile
      echo "export PATH=$PATH:/usr/java/jdk1.7.0_80/bin:/home/vagrant/spark/bin" >> /home/vagrant/.bash_profile
      source /home/vagrant/.bash_profile
      echo "spark.master spark://172.28.128.150:7077" >> /home/vagrant/spark/conf/spark-defaults.conf
      echo "SPARK_LOCAL_IP=172.28.128.150" >> /home/vagrant/spark/conf/spark-env.sh
      echo "SPARK_MASTER_IP=172.28.128.150" >> /home/vagrant/spark/conf/spark-env.sh
    SHELL

    node.vm.provision "shell", run: "always", inline: "/home/vagrant/spark/sbin/start-master.sh -h 172.28.128.150"
  end
  # worker nodes:
  (0..N_WORKERS-1).each do |i|
    config.vm.define "wn#{i}" do |node|
      node.vm.hostname = "wn#{i}"
      node.vm.network "private_network", ip: "172.28.128.16#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.name = "wn#{i}"
        vb.gui = false
        vb.memory = 512
        vb.cpus = 1
      end

      node.vm.provision "shell", inline: <<-SHELL
        mkdir /home/vagrant/spark
        chmod 755 /vagrant/jdk-7u80-linux-x64.rpm
        rpm -ivh /vagrant/jdk-7u80-linux-x64.rpm
        tar xf /vagrant/spark-1.5.1-bin-hadoop2.6.tgz -C /home/vagrant/spark --strip 1
        sudo chown vagrant:vagrant -R /home/vagrant/spark/
        echo "export JAVA_HOME=/usr/java/jdk1.7.0_80" >> /home/vagrant/.bash_profile
        echo "export SPARK_HOME=/home/vagrant/spark" >> /home/vagrant/.bash_profile
        echo "export PATH=$PATH:/usr/java/jdk1.7.0_80/bin:/home/vagrant/spark/bin" >> /home/vagrant/.bash_profile
        source /home/vagrant/.bash_profile
        echo SPARK_LOCAL_IP=172.28.128.16#{i} >> /home/vagrant/spark/conf/spark-env.sh
        echo SPARK_MASTER_IP=172.28.128.150 >> /home/vagrant/spark/conf/spark-env.sh        
      SHELL

      node.vm.provision "shell", run: "always", inline: "/home/vagrant/spark/sbin/start-slave.sh -h 172.28.128.16#{i} spark://172.28.128.150:7077"
    end
  end

end
