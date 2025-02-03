
```
docker run -itd --name sonarqube-server -p 9000:9000 sonarqube:lts-community
```

ip:9000

admin admin

change password


Sonarqube communicates with Jenkins using Webhook.

Sonarqube running on docker container
Sonarqube qulity gates + Sonarqube plugin installed on jenkins

Sonarqube Admin token generated and stored in Jenkins credentials

That credential provided to sonarqube plugin - Mange jenkins -> System -> Sonarqube server 


Soanrqube Quality gates tool installation
Manage Jenkins -> Tools -> Sonar scanner installation -> Add -> Install Automatically


OWASP Dependency Check 
Manage Jenkins -> Tools -> Dependency Check -> Add -> name + install auto + install from github.com

OWASP has libraries and packages on github. Jenkins sends which packages are in project to this and OWASP sends report back.

trivy is locally installed on our jenkins machine
