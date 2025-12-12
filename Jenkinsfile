pipeline{
    agent any
    environment{
        DOCKER_TAG="${getLatestCommitId()}"
    }
    stages{
        stage("Docker Image"){
            steps{
                sh "docker build . -t vigneshak086/pyappeks:${env.DOCKER_TAG}"
            }
        }
        stage("Pust To Docker Hub"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pwd', usernameVariable: 'usr')]) {
                    sh "docker login -u ${usr} -p ${pwd}"
                    sh "docker push vigneshak086/pyappeks:${env.DOCKER_TAG}"
                }
            }
        }
        stage("Update Tag in Deployment YAML and push"){
            steps{
                withCredentials([gitUsernamePassword(credentialsId: 'github-creds', gitToolName: 'Default')]) {
                    sh """
                     git config user.name "Jenkins Server"
                     git config user.email "jenkins@automation.com"
                     git checkout main
                     git pull origin main
                     yq e '.spec.template.spec.containers[0].image = "vigneshak086/pyappeks:${env.DOCKER_TAG}"' -i ./k8s/pyapp-deployment.yml
                     git add .
                     git commit -m 'Docker tag updated by jenkins'
                     git push origin main
                 """
                }
            }
        }
    }
}

def getLatestCommitId() {
    return sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
}
