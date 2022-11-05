pipeline {
    agent any 
    tools {
        maven "maven"
    }
    
    stages {
        stage('checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/akannan1087/myMar2022WeekdayBatchRepo.git']]])
            }
        }
        
        stage('build') {
            steps {
                sh 'mvn clean install -f MyWebApp/pom.xml'
            }
        }
        
        stage('quality') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar -f MyWebApp/pom.xml'
                }
            }
        }
        
        stage('upload artifact') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: 'nexus-pass', groupId: 'com.dept.app', nexusUrl: '10.128.0.55:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage('deploy to stage') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcat-pass', path: '', url: 'http://10.128.0.60:8080')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage('slack message for stage') {
            steps {
                slackSend channel: 'otter-channel', message: 'file deployed to stage successfully '
            }
        }
        
        stage('approval') {
            steps {
                timeout(time: 5, unit: 'HOURS') {
                    input message: 'want to deploy to prod', submitter: 'admin'
                }
            }
        }
        
        stage('prod') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcat-pass', path: '', url: 'http://10.162.0.2:8080')], contextPath: null, war: '**/*.war'
            }
        }
    }
}
