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
        sh '''
        curl -v -u admin:admin123 \
        --upload-file target/demo-0.0.1-SNAPSHOT.war \
        http://192.168.0.6:30081/repository/maven-releases/demo.war
        '''
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
