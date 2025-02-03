```groovy
pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
        VAULT_ADDR = "http://127.0.0.1:8200"  // Vault Server Address
        ROLE_ID = credentials("vault-role-id")  // Jenkins Credential: Vault Role ID
        SECRET_ID = credentials("vault-secret-id")  // Jenkins Credential: Vault Secret ID
    }
    stages {
        stage("Authenticate with Vault") {
            steps {
                script {
                    def response = sh(
                        script: """curl -s --request POST --data '{ "role_id": "'$ROLE_ID'", "secret_id": "'$SECRET_ID'" }' $VAULT_ADDR/v1/auth/approle/login""",
                        returnStdout: true
                    ).trim()
                    def authData = readJSON(text: response)
                    env.VAULT_TOKEN = authData.auth.client_token
                }
            }
        }
        stage("Retrieve SonarQube Token from Vault") {
            steps {
                script {
                    def secretResponse = sh(
                        script: """curl -s --header "X-Vault-Token: $VAULT_TOKEN" $VAULT_ADDR/v1/secret/data/sonarqube""",
                        returnStdout: true
                    ).trim()
                    def secretData = readJSON(text: secretResponse)
                    env.SONAR_TOKEN = secretData.data.data.token
                }
            }
        }
        stage("Clone Code from GitHub") {
            steps {
                git url: "https://github.com/krishnaacharyaa/wanderlust.git", branch: "devops"
            }
        }
        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust -Dsonar.login=$SONAR_TOKEN"
                }
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                sh "npm install"
                dependencyCheck additionalArguments: "--scan ./", odcInstallation: 'OWASPDC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
    }
}
```
