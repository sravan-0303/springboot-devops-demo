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
                url: 'https://github.com/sravan-0303/springboot-devops-demo.git'
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
                sh 'mvn deploy -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                POD=$(kubectl get pods -n jenkins -l app=tomcat -o jsonpath="{.items[0].metadata.name}")

                if [ -z "$POD" ]; then
                  echo "Tomcat pod not found"
                  exit 1
                fi

                kubectl cp -n jenkins target/demo-0.0.1-SNAPSHOT.war \
                $POD:/usr/local/tomcat/webapps/demo.war
                '''
            }
        }
    }
}
