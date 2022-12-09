pipeline{
    agent any
    
    stages {
        stage("Checkout") {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/fayizv/pyton-flask-chart.git']]])
            }
        }  
        
        
    //    stage('Build Maven'){
    //         steps{
    //             sh 'mvn clean package'
    //         }
    //      }
        
        
//       stage('build && SonarQube analysis') {
//             steps {
//                 withSonarQubeEnv('sonarqube-9.2.2') {
//                     // Optionally use a Maven environment you've configured already
                   
//                          sh 'sonar.python.version=2.7.18 app.py sonar:sonar'
                      
                        
                
                    
//                 }
//             }
//         }
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
        
//         stage('Romove image') {
//             steps {
//                 script {
//                     sh 'docker rmi -f fayizv/flask:1.1.1'
//                 }
//             }
//         }

        
        stage('logout docker') {
            steps {
                script {
                    sh 'docker logout'
                }
            }
        }
        
        
        stage('AWS ECR login') {
            steps {
                script {
                   
                        sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/z1n7h1z8'
   
                  
                    

                }
            }
        }
        
        
        stage('AWS ECR push') {
            steps {
                script {
                    sh 'docker tag fayizv/flask:latest public.ecr.aws/z1n7h1z8/flask:${BUILD_NUMBER}'
                    sh 'docker push public.ecr.aws/z1n7h1z8/flask:${BUILD_NUMBER}'

                }
            }
        }
        stage('Helm Push to ECR') {
            steps {
                sh 'echo version : 0.${BUILD_NUMBER}.0 >> flaskchart/Chart.yaml'
                sh 'helm package flaskchart'
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 707032823801.dkr.ecr.us-east-1.amazonaws.com'
                sh 'helm push flaskchart- 0.${BUILD_NUMBER}.0  707032823801.dkr.ecr.us-east-1.amazonaws.com/'
                sh 'rm -rf flaskchart-*'
                }
        }
        stage('Invoke Build number to Pipeline Deployjob') {
            steps {
                build job: 'Deployjob', parameters : [[ $class: 'StringParameterValue', name: 'buildnum', value: "${BUILD_NUMBER}"]]
            }
        }



//         stage("Quality gate") {
//             steps {
//                 waitForQualityGate abortPipeline: true
//             }
//         }
       
    }
}


