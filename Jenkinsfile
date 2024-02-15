pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "mshow1980"
        APP_NAME = "skill_app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'Docker-login'
        }
    stages {
        stage('Cleanup Workspace'){
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM'){
            steps {
                git credentialsId: 'github', 
                url: 'https://github.com/mshow1980/Tuesday-test.git',
                branch: 'master'
            }
        }
        stage('Build Docker Image'){
            steps {
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push Docker Image'){
            steps {
                script{
                    docker.withRegistry('', REGISTRY_CREDS ){
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push('latest')
                    }
                }
            }
        } 
        stage('Delete Docker Images'){
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker rmi ${IMAGE_NAME}:latest"
            }
        }
        stage('Updating Kubernetes deployment file'){
            steps {
                script{
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        withCredentials([usernamePassword(credentialsId: 'git-login', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh """
                        git config --global user.name "mshow1980"
                        git config --global user.email "mshow1980@aol.com"
                        cat deployment.yaml
                        sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                        cat deployment.yaml
                        git add deployment.yaml
                        git commit -m 'Updated the deployment file'
                        git push https://${user}:${pass}@github.com/${user}/Tuesday-test.git HEAD:master
                        """  
                        }
                    }
                }
            }
        }
    }
}


// stage('Build Docker Image'){
//             steps {
//                 sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
//                 sh "docker build -t ${IMAGE_NAME}:latest ."
//             }
//         }
//         stage('Push Docker Image'){
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
//                     sh "docker login -u $user --password $pass"
//                     sh "docker push ${IMAGE_NAME}:${IMAGE_TAG} ."
//                     sh "docker push ${IMAGE_NAME}:latest ."
//                 }
//             }
//         }