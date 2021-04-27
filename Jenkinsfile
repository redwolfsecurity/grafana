#!groovy
pipeline {
    agent any
    environment {
        NPM_JWT_TOKEN = credentials('8463c965-ba79-41b7-83cf-b66aa1e93c4e')
        JWT_TOKEN = credentials('8463c965-ba79-41b7-83cf-b66aa1e93c4e')
        REGISTRY_HOST = 'dockerregistry.development.redwolfsecurity.com'
        PROJECT_NAME = 'ff_grafana'
    }
    stages {
        stage('Build FF Grafana') {
            environment {
                NPM_JWT_TOKEN = credentials('8463c965-ba79-41b7-83cf-b66aa1e93c4e')
            }
            steps {
                sh 'docker build -t $PROJECT_NAME:$BUILD_TAG .'
            }
        }
        stage('Push Image To Docker Registry') {
            environment {
                NPM_JWT_TOKEN = credentials('8463c965-ba79-41b7-83cf-b66aa1e93c4e')
                REGISTRY_CREDENTIALS = credentials('2083839a-b25c-4d1f-a655-0bddab12a5a6')
                VERSION = sh(returnStdout: true, script: 'git tag --sort version:refname | tail -1').trim()
                TIMESTAMP_EPOCH_MS = sh(returnStdout: true, script: 'date +%s%N | cut -b1-13').trim()
            }
            when {
                anyOf {
                    branch 'master';
                    branch 'develop';
                    branch 'staging';
                    buildingTag()
                }
            }
            steps {
                sh 'echo $REGISTRY_CREDENTIALS_PSW | docker login --password-stdin -u $REGISTRY_CREDENTIALS_USR $REGISTRY_HOST'
                script {
                    if (env.BRANCH_NAME == 'master') {
                        sh 'docker tag $PROJECT_NAME:$BUILD_TAG $REGISTRY_HOST/$PROJECT_NAME:latest'
                        sh 'docker push $REGISTRY_HOST/$PROJECT_NAME:latest'
                        sh 'docker rmi $REGISTRY_HOST/$PROJECT_NAME:latest'
                    } else if (env.BRANCH_NAME == 'develop') {
                        sh 'docker tag $PROJECT_NAME:$BUILD_TAG $REGISTRY_HOST/$PROJECT_NAME:develop'
                        sh 'docker push $REGISTRY_HOST/$PROJECT_NAME:develop'
                        sh 'docker rmi $REGISTRY_HOST/$PROJECT_NAME:develop'
                    } else if (env.BRANCH_NAME == 'staging') {
                        sh 'docker tag $PROJECT_NAME:$BUILD_TAG $REGISTRY_HOST/$PROJECT_NAME:staging'
                        sh 'docker push $REGISTRY_HOST/$PROJECT_NAME:staging'
                        sh 'docker rmi $REGISTRY_HOST/$PROJECT_NAME:staging'
                    } else {
                        sh 'docker tag $PROJECT_NAME:$BUILD_TAG $REGISTRY_HOST/$PROJECT_NAME:$VERSION'
                        sh 'docker push $REGISTRY_HOST/$PROJECT_NAME:$VERSION'
                        sh 'docker rmi $REGISTRY_HOST/$PROJECT_NAME:$VERSION'
                    }
                    if (env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'develop') {
                        sh 'docker tag $PROJECT_NAME:$BUILD_TAG $REGISTRY_HOST/$PROJECT_NAME:$TIMESTAMP_EPOCH_MS-$BUILD_NUMBER-$BRANCH_NAME'
                        sh 'docker push $REGISTRY_HOST/$PROJECT_NAME:$TIMESTAMP_EPOCH_MS-$BUILD_NUMBER-$BRANCH_NAME'
                        sh 'docker rmi $REGISTRY_HOST/$PROJECT_NAME:$TIMESTAMP_EPOCH_MS-$BUILD_NUMBER-$BRANCH_NAME'
                    }
                }
            }
        }
        stage('Deploy To Development Server') {
            when { 
                branch 'develop' 
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'dev.redwolfsecurity.com.pem', keyFileVariable: 'ID_PROFILING_KEY')]) {
                    sh 'ssh -o StrictHostKeyChecking=no -i $ID_PROFILING_KEY $DEPLOY_DEV_SERVER "cd ~/control_install && bash src/bootstrap.sh --update --env=development"'
                }
            }
        }
    }
    post {
        always {
            sh 'docker rmi --no-prune $PROJECT_NAME:$BUILD_TAG'
        }
        cleanup {
            echo 'Performing Cleanup Task'
            deleteDir()
        }
    }
}

