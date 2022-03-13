pipeline {

    agent {
        label 'node-BGapp'
    }    
    
    environment {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    stages{
        stage ('Clone project from local git repo') {
            steps{
                echo "Cloningg BGapp project"
                sh 'git clone http://192.168.99.102:3000/kiko/BGapp'    
            }
        }
        stage ('Build test version of the project') {
            steps {
                sh '''
                    echo "Build images from compose file"
                    docker compose build
                    echo "Start the app"
                    docker compose up -d
                '''
                }
        }
        stage ('Reachability Tests') {
            steps {
                echo "Test overall reachability"
                sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080) | grep 200'
    
                echo "Test database connection in 20 seconds"
                sh 'sleep 20'
                sh  'curl -sL localhost:8080 | egrep "Пловдив|Варна"'
            }
        }
        stage ('Clean up test app') {
            steps {
                   sh 'docker compose down'
                }
        }
        stage ('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage ('Push') {
            steps {
                sh '''
                    docker image tag bgapp_db krpetrov/bgapp_db
                    docker push krpetrov/bgapp_db
                    docker image tag bgapp_web krpetrov/bgapp_web
                    docker push krpetrov/bgapp_web
                    '''
            }  
        }
        stage ('Start production app') {
            steps {
                sh 'docker compose -f docker-compose-prod.yml up -d'
            }
        }
    }      
}
