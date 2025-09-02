pipeline {
    agent any
    
    tools {
        jdk 'jdk-17'
        maven 'maven-3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_REGION = "ap-south-1"
        ECR_REPO = "971476708273.dkr.ecr.ap-south-1.amazonaws.com/workshop"
        CLUSTER_NAME = "workshop-cluster"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/apoorvag1207/Boardgame.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('File system scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=BoardGame \
                        -Dsonar.projectKey=BoardGame \
                        -Dsonar.java.binaries=.'''
                }
            }
        }
        // stage('Quality gates') {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        //         }
        //     }
        // }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk-17', maven: 'maven-3', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('Login to ecr & Push Docker Image') {
            steps {
                sh '''
                    # Login to ECR
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    
                    # Build Docker Image
                    docker build -t $ECR_REPO:latest .
                    
                    # Push Docker Image to ECR
                    docker push $ECR_REPO:latest
                '''
            }
        }
        // stage('Deploy to EKS') {
        //     steps {
        //         withKubeConfig([credentialsId: 'eks-kubeconfig']) {
        //             sh 'kubectl get pods -A'
        //         }
        //     }
        // }
        // stage('Deploy to eks'){
        //     steps{
        //         withCredentials([usernamePassword(credentialsId:'aws-eks-creds',
        //         usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        //         )]){
        //             sh '''
        //                 aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
        //                 kubectl apply -f deployment-service.yaml
        //             '''
        //         }
     

                    
        //     }
        // }
        // stage('verify deployment'){
        //     steps{
        //         sh '''
        //             kubectl get pods
        //             kubectl get svc
        //         '''
                    
        //     }
        // }
        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
                kubectl apply -f deployment-service.yaml -n app-namespace
            '''
        }
    }
    

        
        
    }
}
