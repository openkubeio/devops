
### Overview

Installing gitlab offline is pretty staright forward. Visit the official site for more insight 


### Install the dependent packages
```
sudo apt-get install -y curl openssh-server ca-certificates tzdata
```

### Download and add gitlab package repository to the ubuntu
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash*
```

### Verifying the external LBR

Before instaling set up a external url or a load balancer endpoint to access the gitlab applicaton. 
You can also use a your server IP addresss as well. Ensure that external IP is reachable.

echo "192.172.100.50 gitlab.openkube.io" >> /etc/hosts




### Install gitlab-ee with apy package manager
```
sudo EXTERNAL_URL="https://gitlab.openkube.io" apt-get install gitlab-ee
```



### Post install verify

Upon installation below are the major directories to find logs, data, configs and more.

|   &nbsp;             | &nbsp;                              |
|----------------------|-------------------------------------|
| logs                 | /var/log/gitlab/                    |
| configuration        | /etc/gitlab/                        |
| binaries             | /opt/gitlab/sv/                     |
| data                 |                                     |


**gitlab-ctl** cli is installed as part of installation. The cli can be use to manage gitlab configuration, status and much more.
verify status of gitlab services. This will print list and staus of the PID of the services running
gitlab-ctl status

**list all the services related to gitlab**

*gitlab-ctl service-list*


### Reconfigure application

To reconfigure the application, update the file /etc/gitlab/gitlab.rb <br/> 
run the below reconfigure command to apply the configuration<br/>

*gitlab-ctl reconfigure*


### Verifying App

Playing aound with certs in gitlab is pretty interstng. one can curl gitlab url as below.
However it will fail as no ca certificate is attched for cert validation, though it will work with --skip-insecure

*curl https://gitlab.openkube.io*

Lets download the ca cert for the app url usinng openssl tool. Below openssl command will print the certs chain and we
need to copy the cert content and paste in a new file gitlab-ca.crt. 
```
cd /etc/gitlab/ssl

openssl s_client -connect gitlab.openkube.io:443 –showcerts
```
Our ca cert is now ready for ssl cert verification.
we can attach the ca file gitlab-ca.crt to the curl for cert validation.
```
curl https://gitlab.openkube.io --cacert gitlab-ca.crt
```

we can add the ca cert to server certificate truststore, so curl will try to validate it against the server  truststore
to find the ca certificate there. 
```
mv gitlab-ca.crt   /usr/local/share/ca-certificates/gitlab-ca.crt

update-ca-certificates*
```

Now the gitlab-ca.crt is updated to the server truststore, curl to https should work fine.

*curl https://gitlab.openkube.io*



### Installing gitlab runner

GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline.
Installation is pretty staright-forward, you can follow the below official url. we wil install gitlab verson 13.7 from the below official doc

>https://docs.gitlab.com/13.7/runner/install/linux-manually.html

```
sudo curl -L --output /usr/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

sudo chmod +x /usr/bin/gitlab-runner

sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner

sudo gitlab-runner start
```

Once the gitlab-runner is started we can check the status of the services using the below gitlab-runner command

*gitlab-runner status*

gitlab-runner service can be monitored with systemctl

*sudo systectl staus gitlab-runner*

### register your runner

Next is to register your runner to run gitlab CI. Below is the interactive one. 
Copy the token from Gitlab UI -> Admin Area -> Overview -> Runners

we can use gitlab-runner register command only without attaching ca file if you have already added the gitlab-ca.crt to the server truststore

*gitlab-runner register --tls-ca-file gitlab-ca.crt*  

Below is non interactive command to register a docker based gilab runner

*gitlab-runner register -n --url https://gitlab.openkube.io --registration-token runner_token --executor docker --description "Deployment Runner" --docker-image "docker:stable" --tag-list deployment --docker-privileged   --tls-ca-file gitlab-ca.crt* 

Now you are ready to create git-lab yaml files and proceed with your CI/CD to the target platform.


### Uninstall gitlab

**step to uninstall gitlab services and clean data**

*gitlab-ctl uninstall*

*gitlab-ctl cleanse*

**step uninstall gitlab ee**

*apt-get remove --auto-remove gitlab-ee*

**command to purge any data related the previos installation**

*sudo apt-get purge --auto-remove gitlab-ee*
