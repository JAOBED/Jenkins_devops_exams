pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "jaobed" // replace this with your docker-id
DOCKER_IMAGE_movieservice = "movieservice"
DOCKER_IMAGE_castservice = "castservice"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
BRANCH_NAME = "master"
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_castservice:$DOCKER_TAG /home/ubuntu/Exam/Jenkins_devops_exams/cast-service/
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_movieservice:$DOCKER_TAG /home/ubuntu/Exam/Jenkins_devops_exams/movie-service/
                sleep 6
                '''
                }
            }
        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_movieservice:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_castservice:$DOCKER_TAG

                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                cp ChartApp/values.yaml values.yml
                cat values.yml
                sed -i "s/v.00/\${DOCKER_TAG}/g" values.yml
                helm upgrade --install app ChartApp --values=values.yml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                cp ChartApp/valuesQA.yaml valuesQA.yaml
                cat valuesQA.yaml
                sed -i "s/v.00/\${DOCKER_TAG}/g" valuesQA.yaml
                helm upgrade --install app ChartApp --values=valuesQA.yaml --namespace qa
                '''
                }
            }

        }

stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                cp ChartApp/valuesstaging.yaml valuesstaging.yaml
                cat valuesstaging.yaml
                sed -i "s/v.00/\${DOCKER_TAG}/g" valuesstaging.yaml
                helm upgrade --install app ChartApp --values=valuesstaging.yaml --namespace staging
                '''
                }
            }

        }

stage('Deploiement en PROD') {
            when {
                environment name: 'BRANCH_NAME', value: 'master'
                 }

            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }   

                script {
                    sh '''
                    cp ChartApp/valuesprod.yaml valuesprod.yaml
                    cat valuesprod.yaml
                    sed -i "s/v.00/\${DOCKER_TAG}/g" valuesprod.yaml
                    helm upgrade --install app ChartApp --values=valuesprod.yaml --namespace prod
                    '''
                }

            }
        }

}
}
