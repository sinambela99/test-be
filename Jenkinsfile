pipeline {
    agent any
    environment{
        credential = 'paul'
        server = '103.175.219.100'
	port = '1234'       
	directory = '/home/paul/literature-backend'
        branch = 'main'
        service = 'backend'
        image = 'iansinambela/li-be'
    }
    stages {
        stage('Pull code dari repository'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no -p ${port} paul@${server} << EOF 
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
                    sh '''ssh -o StrictHostKeyChecking=no -p ${port} paul@${server} << EOF 
                    cd ${directory}
                    docker compose  up -d database
                    docker build -t ${image}:env.BUILD_NUMBER .
                    exit
                    EOF'''
                }
            }
        }
        stage('Testing application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no -p ${port} paul@${server} << EOF 
                    cd ${directory}
                    docker run --name test_fe -p 3000:3000 -d ${image}:latest
                    wget --no-verbose --tries=1 --spider localhost:3000
                    docker stop test_fe
                    docker rm test_fe
                    exit
                    EOF'''
                }
            }
        }
        stage('Deploy aplikasi on top docker'){
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no -p ${port} paul@${server} << EOF
                    docker compose up -d 
                    cd ${directory}
                    exit
                    EOF'''
                }
            }
        }
        stage('Push image to docker hub'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no -p ${port} paul@${server} << EOF 
                    cd ${directory}
                    docker push ${image}:latest
                    exit
                    EOF'''
                }
            }
        }
        stage('send notification to discord'){
            steps {
                discordSend description: "backend notify", footer: "ian notify", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "https://discord.com/api/webhooks/1192097833042579536/qInbfRCW9G8UdivrNlhqeO-AvSijRIdWYbTaGKgKx8sSDR8bCHllZrZ8dPQR3HwKKjau"
            }
        }
    }
}
