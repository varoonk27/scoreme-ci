pipeline {
    agent any
    
    tools {
        maven 'mvn'
    }
    
    environment {
        SONARQUBE_SERVER = 'SonarQubeServer' // Define your SonarQube server name from Jenkins settings
        JACOCO_REPORT_PATH = '**/target/site/jacoco/jacoco.xml'
        LIZARD_REPORT_PATH = 'lizard-report.txt'
        OWASP_REPORT_PATH = 'dependency-check-report.html'
        BRANCH_NAME = 'main' // Set the branch name to trigger the pipeline
    }

    triggers {
        pollSCM('H/5 * * * *') // Polling for changes every 5 minutes (can be adjusted)
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        checkout scm
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Run Maven build command
                    sh 'mvn clean install'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Code Coverage - JaCoCo') {
            steps {
                script {
                    sh 'mvn jacoco:report'
                    publishHTML([allowMissing: true,
                                 alwaysLinkToLastBuild: true,
                                 keepAll: true,
                                 reportDir: 'target/site/jacoco',
                                 reportFiles: 'index.html',
                                 reportName: 'JaCoCo Code Coverage Report'
                                ])
                }
            }
        }

       //  stage('Install pip') {
       //     steps {
       //         script {
       //              sh 'sudo apt-get install -y python-pip'
       //         }
       //     }
       // }


       //  stage('Setup Virtual Environment') {
       //      steps {
       //          script {
       //              // Create a virtual environment and install lizard
       //              sh '''
       //                  python3 -m venv venv
       //                  source venv/bin/activate
       //                  pip install --upgrade pip
       //                  pip install lizard
       //              '''
       //          }
       //      }
       //  }

       //  stage('Cyclomatic Complexity Analysis') {
       //      steps {
       //          script {
       //              sh 'lizard . > lizard-report.txt' // Run Lizard to calculate cyclomatic complexity
       //              archiveArtifacts artifacts: 'lizard-report.txt', allowEmptyArchive: true
       //          }
       //      }
       //  }

        // stage('Security Vulnerability Scan - OWASP Dependency Check') {
        //     steps {
        //         script {
        //             sh 'mvn org.owasp:dependency-check-maven:check'
        //             publishHTML([allowMissing: true,
        //                          alwaysLinkToLastBuild: true,
        //                          keepAll: true,
        //                          reportDir: 'target',
        //                          reportFiles: 'dependency-check-report.html',
        //                          reportName: 'OWASP Dependency-Check Report'
        //                         ])
        //         }
        //     }
        // }
        /*stage('OWASP Dependency-Check Vulnerabilities') {
           steps {
             dependencyCheck additionalArguments: ''' 
                         -o './'
                         -s './'
                         -f 'ALL' 
                         --prettyPrint''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
            
             dependencyCheckPublisher pattern: 'dependency-check-report.xml'
           }
         }*/
         stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'DP'
                   dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
         }
        // stage('Quality Gate Check') {
        //     steps {
        //         script {
        //             timeout(time: 5, unit: 'MINUTES') {
        //                 def qualityGate = waitForQualityGate()
        //                 if (qualityGate.status != 'OK') {
        //                     error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
        //                 }
        //             }
        //         }
        //     }
        // }

    }

    post {
        always {
            junit '**/target/surefire-reports/*.xml' // Publish test results
           archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            cleanWs() // Clean up workspace after the build
        }

        
        success {
             //emailext subject: 'Build Successful: ${JOB_NAME} #${BUILD_NUMBER}',
             //         body: 'Good news! The build succeeded! Check the details at ${BUILD_URL}',
             //         recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            to: 'varoonk208@gmail.com', // Add recipient email address
            subject: 'Build Successful: ${JOB_NAME} #${BUILD_NUMBER}',
            body: 'Good news! The build succeeded! Check the details at ${BUILD_URL}',
            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
         }

         failure {
            emailext subject: 'Build Failed: ${JOB_NAME} #${BUILD_NUMBER}',
                      body: 'Unfortunately, the build failed. Check the details at ${BUILD_URL}',
                      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
         }
    }
}
   



