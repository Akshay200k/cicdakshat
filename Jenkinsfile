pipeline {
    agent any
    stages {
        stage('code checkout') {
            steps {
                git url: 'https://github.com/Akshay200k/cicdakshat.git', branch: 'master'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-cred', installationName: 'sonar') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=abcd"
                }
            }
        }
        
      stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        
        stage('build package') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('upload artifact') {
            steps {
                dir('/var/lib/jenkins/workspace/cicd/target') {
                   nexusArtifactUploader artifacts: [[artifactId: 'cicd-artifact-id', classifier: '', file: 'devops-integration.jar', type: 'jar']], credentialsId: 'nexus-cred', groupId: 'com.truelearning', nexusUrl: '13.233.131.177:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'cicd-repo', version: '0.0.1-SNAPSHOT'
                }
            }
        }
        
        stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t akshayk170/cicd:v7 .'
                }
            }
        }
        
        stage('Docker login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-pwd', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push akshayk170/cicd:v7'
                }
            }
        }
        
        stage('Deploy to k8s'){
            when{ expression {env.GIT_BRANCH == 'origin/master'}}
            steps{
                script{
                     kubernetesDeploy (configs: 'deploymentservice.yaml' ,kubeconfigId: 'k8sconfigpwd')
                   
                }
            }
        }
    }
}
