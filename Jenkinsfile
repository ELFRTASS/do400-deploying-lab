/* groovylint-disable-next-line CompileStatic */
pipeline {
    agent {
        kubernetes {
            inheritFrom 'maven'
            serviceAccount 'jenkins'
        }
    }

    // env for quay
    environment {
        QUAY = credentials('QUAY_USER')
    }

    stages {
        stage('Test') {
            steps {
                sh './mvnw verify'
            }
        }
        stage('Build & Push Image') {
            steps {
                sh './mvnw quarkus:add-extension -Dextensions="container-image-jib"'
                sh '''
                ./mvnw package -DskipTests \
                    -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                    -Dquarkus.container-image.build=true \
                    -Dquarkus.container-image.registry=quay.io \
                    -Dquarkus.container-image.group=$QUAY_USR \
                    -Dquarkus.container-image.name=do400-deploying-lab \
                    -Dquarkus.container-image.username=$QUAY_USR \
                    -Dquarkus.container-image.password="$QUAY_PSW" \
                    -Dquarkus.container-image.tag=build-$BUILD_NUMBER \
                    -Dquarkus.container-image.additional-tags=latest \
                    -Dquarkus.container-image.push=true
                '''
            }
        }
        // Download oc for use in the deployment stage.
        stage('Download oc tool') {
            steps {
                sh '''
                    curl -L \
                        https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz \
                        -o oc.tar.gz
                    tar -xvf oc.tar.gz
                    chmod +x oc
                '''
            }
        }
        // Deploy the image built for this Jenkins run to the test environment.
        stage('Deploy to Test') {
            steps {
                sh '''
                    export PATH="$PATH:$WORKSPACE"
                    oc set image deployment/home-automation \
                        home-automation=quay.io/$QUAY_USR/do400-deploying-lab:build-$BUILD_NUMBER \
                        -n igalrq-deploying-lab-test --record
                '''
            }
        }
        // add Deploy to PROD
        stage('Deploy to PROD') {
            steps {
                sh '''
                    export PATH="$PATH:$WORKSPACE"
                    oc set image deployment/home-automation \
                        home-automation=quay.io/$QUAY_USR/do400-deploying-lab:build-$BUILD_NUMBER \
                        -n igalrq-deploying-lab-prod --record
                '''
            }
        }
    }
}