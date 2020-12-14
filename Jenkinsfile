pipeline {
    agent { docker { image 'willhallonline/ansible:2.9-alpine' } }

    environment {
        ANSIBLE_INVENTORY = credentials('APP_ANSIBLE_INVENTORY_FILE')
        SSH_PRIVATE_KEY   = credentials('APP_SSH_PRIVATE_KEY_FILE')

        EMAIL             = credentials('APP_DEFAULT_EMAIL')
        PASSWORD          = credentials('APP_DEFAULT_PASSWORD')
        SERVERS_FILE      = credentials('APP_SERVERS_FILE')
    }

    stages {
        stage('deploy') {
            steps {
                sh '''
                eval $(ssh-agent -s)
                cat "$SSH_PRIVATE_KEY" | tr -d '\r' |ssh-add -
                ansible-playbook \
                    --inventory "$ANSIBLE_INVENTORY" \
                    --ssh-common-args '-o StrictHostKeyChecking=no' \
                    --extra-vars "{\"SERVERS_FILE\": \"$SERVERS_FILE\", \"PASSWORD\": \"$PASSWORD\", \"EMAIL\": \"$EMAIL\"}" \
                    ./playbook.yml
                '''
            }
        }
    }
}
