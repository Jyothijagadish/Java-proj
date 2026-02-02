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
                -Dsonar.projectKey=java-proj \
                -Dsonar.host.url=http://107.23.182.47:9000 \
                -Dsonar.login=516ee64be0b8e09052afa0222c3b6121c5c38fdb
                '''
            }
        }

        stage('Docker image Build'){
            steps{
                sh 'docker build -t akashjyothi/twitter-app:${GIT_COMMIT} .'
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
                    sh 'docker push akashjyothi/twitter-app:${GIT_COMMIT}'
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
                to: "jyothij1056@gmail.com"
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
                to: "jyothij1056@gmail.com"
            )
        }
    }
}

