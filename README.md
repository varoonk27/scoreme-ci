# README: CI/CD Pipeline with Jenkins, SonarQube, JaCoCo, Cyclomatic Complexity, and OWASP Dependency-Check

This README describes the steps to set up a Continuous Integration/Continuous Deployment (CI/CD) pipeline using Jenkins, SonarQube, JaCoCo, Lizard, OWASP Dependency-Check, and email notifications. The pipeline uses a Spring Boot application hosted on GitHub and integrates with various tools to ensure code quality, coverage, cyclomatic complexity, and security checks.

## Table of Contents
- [Objective](#objective)
- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Setup and Configuration Steps](#setup-and-configuration-steps)
  - [1. Git Repository Configuration](#1-git-repository-configuration)
  - [2. EC2 Instance Setup for Jenkins](#2-ec2-instance-setup-for-jenkins)
  - [3. Jenkins Pipeline Setup](#3-jenkins-pipeline-setup)
  - [4. SonarQube Setup and Integration](#4-sonarqube-setup-and-integration)
  - [5. Code Coverage with JaCoCo](#5-code-coverage-with-jacoco)
  - [6. Cyclomatic Complexity with Lizard](#6-cyclomatic-complexity-with-lizard)
  - [7. Security Vulnerability Scan with OWASP Dependency-Check](#7-security-vulnerability-scan-with-owasp-dependency-check)
  - [8. Jenkins Email Notifications](#8-jenkins-email-notifications)
- [Jenkinsfile Configuration](#jenkinsfile-configuration)
- [Conclusion](#conclusion)

---

## Objective

The goal of this pipeline is to automate the following tasks:
1. Checkout code from GitHub repository.
2. Build the project using Maven.
3. Run code quality analysis using SonarQube.
4. Measure code coverage using JaCoCo.
5. Calculate cyclomatic complexity using Lizard.
6. Run a security vulnerability scan using OWASP Dependency-Check.
7. Send email notifications based on build status.

---

## Prerequisites

- **Jenkins** installed on an EC2 instance (Ubuntu).
- **SonarQube** installed on a separate EC2 instance.
- **GitHub** repository with a Spring Boot application (`https://github.com/varoonk27/scoreme-ci.git`).
- Required Jenkins plugins: Git, SonarQube Scanner, JaCoCo, OWASP Dependency-Check, and Email Extension.

---

## Architecture Overview

- **Jenkins EC2 Instance**: Used to run Jenkins jobs, build the project, and trigger other analysis tools.
- **SonarQube EC2 Instance**: Used to analyze code quality.
- **GitHub**: Stores the source code of the Spring Boot application.
- **Email**: Sends notifications regarding build success or failure.

---

## Setup and Configuration Steps

### 1. Git Repository Configuration

1. Create a Spring Boot application in your GitHub repository (`https://github.com/varoonk27/scoreme-ci.git`).
2. Ensure your project has a `pom.xml` for Maven dependencies.
3. Push the application code and `pom.xml` to the repository:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/varoonk27/scoreme-ci.git
   git push origin main
   ```

---

### 2. EC2 Instance Setup for Jenkins

1. Launch an EC2 instance (Ubuntu) on AWS.
2. Install Java and Jenkins:
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk
   sudo apt install jenkins
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

3. Access Jenkins through `http://<EC2-Public-IP>:8080` and install the required plugins:
   - Git Plugin
   - Pipeline Plugin
   - SonarQube Scanner
   - JaCoCo Plugin
   - OWASP Dependency-Check Plugin
   - Email Extension Plugin

---

### 3. Jenkins Pipeline Setup

1. Create a new Jenkins job of type **Pipeline**.
2. Configure the job to pull code from your GitHub repository.
3. Add a `Jenkinsfile` to the repository to define the pipeline stages (see the [Jenkinsfile Configuration](#jenkinsfile-configuration) section).

---

### 4. SonarQube Setup and Integration

1. Launch a new EC2 instance and install SonarQube:
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk
   wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.0.1.zip
   unzip sonarqube-9.0.1.zip
   cd sonarqube-9.0.1/bin/linux-x86-64
   ./sonar.sh start
   ```

2. Open SonarQube on `http://<SonarQube-EC2-Public-IP>:9000`.
3. In Jenkins, configure **SonarQube** under **Manage Jenkins > Configure System** by adding the SonarQube server and token.
4. Integrate SonarQube analysis into the Jenkins pipeline.

---

### 5. Code Coverage with JaCoCo

1. Add JaCoCo to the `pom.xml` file in your Spring Boot project.
2. In Jenkins, configure the pipeline to run JaCoCo and publish the report:
   ```groovy
   stage('Code Coverage - JaCoCo') {
       steps {
           sh 'mvn jacoco:report'
           publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, 
                        reportDir: 'target/site/jacoco', reportFiles: 'index.html', 
                        reportName: 'JaCoCo Code Coverage Report'])
       }
   }
   ```

---

### 6. Cyclomatic Complexity with Lizard

1. Install Lizard on the Jenkins EC2 instance:
   Facing error while going through this process, for completion of next step, I skipped this particular process.
   ```groovy
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
---
 
### 7. Security Vulnerability Scan with OWASP Dependency-Check

1. Install the **OWASP Dependency-Check Plugin** in Jenkins.
2. Add a stage to run OWASP Dependency-Check in the pipeline:
   ```groovy
   stage('OWASP Dependency Check') {
       steps {
           dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
           dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
       }
   }
   ```

---

### 8. Jenkins Email Notifications

1. Install and configure the **Email Extension Plugin** in Jenkins.
2. Add email notifications in the `post` block of your pipeline:
   ```groovy
   post {
       success {
           emailext subject: 'Build Successful: ${JOB_NAME} #${BUILD_NUMBER}',
                    body: 'Good news! The build succeeded! Check the details at ${BUILD_URL}',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
       }
       failure {
           emailext subject: 'Build Failed: ${JOB_NAME} #${BUILD_NUMBER}',
                    body: 'Unfortunately, the build failed. Check the details at ${BUILD_URL}',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
       }
   }
   ```

---

## Jenkinsfile Configuration

Below is the full **Jenkinsfile** that integrates all stages:

```groovy
pipeline {
    agent any
    
    tools {
        maven 'mvn'
    }
    
    environment {
        SONARQUBE_SERVER = 'SonarQubeServer'
        JACOCO_REPORT_PATH = '**/target/site/jacoco/jacoco.xml'
        LIZARD_REPORT_PATH = 'lizard-report.txt'
        OWASP_REPORT_PATH = 'dependency-check-report.html'
        BRANCH_NAME = 'main'
    }

    triggers {
        pollSCM('H/5 * * * *')
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
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Code Coverage - JaCoCo') {
            steps {
                sh 'mvn jacoco:report'
                publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, 
                             keepAll: true, reportDir: 'target/site/jacoco', 
                             reportFiles: 'index.html', reportName: 'JaCoCo Code Coverage Report'])
            }
        }

        stage('Cyclomatic Complexity Analysis') {
            steps {
                sh 'lizard . > lizard-report.txt'
                archiveArtifacts artifacts: 'lizard-report.txt', allowEmptyArchive: true
            }
        }

         stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'DP'
                   dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
         }

    post {
        always {
            junit '**/target/surefire-reports/*.xml' // Publish test results
           archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            cleanWs() // Clean up workspace after the build
        }

        
        success {
            
            emailext(
            to: 'varoonk208@gmail.com', 
            subject: 'Build Successful: ${JOB_NAME} #${BUILD_NUMBER}',
            body: 'Good news! The build succeeded! Check the details at ${BUILD_URL}',
            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )    
         }

         failure {
            emailext(
            to: 'varoonk208@gmail.com', subject: 'Build Failed: ${JOB_NAME} #${BUILD_NUMBER}',
                      body: 'Unfortunately, the build failed. Check the details at ${BUILD_URL}',
                      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
         }
    }
}
