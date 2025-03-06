pipeline {
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time:30, unit:'MINUTES')
        disableConcurrentBuilds()
        //retry(1)
    }
    environment {
        DEBUG = 'true'
        appVersion= '' // this will be global, we can use across pipeline
        region='us-east-1'
        account_id= ''
        project= 'expense'
        environment='dev'
        component= 'frontend'
    }

    stages {
        stage('Read Version') {
            steps {
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App Version is ${appVersion}"
                }

            }
        }
        stage('Docker Build') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion} .

                    docker images

                    docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion}
                    """
                }
            }
        }
        stage('Deploy'){
            steps{
                 withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                        helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml
                    """
                 }    
            }
        }
        
    }
    post{
        always{
            sh 'echo This runs always'
            deleteDir()
        }
        success{
            sh 'echo This runs when pipeline is success'
        }
        failure{
            sh 'echo This runs when pipeline fails'
        }
    }
}