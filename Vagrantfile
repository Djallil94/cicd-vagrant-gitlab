Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    set -e
    export DEBIAN_FRONTEND=noninteractive

    echo "=== 1. Update système ==="
    apt-get update -y
    apt-get upgrade -y
    apt-get install -y wget apt-transport-https gpg git curl unzip

    echo "=== 2. Java 24 (Temurin) ==="
    wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public \
      | gpg --dearmor | tee /etc/apt/trusted.gpg.d/adoptium.gpg > /dev/null
    echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" \
      | tee /etc/apt/sources.list.d/adoptium.list
    apt-get update -y
    apt-get install -y temurin-24-jdk

    echo "=== 3. GitLab Runner ==="
    curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | bash
    apt-get update -y
    apt-get install -y gitlab-runner

    echo "=== 4. User myapp & app dir ==="
    useradd -r -s /usr/sbin/nologin myapp || true
    mkdir -p /opt/myapp
    chown -R myapp:myapp /opt/myapp

    echo "=== 5. Service systemd ==="
    cat > /etc/systemd/system/myapp.service <<EOF
[Unit]
Description=My Java App
After=network.target

[Service]
User=myapp
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar
Restart=always

[Install]
WantedBy=multi-user.target
EOF
    systemctl daemon-reload
    systemctl enable myapp

    echo "=== 6. Permissions sudo ==="
    cat > /etc/sudoers.d/gitlab-runner-deploy <<EOF
gitlab-runner ALL=(ALL) NOPASSWD: /bin/systemctl start myapp, /bin/systemctl stop myapp, /bin/systemctl restart myapp
gitlab-runner ALL=(ALL) NOPASSWD: /usr/bin/install
EOF
    chmod 440 /etc/sudoers.d/gitlab-runner-deploy

    echo "PROVISION TERMINÉ ! Runner installé."
  SHELL
end
