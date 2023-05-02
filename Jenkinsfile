pipeline {
    agent any
    stages {
        stage ('SCM') {
            steps {
                git branch: 'master', 
                credentialsId: 'git_cred', 
                url: 'https://github.com/Abhi-chintu/Springboot_cicd_jenkins.git'
            }
        }
        stage ('Build') {
            steps {
                sh "mvn clean install"
            }
        }
        stage ('Docker Image') {
            steps {
                sh "docker build -t springboothelloeks:${BUILD_NUMBER} ." 
            }
        }
        stage ('Pushing image') {
                steps {
                    withCredentials([usernamePassword(credentialsId: 'dockercred', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh ''' 
                    docker login -u ${username} -p ${password}
                    docker tag springboothelloeks:${BUILD_NUMBER} ${username}/springboothelloeks:${BUILD_NUMBER}
                    docker push ${username}/springboothelloeks:${BUILD_NUMBER}
                    '''
                } 
            }
        }
        stage ('Connect to eks') {
            steps {
                sh "aws eks update-kubeconfig --region ap-south-1 --name eksdemo"
            }
        }
         stage ('Deploy Manifest files') {
            steps{
                dir('k8s') {
                    withCredentials([usernamePassword(credentialsId: 'dockercred', passwordVariable: 'password', usernameVariable: 'username')]) {
                        sh '''
                            cat deployment.yaml | sed "s/{{reponame}}/${username}/g" | sed "s/{{ImageTag}}/${BUILD_NUMBER}/g" | kubectl apply -f -
                            kubectl apply -f service.yaml
                            cat service.yaml
                        '''
                    }
                }
            }
        }
    }
}