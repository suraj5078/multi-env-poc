pipeline {
    agent any
    
    tools {
        maven 'Maven3'
    }
    
    environment {
        registryDev = "${env.REGISTRY_DEV}"
        registryQa = "${env.REGISTRY_QA}"
        registryUat = "${env.REGISTRY_UAT}"
        registryProd = "${env.REGISTRY_PROD}"
    }

    stages {
        stage('Cloning Git') {
            steps {
                // checkout([$class: 'GitSCM', branches: [[name: '*/dev'], [name: '*/qa'], [name: '*/uat'], [name: '*/prod']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/ayushimishra2601/docker-spring-boot']]])
            
                checkout scmGit(branches: [[name: '**']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/suraj5078/multi-env-poc.git']])
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        // Building Docker images
        // stage('Building image') {
        //     steps {
        //         script {
        //             dockerImage = docker.build registry
        //             dockerImage.tag("$BUILD_NUMBER")
        //         }
        //     }
        // }

        stage('Pushing to ECR') {
            steps {
                script {
                    def branchName = env.GIT_BRANCH
                    // def branchName = env.BRANCH_NAME
                    echo "Branch Name: ${branchName}"
                    def registry

                    switch (branchName) {
                        case 'dev':
                            registry = registryDev
                            break
                        case 'qa':
                            registry = registryQa
                            break
                        case 'uat':
                            registry = registryUat
                            break
                        case 'prod':
                            registry = registryProd
                            break
                        default:
                            echo "Invalid branch"
                            return
                    }
                    dockerImage = docker.build registry
                    dockerImage.tag("$BUILD_NUMBER")
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${registry}"
                    sh "docker tag ${dockerImage.id} ${registry}:${BUILD_NUMBER}"
                    sh "docker push ${registry}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Helm Deploy') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    def namespace

                    switch (branchName) {
                        case 'dev':
                            namespace = 'dev-namespace'
                            break
                        case 'qa':
                            namespace = 'qa-namespace'
                            break
                        case 'uat':
                            namespace = 'uat-namespace'
                            break
                        case 'prod':
                            namespace = 'prod-namespace'
                            break
                        default:
                            echo "Invalid branch"
                            return
                    }

                    withKubeConfig([credentialsId: 'POC-TEST-EKS', serverUrl: '']) {
                        sh '''
                            helm upgrade first --install mychart --namespace ${namespace} --set image.repository=${registry}:${BUILD_NUMBER}
                            kubectl get all -n ${namespace}
                            helm ls -n ${namespace}
                            kubectl get pods -n ${namespace}
                            kubectl get services -n ${namespace}
                            helm list -n ${namespace}
                        '''
                    }
                }
            }
        }
    }
}
