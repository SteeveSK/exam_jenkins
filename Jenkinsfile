pipeline {

    environment { // Declaration of environment variables
    KUBE_NAMESPACE_dev = 'dev'
    KUBE_NAMESPACE_qa = 'qa'
    KUBE_NAMESPACE_staging = 'staging'
    KUBE_NAMESPACE_prod = 'prod'
    KUBECONFIG = credentials("KUBE_CONFIG") 
    IP_DEV = '34.240.50.23'
    IP_QA = '34.240.50.23'
    IP_STAGING = '34.240.50.23'
    IP_PROD = '34.240.50.23'
    NODEPORT_CAST_DEV = 30000
    NODEPORT_CAST_QA = 30001
    NODEPORT_CAST_STAGING = 30002
    NODEPORT_CAST_PROD = 30003
    NODEPORT_MOVIE_DEV = 30010
    NODEPORT_MOVIE_QA = 30011
    NODEPORT_MOVIE_STAGING = 30012
    NODEPORT_MOVIE_PROD = 30013
    CI_CAST_IMAGE = 'steevesk/cast-service'
    CI_MOVIE_IMAGE = 'steevesk/movie-service'
    TAG_IMAGE_dev = 'dev'
    TAG_IMAGE_prod = 'prod'
    DOCKER_HUB_USER = 'steevesk'
    GITHUB_USERNAME = 'steevesk1'
    GITHUB_URL = 'github.com/SteeveSK/exam_jenkins.git'
    SSH_URL = 'git@github.com:SteeveSK/exam_jenkins.git'
    DOCKER_TOKEN = credentials("CI_DOCKER_TOKEN")
    }  
  
    agent any

 //   environment {
   //     commonEnv.each { key, value ->
     //       key key, value
       // }
   // }

    stages {
        stage('test') {
        //    when {
         //       expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'dev' }
         //   }
            steps {
                sh '''
                ls
                curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
                chmod 700 get_helm.sh
                ./get_helm.sh
                helm lint ./exam_jenkins_KOM_Steeve/helm
                rm -rf ./get_helm.sh
                '''
            }
        }

        stage('build') {
            when {
                expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'dev' }
            }
            steps {
                script {
                    // Build the CAST container  image
                    sh "docker build -t $CI_CAST_IMAGE:$TAG_IMAGE_dev ./Jenkins_devops_exams/cast-service/"
                    // Tag the CAST container image from latest to the commit ref
                    sh "docker tag $CI_CAST_IMAGE:$TAG_IMAGE_dev $CI_CAST_IMAGE:$CI_COMMIT_SHORT_SHA"

                    // Build the MOVIE container image
                    sh "docker build -t $CI_MOVIE_IMAGE:$TAG_IMAGE_dev ./Jenkins_devops_exams/movie-service/"
                    // Tag the MOVIE container image from latest to the commit ref
                    sh "docker tag $CI_MOVIE_IMAGE:$TAG_IMAGE_dev $CI_MOVIE_IMAGE:$CI_COMMIT_SHORT_SHA"
                }
            }
        }

        stage('push') {
            when {
                expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'dev' }
            }
            steps {
                script {
                    // Login to the Docker Hub Container registry
                    sh "docker login -u $DOCKER_HUB_USER -p $DOCKER_TOKEN"

                    // Push the CAST image
                    sh "docker push $CI_CAST_IMAGE:$CI_COMMIT_SHORT_SHA"
                    // If we are building a tag, push the `latest` container image tag too
                    if (currentBuild.rawBuild.buildVariables.get('CI_COMMIT_TAG')) {
                        sh "docker push $CI_CAST_IMAGE:latest"
                    }

                    // Push the MOVIE image
                    sh "docker push $CI_MOVIE_IMAGE:$CI_COMMIT_SHORT_SHA"
                    // If we are building a tag, push the `latest` container image tag too
                    if (currentBuild.rawBuild.buildVariables.get('CI_COMMIT_TAG')) {
                        sh "docker push $CI_MOVIE_IMAGE:latest"
                    }
                }
            }
        }

        stage('deploy_dev') {
            when {
                expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'dev' }
            }
            steps {
                script {
                    // Deploy to the 'dev' environment
                    withDockerServer([uri: 'tcp://docker:2375']) {
                        sh 'docker info'
                        sh 'rm -Rf .kube'
                        sh 'mkdir .kube'
                        sh 'cat $KUBECONFIG > .kube/config'
                        sh 'helm upgrade --install helm exam_jenkins_KOM_Steeve/helm/ --values=exam_jenkins_KOM_Steeve/helm/values-dev.yaml -n dev'
                    }
                }
            }
        }

        stage('deploy_qa') {
            when {
                expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'dev' }
            }
            steps {
                script {
                    // Deploy to the 'qa' environment
                    withDockerServer([uri: 'tcp://docker:2375']) {
                        sh 'docker info'
                        sh 'rm -Rf .kube'
                        sh 'mkdir .kube'
                        sh 'cat $KUBECONFIG > .kube/config'
                        sh 'helm upgrade --install helm ./exam_jenkins_KOM_Steeve/helm/ --values=./exam_jenkins_KOM_Steeve/helm/values-qa.yaml -n qa'
                    }
                }
            }
        }

        stage('merge_main_prod') {
            when {
                expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'dev' }
            }
            steps {
                script {
                    // Merge 'dev' into 'main'
                    sh 'git branch'
                    sh 'git checkout main'
                    sh 'git merge origin/dev --allow-unrelated-histories'
                    sh "git push https://$GITHUB_USERNAME:$CI_GIT_TOKEN@$GITHUB_URL main"
                }
            }
        }

        stage('deploy_staging') {
            when {
                expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'main' }
            }
            steps {
                script {
                    // Deploy to the 'staging' environment
                    withDockerServer([uri: 'tcp://docker:2375']) {
                        sh 'docker info'
                        sh 'rm -Rf .kube'
                        sh 'mkdir .kube'
                        sh 'cat $KUBECONFIG > .kube/config'
                        sh 'helm upgrade --install helm ./exam_jenkins_KOM_Steeve/helm/ --values=./exam_jenkins_KOM_Steeve/helm/values-staging.yaml -n staging'
                    }
                }
            }
        }

        stage('deploy_prod') {
            when {
                expression { currentBuild.rawBuild.buildVariables.get('CI_COMMIT_REF_NAME') == 'main' }
            }
            steps {
                script {
                    // Deploy to the 'prod' environment
                    withDockerServer([uri: 'tcp://docker:2375']) {
                        sh 'docker info'
                        sh 'rm -Rf .kube'
                        sh 'mkdir .kube'
                        sh 'cat $KUBECONFIG > .kube/config'
                        sh 'helm upgrade --install helm ./exam_jenkins_KOM_Steeve/helm/ --values=./exam_jenkins_KOM_Steeve/helm/values-prod.yaml -n prod'
                    }
                }
            }
        }
    }
}
               
