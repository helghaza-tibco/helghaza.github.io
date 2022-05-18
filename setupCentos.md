# Install Docker (https://docs.docker.com/engine/install/centos/)
```
sudo yum -y update
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl enable containerd.service
sudo usermod -aG docker $USER
```

# Install Java 11
```
sudo yum install java-11-openjdk
```

# Install Maven

```
wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.zip
unzip apache-maven-3.8.5-bin.zip
mv apache-maven-3.8.5 /usr/local/apache-maven
```
- Edit .bashrc file :
```
export M2_HOME=/usr/local/apache-maven
export M2=$M2_HOME/bin 
export PATH=$M2:$PATH
```

- Reload profile script and check maven installation:
```
source ~/.bashrc
mvn -version
```


# Install Jenkins
```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install jenkins
sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
- Open a browser, then go to your Linux box on the port 8080.
