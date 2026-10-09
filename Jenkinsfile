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
            // Build only from version branches: 1.0, 2.2, ...  (no 'master' anymore)
            when { branch pattern: '\\d+\\.\\d+', comparator: 'REGEXP' }
            steps {
                script {
                    // version = pom.xml <version> (keep its major.minor == the branch name, e.g. branch 1.0 -> 1.0.x)
                    def ver = sh(script: "mvn -q -DforceStdout help:evaluate -Dexpression=project.version", returnStdout: true).trim()
                    def tag = "${ver}-${env.BUILD_NUMBER}"   // e.g. 1.0.0-7  (immutable, branch-scoped)
                    docker.withRegistry(env.REGISTRY, env.DOCKERHUB_CRED) {
                        def image = docker.build("${env.IMAGE}:${tag}", ".")
                        image.push()   // the only tag we push now — no moving ':latest'
                    }
                    echo "Pushed ${env.IMAGE}:${tag}"
                }
            }
        }
    }
}