pipeline{
    agent any
    environment{
        SONAR_HOME= tool "sonar"
    }
    
    stages{
        stage("code clone"){
            steps{
                git url: "https://github.com/Krishnavats56/chattingo.git", branch: "main"
            }
        }
        stage("Sonarqube Quality analysis") {
    steps {
        withSonarQubeEnv('sonar') {
            sh '''
                ${SONAR_HOME}/bin/sonar-scanner \
                -Dsonar.projectKey=chattingo \
                -Dsonar.projectName=chattingo \
                -Dsonar.sources=. \
                -Dsonar.exclusions=**/*.java
            '''
        }
    }
}

        stage("OWASP dependency check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("trivy file system scan"){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage("build"){
            steps{
                sh "docker build -t frontend-img ./frontend"
                sh "docker build -t backend-img ./backend"
            }
        }
          stage("push to dockerhub"){
            steps{
                withCredentials([usernamePassword(
                    credentialsId: "dockerhubcreds",
                    passwordVariable: "dockerhubPass",
                    usernameVariable: "dockerhubUser",
                    )]){
                        sh "docker login -u ${env.dockerhubUser} -p ${env.dockerhubPass}"
                        sh "docker image tag frontend-img ${env.dockerhubUser}/chattingo-frontend:latest"
                        sh "docker image tag backend-img ${env.dockerhubUser}/chattingo-backend:latest"
                        sh "docker push ${env.dockerhubUser}/chattingo-frontend:latest && docker push ${env.dockerhubUser}/chattingo-backend:latest"
                    }
            }
        }
          stage("deploy"){
            steps{
                sh "docker compose up -d"
            }
        }
    }
}
