/* groovylint-disable-next-line CompileStatic */
pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        ARGOSERVER = 'argocd-server.example.com'  // Replace with your ArgoCD server address
        ARGOPORT = '443'  // Replace with your ArgoCD server port
        ARGOAPP = 'myapp'  // Replace with your ArgoCD application name
        IMAGE_NAME = 'sagarkp/fakeweb'
        GITHUB_REPO = 'sagarkrp/ArgoMagic'  // Replace with your GitHub repository
        DEPLOYMENT_YAML = 'deployment.yml'  // Replace with the path to your deployment YAML file
    }

    stages {

        stage('Docker Build') {
            steps {
                echo 'Building docker Image'
                sh 'whoami'
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Image to Docker Hub'
                sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }

        
       stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "ArgoMagic"
            GIT_USER_NAME = "sagarkp"
        }
        steps {
            withCredentials([string(credentialsId: 'git_creds', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "$GIT_EMAIL"
                    git config user.name "Sagar"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i 's|image: sagarkp/fakeweb:.*|image: sagarkp/fakeweb:BUILD_NUMBER|' deployment.ym deployment.yml
                    git add ArgoMagic/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
}


    post {
        success {
            script {
                // echo "Build success! Updating deployment to use image:${BUILD_NUMBER}"

                // // Update deployment.yml with the new image and tag
                // sh "sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|' deployment.yml"

                // // Commit the changes to Git (assuming Git is configured in your Jenkins environment)
                // sh "git commit -am 'Update deployment image to ${IMAGE_NAME}:${BUILD_NUMBER}'"
                // sh "git push origin master"  // Adjust branch name if necessary

                // Trigger ArgoCD sync after updating deployment
                sh "curl -k -X POST https://${ARGOSERVER}:${ARGOPORT}/api/v1/applications/${ARGOAPP}/sync"
            }
        }
    }
}
