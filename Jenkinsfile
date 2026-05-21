pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url:'https://github.com/sravan-0303/springboot-devops-demo.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=demo \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Upload WAR to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'nexus-service.default.svc.cluster.local:8081',
                    groupId: 'com.example',
                    version: '0.0.1',
                    repository: 'maven-releases',
                    credentialsId: 'nexus-creds',
                    artifacts: [
                        [
                            artifactId: 'demo',
                            classifier: '',
                            file: 'target/demo-0.0.1-SNAPSHOT.war',
                            type: 'war'
                        ]
                    ]
                )
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                POD=$(kubectl get pod -l app=tomcat -o jsonpath="{.items[0].metadata.name}")

                kubectl cp target/demo-0.0.1-SNAPSHOT.war default/$POD:/usr/local/tomcat/webapps/demo.war
                '''
            }
        }
    }
}
