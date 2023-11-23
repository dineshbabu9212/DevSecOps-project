pipeline{
    agent {
        docker {
            image "dineshbabu12/red-ja-maven:v3"
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages{
        stage('1.Clean the workspace'){
            steps{
                cleanws()
            }
        }
        stage('2.Clone the code from the GITHUB'){
            steps{
                checkout scmGit(branches: [[name: '*/dinesh_mod']], extensions: [], userRemoteConfigs: [[credentialsId: 'GITHUB_PASS', url: 'https://github.com/dineshbabu9212/DevSecOps-project.git']])
            }
        }
        stage('3.Build the package using maven  and run the junit if the build success'){
            steps{
                sh ' mvn clean install'

            }
            post{
                success{
                    junit 'target/surefire-reports/**/*.xml'
                }
            }
        }
        stage('4.scan the code with sonar'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-devops-ajay') {
                     sh ' mvn sonar:sonar -Dsonar.projectName=praveen-spring -Dsonar.projectKey=praveen-spring'
                    }
                }
            }
        }
        stage('5.wait for the qulity gate for 1 hours max'){
            steps{
               timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-devops-ajay'
               }
            }
        }
        stage('6.Build the Docker image with the pakcage file'){
            steps{                 
                sh ' docker build -t spring-praveen:v${BUILD_NUMBER} .'
                sh ' docker tag spring-praveen:v${BUILD_NUMBER} dineshbabu12/spring-praveen:v${BUILD_NUMBER} ' 
            }
        }
        stage('7.SCAN the image with trivy'){
            steps{
                sh ' trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html dineshbabu12/spring-praveen:v${BUILD_NUMBER} '
            }
        }
        stage('8.Update the docker image to docker Hub'){
            steps{
                script{                   
                   withDockerRegistry(credentialsId: 'DOCKER_HUB') {
                    sh ' docker push dineshbabu12/spring-praveen:v${BUILD_NUMBER} '
                   }
                }
            }
            
        }

    }
    post{
        always{
            sendSlackNotification()
        }
    }
}

def sendSlackNotification()
{
    if (currentBuild.currentResult == "SUCCESS"){
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *SUCCESS*\n Build_url: ${BUILD_URL}\n Job_url: ${JOb_URL} \n "
        slackSend ( channel: '#jenkins', tokenCredentialId: 'slack' , color: 'good', message: "${buildSummary}")
    }
    else{
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *FAILURE*\n Build_url: ${BUILD_URL}\n Job_url: ${JOb_URL} \n "
        slackSend ( channel: '#jenkins', tokenCredentialId: 'slack' , color: 'danger', message: "${buildSummary}")

    }
}
