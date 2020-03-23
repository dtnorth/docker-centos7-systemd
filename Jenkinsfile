#!/usr/bin/env groovy
 
def call(Map params = [:]) {
    
    def projectKey = params.containsKey('projectKey') ? params.projectKey : 'GDGER'
    def imageName = params.containsKey('imageName') ? params.imageName : 'FROM_GIT_REPO'
    def tagName = params.containsKey('tag') ? params.tag : "${BRANCH_NAME}"
    def repository = params.containsKey('repository') ? params.repository : 'davis.wincor-nixdorf.com:8095'
    
    pipeline {
        agent { node { label 'docker-centos7-systemd' } }
    
        environment {
            REPO_URL="$repository"
            IMAGE_NAME="${imageName == 'FROM_GIT_REPO' ? determineRepoName() : $imageName}"
            TAG_NAME="$tagName".replace("feature/","")
        }
            
        stages {
            
            stage('Checkout SCM') {
                steps {
                    sh 'git checkout $BRANCH_NAME'
                    sh 'git pull'
                }
            }
    
            stage('Build Feature') {
                steps {
                    sh "docker build --tag $REPO_URL/$IMAGE_NAME:tmp ."
                }
            }
            
            stage('Upload Feature') {
                when { not { branch 'master' } }
                steps {
                    sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:$TAG_NAME"
                    sh "docker push $REPO_URL/$IMAGE_NAME:$TAG_NAME"
                }
            }
    
            stage('Upload Release') {
                when { branch 'master' }
                    
                steps {
                    sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:$BUILD_NUMBER"
                    sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:latest"
                    
                    sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:latest"
                    sh "docker push $REPO_URL/$IMAGE_NAME:$BUILD_NUMBER"
                    sh "docker push $REPO_URL/$IMAGE_NAME:latest"
                    
                    sh "git tag master-$BUILD_NUMBER"
                    sh "git push --tags"
                }
            }
        }
    }
}
 
def String determineRepoName() {
    return scm.getUserRemoteConfigs()[0].getUrl().tokenize('/')[3].split("\\.")[0]
}
 
 
# Call the Pipeline
call();
