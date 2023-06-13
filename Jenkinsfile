pipeline {
    agent { label 'final_project' }
    stages {
        stage('build') {
            steps {
                echo 'build'
                script{
                        withCredentials([usernamePassword(credentialsId: 'dockerhub_cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            sh '''
                                docker login -u ${USERNAME} -p ${PASSWORD}
                                docker build -t msami74/final_project:v${BUILD_NUMBER} .
                                docker push msami74/final_project:v${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../build.txt
                            '''
                        }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                        withCredentials([file(credentialsId: 'my_config_file', variable: 'KUBECONFIG'),
                                        file(credentialsId: 'key_file', variable: 'KUBECONFIG1')]) {
                            sh '''
                                export BUILD_NUMBER=$(cat ../build.txt)
                                mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                rm -f Deployment/deploy.yaml.tmp
                                gcloud auth activate-service-account control-gke@lab1-test-project.iam.gserviceaccount.com --key-file=${KUBECONFIG1}
                                gcloud container clusters get-credentials primary-cluster --zone us-central1-a --project lab1-test-project
                                kubectl apply -f Deployment --kubeconfig=${KUBECONFIG}
                            '''
                        }
                }
            }
        }
    }
}
