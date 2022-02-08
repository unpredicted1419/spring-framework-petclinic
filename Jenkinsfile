pipeline {
    agent any
    environment {
        git_repo_url = "https://github.com/crazy4devops/spring-framework-petclinic.git"
        git_repo_br = "dev"
    }
    stages{
        // stage("Checkout"){
        //     steps {
        //         // Clean before build
        //         cleanWs()
        //         git url: "${git_repo_url}", branch: "${git_repo_br}"
        //         echo 'Pulling...' + env.BRANCH_NAME
        //     }
        // }
        stage("Build Source Code"){
            steps {
                    //sh "./mvnw package"
                    sh "./mvnw package"
                    sh "mv target/petclinic*.war target/petclinicApp-${BUILD_NUMBER}.war"
            }
        }
        // stage("Run Unit-Tests"){
        //     steps {
        //             sh "./mvnw test"
        //     }
        // }
        stage('Sonarqube Analysis') {
            environment {
              def scannerHome = tool 'SonarQubeScanner'
            }
            steps {  
                withSonarQubeEnv('sonarserver') {
                    sh "/opt/sonar/bin/sonar-scanner"
                }
                sleep time: 30000, unit: 'MILLISECONDS'
                script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                }
            }
        }
        stage("Upload Artifacts"){
            steps{
                rtUpload (
                    serverId: 'jfrog-server',
                    spec: '''{
                        "files": [
                            {
                            "pattern": "target/*.war",
                            "target": "dummyrepo/petclinicApp/"
                            }
                        ]
                    }''',
                )
                    
            }
        }            
        // stage("Deploy - DEV"){
        //     steps{
        //         echo "Running Deployment on Dev"
        //         sh """
        //           sshpass -p 'Venkat@123' scp -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no target/*.war cloud_user@172.31.46.115:/opt/tomcat/webapps
        //         """
        //     }
        // }
        stage("deploy-aws-dev"){
            steps{
                sshagent(['aws-ec2-creds']) {
                        sh """
                            scp -o StrictHostKeyChecking=no target/*.war   ubuntu@ec2-65-0-95-227.ap-south-1.compute.amazonaws.com:/opt/tomcat/webapps/
                        """
                }
            }
        }
        stage("deploy-aws-uat"){
            steps{
                sshagent(['aws-ec2-creds']) {
                        sh """
                            scp -o StrictHostKeyChecking=no target/*.war   ubuntu@ec2-13-233-97-79.ap-south-1.compute.amazonaws.com:/opt/tomcat/webapps/
                        """
                }
            }
        }
        stage("deploy-aws-prd"){
            steps{
                sshagent(['aws-ec2-creds']) {
                        sh """
                            scp -o StrictHostKeyChecking=no target/*.war   ubuntu@ec2-3-109-47-34.ap-south-1.compute.amazonaws.com:/opt/tomcat/webapps/
                        """
                }
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}
