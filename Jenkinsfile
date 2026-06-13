pipeline{

    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    tools{
        maven 'Maven 3.9.12'
    }

    environment {
        REGISTRY       = 'https://index.docker.io/v1/'
        DOCKERHUB_CRED = 'bb835d88-f2f7-402f-a964-a492f5cd7cbf'
        IMAGE          = 'andriohanania/carts'
    }

    stages{
        stage('build'){
            steps{
                sh 'mvn compile -Dmaven.compiler.source=17 -Dmaven.compiler.target=17'
            }
        }
        stage('test'){
            steps{
                sh 'mvn clean test -Dmaven.compiler.source=17 -Dmaven.compiler.target=17'
            }
        }
        stage('package'){
            steps{
                sh 'mvn package -DskipTests -Dmaven.compiler.source=17 -Dmaven.compiler.target=17'
            }

            post {
                success {
                    archiveArtifacts '**/target/*.jar'
                }
            }
        }
        stage('docker build & push'){
            when { branch 'master' }   // build/push images only from the main line, not from PRs
            steps {
                script {
                    def gitsha = (env.GIT_COMMIT ?: 'dev').take(7)
                    def tag    = "${env.BUILD_NUMBER}-${gitsha}"
                    docker.withRegistry(env.REGISTRY, env.DOCKERHUB_CRED) {
                        def image = docker.build("${env.IMAGE}:${tag}", ".")
                        image.push()          // immutable tag
                        image.push('latest')  // moving tag
                    }
                }
            }
        }
    }
}