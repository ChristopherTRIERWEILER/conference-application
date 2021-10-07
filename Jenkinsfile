pipeline {
    agent any

    tools {
        maven "Maven 3.6.3"
    }
    options {
        ansiColor('xterm')
    }
    stages {
        stage('Conference app') {
            steps {
                git 'https://github.com/ChristopherTRIERWEILER/conference-application.git'
                sh "mvn clean install package -Dmaven.test.skip=true"
                archiveArtifacts artifacts: '**/*.war', followSymlinks: false
            }
        }
        stage('Conference app sonar') {
            steps {
                withSonarQubeEnv('SonarQube') {
                sh "mvn  clean package sonar:sonar -Dmaven.test.skip=true -Dsonar.host_url=$SONAR_HOST_URL "
                }
            }
        }
        stage('Conference app Nexus') {
            steps {
                nexusPublisher nexusInstanceId: 'Nexus', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/conference-app-3.0.0.war']], mavenCoordinate: [artifactId: 'conference-project', groupId: 'de.conference-project', packaging: 'war', version: '1.0']]]
                }
        }
        stage('Conference app Docker build') {
            steps {
                sh " rm -Rf conference-app.war && \
                     wget http://nexus:8081/repository/maven-releases/de/conference-project/conference-project/1.0/conference-project-1.0.war -O ${WORKSPACE}/conference-app.war && \
                     docker build -t conference-app:latest ."
                }
        }
        stage ('Conference app docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh "docker login -u $USERNAME -p $PASSWORD && \
                    docker tag conference-app $USERNAME/conference-app && \
                    docker push $USERNAME/conference-app "
                }
            }
        }
        stage ('Conference app install docker') {
            steps {
                ansibleTower jobTemplate: 'Install docker', jobType: 'run', throwExceptionWhenFail: false, towerCredentialsId: '', towerLogLevel: 'false', towerServer: 'AWX'
            }
        }
        stage ('Conference app install app') {
            steps {
                ansibleTower jobTemplate: 'Install conference app', jobType: 'run', throwExceptionWhenFail: false, towerCredentialsId: '', towerLogLevel: 'false', towerServer: 'AWX'
            }
        }
    }
}