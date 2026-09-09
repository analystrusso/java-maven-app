def gv

pipeline {   
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("increment version") {
            steps {
                script {
                        echo 'incrementing app version...'
                        sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} versions:commit'
                        def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                        def version = matcher[0][1]
                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'

                }
            }
        }

        stage("build image") {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "docker build -t analystrusso/twn-bootcamp-repo:${IMAGE_NAME} ."
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push analystrusso/twn-bootcamp-repo:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage("deploy") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
            }
            steps {
                script {
                    echo "deploying app..."
                    sh '''
                        echo "== who am I in AWS =="
                        aws sts get-caller-identity

                        echo "== configure kubeconfig for this identity =="
                        aws eks update-kubeconfig --name <YOUR_CLUSTER_NAME> --region <YOUR_REGION>

                        echo "== can I do anything at all =="
                        kubectl auth can-i create deployments -v=8
                    '''
                    sh 'kubectl create deployment nginx-deployment --image=nginx'
                }
            }
        }

         stage("commit version update") {
            steps {
                script {
                    echo "pushing to github..."
                    sshagent(credentials: ['github-key']) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        sh 'mkdir -p ~/.ssh'
                        sh 'ssh-keyscan -H github.com >> ~/.ssh/known_hosts'
                        sh 'git remote set-url origin git@github.com:analystrusso/java-maven-app.git'
                
                        sh "git fetch origin main"
                        sh 'git add .'
                        sh 'git commit -m "ci:version bump"'
                        sh 'git status'
                        sh "git rebase origin/main"
                        sh "git push origin HEAD:main"
                    }
                }
            }
        }
    } 
}
