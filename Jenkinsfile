pipeline {
    agent none
    stages {
        stage('test') {
            agent { docker 'python:3' }
            environment {
                GIT_TAG = 'REL-4_29'
            }
            steps {
                sh '''
               rm -rf pgadmin4
               git clone --depth=1 -b $GIT_TAG https://github.com/postgres/pgadmin4
               cd pgadmin4
               pip install -r ./requirements.txt
               pip install -r ./web/regression/requirements.txt
               make check-pep8
               cd ..
               rm -rf pgadmin4
               '''
            }
        }
        stage('build') {
            environment {
                GIT_TAG = 'REL-4_29'
            }
            agent {
                docker {
                    image 'docker:git'
                    args  '-v "/var/run/docker.sock:/var/run/docker.sock"'
                }
            }
            steps {
                sh '''
                   git clone --depth=1 -b $GIT_TAG https://github.com/postgres/pgadmin4
                   cd pgadmin4
                   docker build .
                   cd ..
                   rm -rf pgadmin4
               '''
            }
        }
        stage('deploy') {
            agent { docker 'willhallonline/ansible:2.9-alpine' }
            environment {
                ANSIBLE_INVENTORY = credentials('APP_ANSIBLE_INVENTORY_FILE')
                SSH_PRIVATE_KEY   = credentials('APP_SSH_PRIVATE_KEY_FILE')

                EMAIL             = credentials('APP_DEFAULT_EMAIL')
                PASSWORD          = credentials('APP_DEFAULT_PASSWORD')
                SERVERS_FILE      = credentials('APP_SERVERS_FILE')

                VERSION = '4.29'
            }
            steps {
                sh '''
                eval $(ssh-agent -s)
                cat "$SSH_PRIVATE_KEY" | tr -d '\r' |ssh-add -
                ansible-playbook \
                    --inventory "$ANSIBLE_INVENTORY" \
                    --ssh-common-args '-o StrictHostKeyChecking=no' \
                    --extra-vars "{\"SERVERS_FILE\": \"$SERVERS_FILE\", \"PASSWORD\": \"$PASSWORD\", \"EMAIL\": \"$EMAIL\", \"VERSION\": \"$VERSION\"}" \
                    ./playbook.yml
                '''
            }
        }
    }
}
