pipeline {

    agent any

    parameters {
        string(name: 'CD_NAME', defaultValue: 'cd-aulautil', description: 'Codedeploy name')
        string(name: 'CD_GROUP', defaultValue: 'cd-group-aulautil', description: 'Codedeploy name')
        string(name: 'BUCKET_DEPLOY', defaultValue: 'aulautil.deploy', description: 'Aws Bucket')

        booleanParam(name: 'BUILD_DEPENDENCIES', defaultValue: false, description: 'Run build job toolchain')
        choice(name: 'CUDA_VERSION', choices: '10.0\n10.1\n9.2', description: 'Cuda Version')
    }

    environment {
        ARTIFACTOR = "${env.BUILD_NUMBER}.zip"
        SLACK_MESSAGE = "Job '${env.JOB_NAME}' Build ${env.BUILD_NUMBER} URL ${env.BUILD_URL}"
    }

    stages {
        stage ('Repository') {
            steps {
                checkout scm
            }
        }

        stage ('Build') {
            steps {
                writeFile file: "build_number.txt", text: "${env.BUILD_NUMBER}"
                sh "./scripts/aws_artifactor_build ${ARTIFACTOR}"
            }
        }

        stage ('Deploy') {
            steps {
                sh "./scripts/aws_artifactor_upload ${params.BUCKET_DEPLOY} ${ARTIFACTOR}"
                sh "./scripts/aws_codedeploy_deployment ${params.CD_NAME} ${params.CD_GROUP} ${params.BUCKET_DEPLOY} ${ARTIFACTOR}"
            }
        }
    }

    post {
        always {
            echo "Job has finished"
            sh "rm -f ${ARTIFACTOR}"
        }
        success {
            slackSendMessage "good"
        }
        failure {
            slackSendMessage "danger"
        }
        unstable {
            slackSendMessage "warning"
        }
    }
}

def slackSendMessage(String color) {
    slackSend channel: "${params.SLACK_CHANNEL}", color: color, failOnError: true, message: "$SLACK_MESSAGE"
}
