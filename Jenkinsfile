pipeline {
    agent any
    environment{
        credential = 'id_ed25519'
        server = 'baiksekali@103.150.92.227'
        directory = '/home/baiksekali/literature-backend'
        branch = 'main'
        service = 'backend'
        image = 'iansinambela/lite-be'
    }

    stages {
        stage('Pull code dari repository'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    docker compose down
                    cd ${directory}
                    git pull origin ${branch}
                    exit
                    EOF'''
                }
            }
        }

        stage('Building application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker compose  up -d database
                    docker build -t ${image}:${BUILD_NUMBER} .
                    exit
                    EOF'''
                }
            }
        }

        stage('Testing application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker run --name test_fe -p 3000:3000 -d ${image}:${BUILD_NUMBER}
                    wget --no-verbose --tries=1 --spider localhost:5000
                    docker stop test_fe
                    docker rm test_fe
                    exit
                    EOF'''
                }
            }
        }

        stage('Deploy aplikasi on top kubectl'){
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
		    kubectl apply -f be-app.yaml
                    exit
                    EOF'''
                }
            }
        }

        stage('Push image to docker hub'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker push ${image}:${BUILD_NUMBER}
                    exit
                    EOF'''
                }
            }
        }

        stage('send notification to discord'){
            steps {
                discordSend description: "backend notify", footer: "ian notify", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "https://discord.com/api/webhooks/1232551770614665298/xQdk4sfscxduagJVQ6gdpN1aYAXCIKr-D_L2fALi9pc0qUdcDNTMgq_vHzrxPxpOT-4V"
            }
        }
    }
}
