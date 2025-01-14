pipeline {
    agent any
    
    tools {
        maven 'Maven3'
    }
    
    environment {
        registry = ""
        namespace = ""
        registryDev = "${env.REGISTRY_DEV}"
        registryQa = "${env.REGISTRY_QA}"
        registryUat = "${env.REGISTRY_UAT}"
        registryProd = "${env.REGISTRY_PROD}"
        namespaceDev = "${env.NAMESPACE_DEV}"
        namespaceQa = "${env.NAMESPACE_QA}"
        namespaceUat = "${env.NAMESPACE_UAT}"
        namespaceProd = "${env.NAMESPACE_PROD}"
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

        stage('Build Docker image and Pushing to ECR') {
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
                    env.finalregistry = "${registry}"
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${registry}"
                    sh "docker tag ${dockerImage.id} ${registry}:${BUILD_NUMBER}"
                    sh "docker tag ${dockerImage.id} ${registry}:latest"
                    sh "docker push ${registry}:${BUILD_NUMBER}"
                    sh "docker push ${registry}:latest"
                }
            }
        }

        stage('Helm Deploy') {
    steps {
        script {
            def branchName = env.GIT_BRANCH
            echo "Branch Name: ${branchName}"
            def namespace

            switch (branchName) {
                case 'dev':
                    namespace = namespaceDev
                    break
                case 'qa':
                    namespace = namespaceQa
                    break
                case 'uat':
                    namespace = namespaceUat
                    break
                case 'prod':
                    namespace = namespaceProd
                    break
                default:
                    echo "Invalid branch"
                    return
            }

//             withKubeConfig([credentialsId: 'POC-TEST-EKS', serverUrl: '']) {
                echo "Environment namespace: ${namespace}"
                echo "Environment registry: ${finalregistry}"
                echo "Registry URL is: ${finalregistry}:${BUILD_NUMBER}"
                sh "kubectl config get-contexts"
                sh "aws eks update-kubeconfig --name POC-TEST --region us-east-1"
                sh "kubectl get all -n ${namespace}"
                sh "kubectl get all"
                sh "kubectl get ns"
                sh "helm upgrade first --install mychart --namespace ${namespace} --set image.repository=${finalregistry} --set image.tag=latest"
 //               sh "helm upgrade first --install mychart --namespace ${namespace} --set image.repository=${finalregistry} --set image.tag=latest"
                echo "Sleeping for 1 minute..."
                // Sleep for 60 seconds (1 minute)
                sleep(time: 60, unit: 'SECONDS')
                echo "Finished sleeping."
//                 export SERVICE_IP=$(kubectl get svc --namespace ${namespace} first-mychart --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
//                 echo http://$SERVICE_IP:80
                sh "kubectl get all -n ${namespace}"
                sh "helm ls -n ${namespace}"
                sh "kubectl get pods -n ${namespace}"
                sh "kubectl get services -n ${namespace}"
                sh "helm list -n ${namespace}"
 //           }
          }
       }
     }
   }
}
