### Zero Trust Principles 
1. Identity and Access management
1. Secrets Management: Jenkins Credentials or HashiCorp vault
2. Code Integrity: SonarQube 
3. Network Security:
4. Monitoring: ELK stack

### Aspects of zero trust security
**Strict Access controls** 
	Least privilege, only authorized users and systems to interact with specific parts of the pipeline
	Role based access control

**Continuous Verification**
	Verify identity and integrity of users and systems, and  code throughout the pipeline. 
	Using MFA, code scanning, container image scanning

**Automated security checks**
	Integrate security scans into the CICD pipeline
	Static code analysis, dependency vulnerability scanning, infra configuration checks 

**Data Encryption**
	Encrypt sensitive data or all the data at rest and in transit within the pipeline to protect against unauthorized despite breach occurrence
	Tokenization 
**Monitoring and logging**
	Real time logging and log analysis to report anomalies or suspicious behaviors

### Tools 
1. Pipeline: Jenkins
2. Secret scanning: Truffle Hog or Gitleaks
3. Secret management: Hashicorp
4. Sofware Composition analysis: OWASP Dependency-Check
5. SAST: SonarQube
6. Container Image scanning: DockerBench
7. DAST: OWASP Zap
8. Security Monitoring + incident reporting: ELK 
9. Vuln Assessment : trivy (container and IaC scan) Checkov (IaC security and compliance scan)
10. Vuln Management: Defect Dojo 



## Overview 
1. Code pushed to repo - Git
2. Check Secrets in code - TruffleHog 
3. SAST - SonarQube
4. SCA - OWASP Dependency-Check
5. Build and image creation - Docker 
6. Container image scan - DockerBench
7. DAST - OWASP ZAP
8. IAC Scan - Trivy
9. Vuln Reporting & management - DefectDojo
10. Security Monitoring  & incident reporting - ELK

#### HashiCorp Vault

Secrets - Info used to login or authorize various services
Username Password, API key
Human Users or System users
E.g. 











Open source secrets management tool
Automates access to secrets, data or systems
Can be integrated with CICD, Terraform, K8s and Ansible 

Secret storage
Employee credential storage
API key generation  for scripts
Data encryption

Dynamic secrets





