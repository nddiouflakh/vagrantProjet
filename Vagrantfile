Vagrant.configure("2") do |config|

  # Configuration de la VM devapp01
  config.vm.define "devapp01" do |devapp|
    devapp.vm.box = "bento/ubuntu-22.04"
    devapp.vm.box_check_update = false
    devapp.vm.network "private_network", ip: "172.16.238.10"
    
    devapp.vm.provider "parallels" do |prl|
      prl.memory = "2048"
      prl.name = "devapp01"
    end
    devapp.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y openjdk-11-jdk tomcat9 git
      sudo systemctl start tomcat9
      sudo systemctl enable tomcat9

      # Setup cron job for code backup
      echo '0 * * * * vagrant rsync -a /path/to/code/ vagrant@172.16.238.12:/path/to/backup/code/' | sudo tee /etc/cron.d/code_backup
      sudo service cron restart
    SHELL
  end

  # Configuration de la VM dbapp01
  config.vm.define "dbapp01" do |dbapp|
    dbapp.vm.box = "bento/ubuntu-22.04"
    dbapp.vm.box_check_update = false
    dbapp.vm.network "private_network", ip: "172.16.238.11"
    
    dbapp.vm.provider "parallels" do |prl|
      prl.memory = "2048"
      prl.name = "dbapp01"
    end
    dbapp.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y postgresql postgresql-contrib
      
      # Configurer PostgreSQL pour accepter les connexions uniquement depuis devapp01
      echo "host all all 172.16.238.10/32 md5" | sudo tee -a /etc/postgresql/12/main/pg_hba.conf
      echo "listen_addresses = '*'" | sudo tee -a /etc/postgresql/12/main/postgresql.conf
      sudo systemctl restart postgresql

      # Setup cron job for database backup
      echo '*/30 * * * * vagrant pg_dumpall -U postgres | gzip > /path/to/backup/db/db_backup_$(date +\\%F_\\%T).sql.gz' | sudo tee /etc/cron.d/db_backup
      sudo service cron restart
    SHELL
  end

  # Configuration de la VM backup01
  config.vm.define "backup01" do |backup|
    backup.vm.box = "bento/ubuntu-22.04"
    backup.vm.box_check_update = false
    backup.vm.network "private_network", ip: "172.16.238.12"
    
    backup.vm.provider "parallels" do |prl|
      prl.memory = "1024"
      prl.name = "backup01"
    end
    backup.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y rsync
      sudo mkdir -p /path/to/backup/code
      sudo mkdir -p /path/to/backup/db
    SHELL
  end

end
