pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }
    environment {
        packageVersion = ''
        nexusURL = '172.31.1.211:8081'
    }
    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    parameters {
        // string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        // text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

        booleanParam(name: 'Deploy', defaultValue: false, description: 'Toggle this value')

        // choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

        // password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }

    stages {
        stage('Setup Node.js') {
            steps {
                script {
                    // Check if Node.js is already installed
                    def nodeCheck = sh(script: 'command -v node || true', returnStdout: true).trim()
                    def npmCheck = sh(script: 'command -v npm || true', returnStdout: true).trim()
                    
                    if (!nodeCheck || !npmCheck) {
                        echo 'Installing Node.js and npm...'
                        // For CentOS/RHEL (assuming AGENT-1 is CentOS based on your path)
                        sh '''
                            curl -fsSL https://rpm.nodesource.com/setup_16.x | sudo bash -
                            sudo yum install -y nodejs
                            echo "Node.js version: $(node --version)"
                            echo "npm version: $(npm --version)"
                        '''
                    } else {
                        echo 'Node.js and npm are already installed'
                        sh 'node --version'
                        sh 'npm --version'
                    }
                }
            }
        }

        stage('Get the version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    packageVersion = packageJson.version
                    echo "application version: $packageVersion"
                }
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Unit tests') {
            steps {
                sh """
                    echo "unit tests will run here"
                """
            }
        }

        stage('Sonar scan') {
            steps{
                sh """
                    sonar-scanner
                """
            }
        }

        stage('Build') {
            steps {
                sh """
                    ls -la
                    zip -q -r catalogue.zip ./* -x ".git" -x "*.zip"
                    ls -ltr
                """
            }
        }

        stage('Publish Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusURL}",
                    groupId: 'com.roboshop',
                    version: "${packageVersion}",
                    repository: 'catalogue',
                    credentialsId: 'nexus-auth',
                    artifacts: [
                        [artifactId: 'catalogue',
                        classifier: '',
                        file: 'catalogue.zip',
                        type: 'zip']
                    ]
                )
            }
        }

        stage('Deploy') {
            when {
                expression {
                    params.Deploy == true  // Fixed: Changed = to == for comparison
                }         
            }
            steps {
                script {
                    def deployParams = [
                        string(name: 'version', value: "${packageVersion}"),
                        string(name: 'environment', value: 'dev')
                    ]
                    build job: "catalogue-deploy", wait: true, parameters: Params
                }
            }
        }
    }

    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure {
            echo 'this runs when pipeline is failed, used to send some alerts'
        }
        success {
            echo 'I will say Hello when pipeline is success'
        }
    }
}