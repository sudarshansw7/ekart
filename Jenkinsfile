pipeline
{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        DOCKER_USER = "sudarshansw7"
        DOCKER_PASS = 'docker-token'
        
    }
    stages
    {
        stage('Clean-workspace')
        {
            steps
            {
                cleanWs()
            }
        }
        stage('Git-Checkout')
        {
            steps
            {
                git 'https://github.com/sudarshansw7/ekart.git'
            }
        }
        stage('maven compile')
        {
            steps
            {
                sh 'mvn compile'
            }
        }
        stage('unit Tests')
        {
            steps
            {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('SonarQube Analysis')
        {
            steps
            {
                withSonarQubeEnv('sonar') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=ekart -Dsonar.projectName=ekart \
                   -Dsonar.java.binaries=. '''
              }
            }
        }
        stage('OWASP Dependency-Check')
        {
            steps
            {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('continous build')
        {
            steps
            {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('Deploy artifact to Nexus')
        {
            steps{
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('build image & tag')
        {
            steps
            {
                    sh 'docker build -t sudarshansw7/ekart:latest  -f docker/Dockerfile . '
            }
        }
        stage('Trivy Scan')
        {
            steps
            {
                sh 'trivy image sudarshansw7/ekart:latest'
            }
        }
        stage('docker push')
        {
            steps
            {
               script
               {
                    docker.withRegistry('',DOCKER_PASS) {
                      sh 'docker push sudarshansw7/ekart:latest'
                    }
               }
            }
        }
       
        
    }
}
