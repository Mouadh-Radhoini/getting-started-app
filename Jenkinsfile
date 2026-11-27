pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(
            name: 'FORCE_TEST_FAILURE',
            defaultValue: false,
            description: 'Demonstrate a failed test'
        )
    }

    environment {
        IMAGE = 'mouadhradhouani/taches'
    }

    stages {

        stage('Image') {
            steps {
                sh '''
                    docker build -t "$IMAGE:$BUILD_NUMBER" .
                    docker tag "$IMAGE:$BUILD_NUMBER" "$IMAGE:recette"
                '''
            }
        }

        stage('Push') {
            steps {
                withCredentials([
                    string(credentialsId: 'docker_hub_pwd',
                           variable: 'DOCKER_TOKEN')
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_TOKEN" | docker login \
                          -u mouadhradhouani --password-stdin

                        trap 'docker logout' EXIT
                        docker push "$IMAGE:$BUILD_NUMBER"
                    '''
                }
            }
        }

        stage('Recette') {
            steps {
                dir('deploy/recette') {
                    sh 'docker compose up -d'
                }
            }
        }

        stage('Tests') {
            steps {
                sh '''
                    READY=0

                    for i in 1 2 3 4 5
                    do
                        echo "Tentative $i"

                        if curl -fsS --max-time 5 \
                          http://localhost:3001/items
                        then
                            READY=1
                            break
                        fi

                        if [ "$i" -lt 5 ]; then
                            sleep 10
                        fi
                    done

                    test "$READY" = "1"

                    TASK="recette-$BUILD_NUMBER"

                    curl -fsS -X POST \
                      http://localhost:3001/items \
                      -H "Content-Type: application/json" \
                      -d "{\\"name\\":\\"$TASK\\"}"

                    curl -fsS http://localhost:3001/items > items.json
                    cat items.json

                    EXPECTED="$TASK"

                    if [ "$FORCE_TEST_FAILURE" = "true" ]; then
                        EXPECTED="impossible-task-$BUILD_NUMBER"
                    fi

                    if grep -Fq "$EXPECTED" items.json
                    then
                        echo "TEST REUSSI"
                    else
                        echo "TEST ECHOUE"
                        exit 1
                    fi
                '''
            }
        }

        stage('Validation') {
            steps {
                withCredentials([
                    string(credentialsId: 'docker_hub_pwd',
                           variable: 'DOCKER_TOKEN')
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_TOKEN" | docker login \
                          -u mouadhradhouani --password-stdin

                        trap 'docker logout' EXIT

                        docker tag "$IMAGE:$BUILD_NUMBER" "$IMAGE:valide"
                        docker push "$IMAGE:valide"
                    '''
                }
            }
        }
    }
}
