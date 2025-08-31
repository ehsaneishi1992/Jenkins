# Add Docker's official GPG key:
```bash
apt-get update
apt-get install ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

# Add the repository to Apt sources:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list
```
## install docker and docker compose
```bash
apt-get install docker.io docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose
```  

# set arvan cloud repository
```bash
bash -c 'cat > /etc/docker/daemon.json <<EOF
{up
  "insecure-registries" : ["https://docker.arvancloud.ir"],
  "registry-mirrors": ["https://docker.arvancloud.ir"]
}
EOF' 

docker logout
systemctl restart docker 
```
# Jenkins with Docker Compose
```bash
mkdir -p devops/jenkins
cd devops/jenkins
nano docker-compose.yml
```
```yaml
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: always
    ports:
      - "8080:8080"
      - "50000:50000"  # For Jenkins agent communication
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock  # For Docker inside Jenkins
    environment:
      JENKINS_OPTS: --httpPort=8080

volumes:
  jenkins_home:
```
### Access Jenkins
Open your browser and go to http://<your-server-ip>:8080.

To unlock Jenkins, you'll need the initial password:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
Copy the password and paste it into the Jenkins setup screen.

### Install Docker Inside Jenkins
To allow Jenkins to run Docker commands inside jobs, you need to install Docker inside the Jenkins container:

Access the Jenkins container:

docker exec -it jenkins bash
Inside the Jenkins container, install Docker:

```bash
docker ps
docker exec -u root -it jenkins bash
apt update
apt install -y docker.io
usermod -aG docker jenkins
```
##### in first step install Plugins "SSH Agen"

### Create the Jenkins User in agent server;
```bash
sudo adduser jenkins
sudo usermod -aG sudo jenkins

visudo
jenkins ALL=(ALL) NOPASSWD:ALL
```
### Use SSH Keys Instead of Passwords:

On the Jenkins server, generate an SSH key pair:
```bash
ssh-keygen -t rsa -b 4096 -C "jenkins@agent-server-ip"
ssh-keygen -t rsa -b 4096 -C "jenkins@customer1" -f /var/jenkins_home/.ssh/id_rsa_customer1
ssh-keygen -t rsa -b 4096 -C "jenkins@customer2" -f /var/jenkins_home/.ssh/id_rsa_customer2
```
Then, copy the public key to the remote machine:
```bash
ssh-copy-id root@agent-server-ip
ssh-copy-id -i /var/jenkins_home/.ssh/id_rsa_customer1.pub -p PORT jenkins@customer1-server-ip
```
Add the Host Key Manually: From the Jenkins server, add the target server’s SSH key to the known_hosts file:

```bash
ssh-keyscan -p PORT agent-server-ip >> /var/jenkins_home/.ssh/known_hosts
```
now "Add Credentials" and create new with "SSH username with private key" kind and Enter Private Key generated in jenkins server

Remember to enter the "Remote root directory" in define node 

Create the Known Hosts File: Ensure the directory /var/jenkins_home/.ssh/ exists and is writable by Jenkins.

```bash
mkdir -p /var/jenkins_home/.ssh 
touch /var/jenkins_home/.ssh/known_hosts 
chmod 600 /var/jenkins_home/.ssh/known_hosts
chown -R jenkins:jenkins /var/jenkins_home/.ssh
```
### Install OpenJDK via a PPA (Personal Package Archive) in agent server
```bash
apt install software-properties-common
add-apt-repository ppa:openjdk-r/ppa
apt update
apt install openjdk-17-jdk
```
