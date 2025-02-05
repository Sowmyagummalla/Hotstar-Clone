pipeline {
    agent any
    tools {
     jdk 'java17'
     nodejs 'node16'    
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout GIT') {
            steps {
                git branch: 'main', url: 'https://github.com/Sowmyagummalla/Hotstar-Clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Hotstar \
                    -Dsonar.projectKey=Hotstar'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage("Install Dependencies"){
           steps {
                sh 'npm install'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Scout FS') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'Docker', toolName: 'docker'){
                       sh 'docker-scout quickview fs://.'
                       sh 'docker-scout cves fs://.'
                   }
                }   
            }
        }
        stage("Docker Build &tag; Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'Docker', toolName: 'docker'){
                       sh "docker build -t hotstar ."
                       sh "docker tag hotstar sowmyagummalla/hotstar:latest "
                       sh "docker push sowmyagummalla/hotstar:latest"
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'Docker', toolName: 'docker'){
                       sh 'docker-scout quickview sowmyagummalla/hotstar:latest'
                       sh 'docker-scout cves sowmyagummalla/hotstar:latest'
                       sh 'docker-scout recommendations sowmyagummalla/hotstar:latest'
                   }
                }   
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('K8S') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }   
                    }
                }
            }
        }
    }
}
