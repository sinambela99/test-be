pipeline {
    agent any
    environment{
        credential = 'paul'
        server = '103.175.219.100'
	port = '1234'       
	directory = '/home/paul/literature-backend'
        branch = 'main'
        service = 'backend'
        image = 'iansinambela/li-be:latest'
    }
    stages {
        stage('Pull code dari repository'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no -p ${port} paul@${server} << EOF 
                    docker compose down backend
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
                    docker build -t ${image} .
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
                    docker run --name be -p 5000:5000 -d ${image}
                    wget --no-verbose --tries=1 --spider localhost:5000
                    docker stop be
		    docker rm be
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
                    docker push ${image}
                    exit
                    EOF'''
                }
            }
        }
	
	
        stage('send notification to discord'){
            steps {
                discordSend description: "backend notify", 
		footer: "ian notify",
		link: env.BUILD_URL, 
		result: currentBuild.currentResult,
                successful: currentBuild.resultIsBetterOrEqualTo('SUCCESS'), 
		title: JOB_NAME, 
		webhookURL: "https://discord.com/api/webhooks/1232551770614665298/xQdk4sfscxduagJVQ6gdpN1aYAXCIKr-D_L2fALi9pc0qUdcDNTMgq_vHzrxPxpOT-4V"
            }
        }
    }
}
