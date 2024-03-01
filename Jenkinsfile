pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCESS_KEY_ID = credentials("AWS_ACCESS_KEY_ID")
        AWS_SECRET_ACCESS_KEY = credentials("AWS_SECERT_KEY")
        AWS_DEFAULT_REGION="us-east-1"
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Chethu28/Netflix-clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=netflix \
                    -Dsonar.projectKey=netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
     
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=6852c667fd0789eab6f5d575e459a6b5 -t chethanreddy28/netflix-clone:${BUILD_NUMBER} ."
                       sh "docker push  chethanreddy28/netflix-clone:${BUILD_NUMBER} "
                    }
                }
            }
        }
        
        stage("TRIVY"){
            steps{
                sh "trivy image chethanreddy28/netflix-clone:${BUILD_NUMBER} > trivyimage.txt" 
            }
        }
        /*stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 chethanreddy28/netflix-clone:${BUILD_NUMBER}'
            }
        }*/
        
        stage('Terraform Operations') {
            steps {
                script {
                // Git checkout
                    sh 'mkdir terraform'
                    sh 'cd terraform/'
                    sh 'git clone https://github.com/Chethu28/Terraform.git'
                dir('Terraform/EKS') {
                // Terraform init
                    sh 'terraform init'
                
                // Terraform format and validate
                    sh 'terraform fmt'
                    sh 'terraform validate'
                
                // Terraform plan
                    sh 'terraform plan'
                    input message: 'Are you sure, can we deploy the EKS cluster?', ok: 'proceed'
                
                // Ask for apply or destroy
                    def choice = input message: 'Do you want to apply or destroy the infrastructure?', parameters: [choice(name: 'Action', choices: ['apply', 'destroy'], description: 'Choose to apply or destroy')]
                
                // Terraform apply or destroy based on choice
                    sh "terraform ${choice} --auto-approve"
            }
                }
    }
}

}
}
