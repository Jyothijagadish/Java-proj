pipeline{
    agent any

    tools{
        jdk 'java-17'
        maven 'maven'
    }

    stages{
        stage('git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/Jyothijagadish/Java-proj.git'
            }
        }

        stage('compile'){
            steps{
                sh 'mvn compile'
            }
        }

        stage('build'){
            steps{
                sh 'mvn clean package'
            }
        }

        stage('code quality check'){
            steps{
                sh '''
                mvn sonar:sonar \
                -Dsonar.projectKey=twitter-app \
                -Dsonar.host.url=http://54.174.198.199:9000 \
                -Dsonar.login=16c2bffbefd4dd7d312727682fd9409d1f15ac13
                '''
            }
        }

        stage('Docker image Build'){
            steps{
                sh 'docker build -t akashjyothi/twitter-app-clone:${GIT_COMMIT} .'
            }
        }

        stage('Docker push'){
            steps{
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push akashjyothi/twitter-app-clone:${GIT_COMMIT}'
                }
            }
        }

        stage('k8s Deployment'){
            steps{
                withKubeConfig(credentialsId: 'kubecredID') {
                    sh '''
                    kubectl apply -f Deployment.yml
                    kubectl apply -f service.yml
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "✅ SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Hi Team,

Good news 🎉

Pipeline completed successfully.

Job Name   : ${JOB_NAME}
Build No   : ${BUILD_NUMBER}
Status     : SUCCESS
Build URL  : ${BUILD_URL}

Regards,
Jenkins
""",
                to: "akashjyothi369@gmail.com"
            )
        }

        failure {
            emailext (
                subject: "❌ FAILURE: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Hi Team,

Pipeline has FAILED ❌

Job Name   : ${JOB_NAME}
Build No   : ${BUILD_NUMBER}
Status     : FAILURE
Build URL  : ${BUILD_URL}

Please check logs for details.

Regards,
Jenkins
""",
                to: "akashjyothi369@gmail.com"
            )
        }
    }
}

