pipeline {
    agent any
    
    environment {
            imagename = "muzafferjoya/two-tier"
            registryCredential = 'muzaffar-hub-id'
            dockerImage = ''
            SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        stage('Clean WS'){
            steps{
                cleanWs()
            }
        }
        stage('SCM'){
            steps{
                git branch: 'main', url: 'https://github.com/muzafferjoya/two-tier-jenkins.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                dir('application-code/frontend') {
                    withSonarQubeEnv('sonar-scanner') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=two-tier-frontend \
                        -Dsonar.projectKey=two-tier-frontend '''
                    }
                }
            }
        }
        // stage('OWASP Dependency-Check Scan') {
        //     steps {
        //         dir('application-code/frontend') {
        //             dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //             dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //         }
        //     }
        // }
        // stage('Trivy File Scan') {
        //     steps {
        //         dir('application-code/frontend') {
        //             sh "trivy fs --format table -o trivy-fs-report.html ."
        //         }
        //     }
        // }
        stage('Building image') {
          steps{
              dir('application-code/frontend'){
                script {
                  dockerImage = docker.build imagename
                }
              }
          }
        }
        stage('Push Image') {
          steps{
            script {
              docker.withRegistry( '', registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                 dockerImage.push('latest')
    
              }
            }
          }
        }
        stage('Image Scan'){
            steps{
                sh 'trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o trivy-image-report.html --severity HIGH,CRITICAL ${imagename}:latest'
                //sh "trivy image --format table -o trivy-image-report.html ${imagename}:latest"
            
                }
        }
        
    }
    post {
        failure {
            emailext attachLog: true, attachmentsPattern: 'trivy-image-report.html', body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                    mimeType: 'text/html',to: "muzafferjoya@gmail.com"
            }
         success {
               emailext attachLog: true, attachmentsPattern: 'trivy-image-report.html', body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                    mimeType: 'text/html',to: "muzafferjoya@gmail.com"
          }      
    }
}
