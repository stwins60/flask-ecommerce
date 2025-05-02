pipeline {
    
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "idrisniyi94/flask-ecom"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "lab-server-flask-ecom"
        DEV_NAMESPACE = "dev"
        PROD_NAMESPACE = "prod"
        BRANCH_NAME    = "${env.GIT_BRANCH.replace('refs/heads/', '')}"
        GIT_COMMIT_SHORT = "${env.GIT_COMMIT}" // gd389303037474832392ujsks // gd38930
    }


        stages {
            stage("Checkout") {
                steps {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${env.BRANCH_NAME}"]],
                        userRemoteConfigs: [[url: 'https://github.com/stwins60/flask-ecommerce.git']]
                    ])
                }
            }
            stage("Install Dependencies & Run Tests") {
                steps {
                    script {
                        sh 'pip install -r requirements.txt --break-system'
                        sh 'pytest --cov=. --cov-report=xml'
                    }
                }
            }
            stage("Sonarqube Scan") {
                steps {
                    withSonarQubeEnv('sonar-server') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=flask-ecom -Dsonar.projectName=flask-ecom -Dsonar.python.coverage.reportPaths=coverage.xml"
                    }
                }
            }
    
            stage("Build Docker Image") {
                steps {
                    script {
                        def tag = "${GIT_COMMIT_SHORT}-${BRANCH_NAME}"
                        sh "docker build -t ${IMAGE_NAME}:${tag} ."
                    }
                }
            }
            stage("Deploy to Kubernetes") {
                steps {
                    script {
                        kubeconfig(credentialsId: '2b8306e4-0b63-4b36-a84b-e6bf5b20e465', serverUrl: '') {
                            def tag = "${GIT_COMMIT_SHORT}-${BRANCH_NAME}"
                            def namespace = "${BRANCH_NAME == 'master' ? PROD_NAMESPACE : DEV_NAMESPACE}"
                            def deploymentEnvironment = "${BRANCH_NAME == 'master' ? 'prod' : 'dev'}"
                            def nodePort = "${BRANCH_NAME == 'master' ? '32421' : '32422'}"
                            def replicas = "${BRANCH_NAME == 'master' ? '2' : '1'}"
    
                            sh "sed -i 's|IMAGE_NAME|${IMAGE_NAME}:${tag}|g' k8s/deploy.yaml"
                            sh "sed -i 's|NAMESPACE|${namespace}|g' k8s/deploy.yaml"
                            sh "sed -i 's|ENV_NAME|${deploymentEnvironment}|g' k8s/deploy.yaml"
                            sh "sed -i 's|REPLICAS|${replicas}|g' k8s/deploy.yaml"
    
                            sh "sed -i 's|NAMESPACE|${namespace}|g' k8s/svc.yaml"
                            sh "sed -i 's|ENV_NAME|${deploymentEnvironment}|g' k8s/svc.yaml"
                            sh "sed -i 's|NODE_PORT|${nodePort}|g' k8s/svc.yaml"
    
                            sh "kubectl apply -f k8s/deploy.yaml"
                            sh "kubectl apply -f k8s/svc.yaml"
                        }
                    }
                }
            }
        }
    }
}