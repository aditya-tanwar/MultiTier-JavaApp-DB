pipeline {
    agent any

    options {
        skipDefaultCheckout true
    }

    // START OF THE STAGES
    stages {

        stage('Cleaning Workspace') {
            steps{
                cleanWs() // jenkins inbuilt function to clean the workspace
            }
        }


        stage('Checkout') {
            steps {
                git credentialsId: 'git', poll: false, url: 'https://github.com/aditya-tanwar/MultiTier-JavaApp-DB.git'
                sh 'mkdir Test-Results'
            }
        }


        // stage('Trivy Filesystem Scan') {
        //     steps {
        //         sh 'ls && pwd'
        //         sh 'trivy fs . --format json'
        //     }
        // }


        // stage('SonarQube Code Analysis') {
        //     environment {
        //         scannerHome = tool 'sonar'
        //     }
        //     steps {
        //         scritp {
        //             withSonarQubeEnv('sonar') {
        //                 sh '''
        //                     ${scannerHome}/bin/sonar-scanner \
        //                     -Dsonar.projectKey=MultiTier-JavaApp-DB \
        //                     -Ssonar.projectName=MultiTier-JavaApp-DB \
        //                     -Dsonar.java.binaries=.
        //                 '''
        //             }
        //         }

        //     }

        // }


        stage('Quality Gate'){
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
            }
        }


        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
               }
        }


        stage('Docker Build') {
            steps {
                sh 'docker build -t multitier-javaapp-db:v1 .'
            }
        }

        stage('Scanning Docker image') {
            steps {
                sh 'trivy image --scanners vuln,misconfig,secret multitier-javaapp-db:v1 --format json -o Test-Results/trivy-image-results.json'
                sh 'dockle -f json multitier-javaapp-db:v1 > Test-Results/dockle-image-results.json'
            }
        }


        stage('Docker Push') {
            steps {
                sh 'docker tag multitier-javaapp-db:v1 aditya-tanwar/multitier-javaapp-db:v1'
                sh 'docker push aditya-tanwar/multitier-javaapp-db:v1'
            }
        }


    } // END OF  THE STAGES
    
}