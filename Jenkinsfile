pipeline{
    agent any
    
    stages {
        stage("Checkout") {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/fayizv/python-flask-chart.git']]])
            }
        }  
        
       stage("Build Docker Image") {
            steps {
                script {
                    sh 'docker build -t fayizv/flask .'
                    
                }
            }
        }
         
       stage('pushing to dockerhub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'dockerhubpwd')]) {
                    // some block
                    sh 'docker login -u fayizv -p ${dockerhubpwd}'
                    }

                    sh 'docker push fayizv/flask'
                }
            }
        }  
        
        stage('AWS ECR login') {
            steps {
                script {
                   
                        sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 707032823801.dkr.ecr.us-east-1.amazonaws.com'
   
                  
                    

                }
            }
        }
        
        
        stage('AWS ECR push') {
            steps {
                script {
                    sh 'docker tag fayizv/flask:latest 707032823801.dkr.ecr.us-east-1.amazonaws.com/flask-deploy:${BUILD_NUMBER}' 
                    sh 'docker push 707032823801.dkr.ecr.us-east-1.amazonaws.com/flask-deploy:${BUILD_NUMBER}'

                }
            }
        }
        stage('Helm Push to ECR') {
            steps {
                sh 'echo version : 0.${BUILD_NUMBER}.0 >> flaskchart/Chart.yaml'
                sh 'helm package flaskchart'
//                 sh 'tar cvzf flask-deploy.0.${BUILD_NUMBER}.0.tgz flaskchart '
                sh 'aws ecr get-login-password  --region us-east-1 | helm registry login --username AWS  --password-stdin 707032823801.dkr.ecr.us-east-1.amazonaws.com'
                sh 'helm push flaskchart-0.${BUILD_NUMBER}.0.tgz oci://707032823801.dkr.ecr.us-east-1.amazonaws.com/'
                sh 'rm -rf flaskchart-*'
                }
        }
        stage('Invoke Build number to Pipeline Deployjob') {
            steps {
                build job: 'DeployJob', parameters : [[ $class: 'StringParameterValue', name: 'buildnum', value: "${BUILD_NUMBER}"]]
            }
        }
       
    }
}


