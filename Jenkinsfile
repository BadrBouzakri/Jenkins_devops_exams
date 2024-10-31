pipeline {
    environment {
        DOCKER_ID = "bouzakri" // Remplacez par votre Docker ID
        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE = "cast-service"
        MOVIE_TAG = "v.${BUILD_ID}.0" // Tag unique pour chaque build de movie-service
        CAST_TAG = "v.${BUILD_ID}.0"  // Tag unique pour chaque build de cast-service
    }
    agent any
    
    stages {
        // Étape de build et push pour movie-service
        stage('Build and Push Movie Service') {
            steps {
                dir('movie-service') { // Navigue dans le répertoire movie-service
                    script {
                        // Construction de l'image Docker pour movie-service
                        sh '''
                        docker build -t $DOCKER_ID/$MOVIE_IMAGE:$MOVIE_TAG .
                        '''

                        // Push de l'image vers DockerHub
                        withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_PASS')]) {
                            sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$MOVIE_IMAGE:$MOVIE_TAG
                            '''
                        }
                    }
                }
            }
        }

        // Étape de build et push pour cast-service
        stage('Build and Push Cast Service') {
            steps {
                dir('cast-service') { // Navigue dans le répertoire cast-service
                    script {
                        // Construction de l'image Docker pour cast-service
                        sh '''
                        docker build -t $DOCKER_ID/$CAST_IMAGE:$CAST_TAG .
                        '''

                        // Push de l'image vers DockerHub
                        withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_PASS')]) {
                            sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$CAST_IMAGE:$CAST_TAG
                            '''
                        }
                    }
                }
            }
        }

        // Déploiement dans le namespace dev avec mise à jour des images
        stage('Deploy to Dev') {
            environment {
                KUBECONFIG = credentials("config") // Récupère kubeconfig pour accéder au cluster
            }
            steps {
                script {
                    // Configuration pour Kubernetes et déploiement dans le namespace dev
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    echo "$KUBECONFIG" > .kube/config

                    # Mise à jour des fichiers YAML pour utiliser les nouveaux tags
                    sed -i "s+image:.*+image: $DOCKER_ID/$MOVIE_IMAGE:$MOVIE_TAG+g" kube/movie-service.yaml
                    sed -i "s+image:.*+image: $DOCKER_ID/$CAST_IMAGE:$CAST_TAG+g" kube/cast-service.yaml
                    
                    # Déploiement dans le namespace dev
                    kubectl apply -f kube/cast-db.yaml -n dev
                    kubectl apply -f kube/movie-db.yaml -n dev
                    kubectl apply -f kube/cast-service.yaml -n dev
                    kubectl apply -f kube/movie-service.yaml -n dev
                    '''
                }
            }
        }

        // Test de déploiement en dev
        stage('Test Deployment in Dev') {
            steps {
                script {
                    // Vérifie que les services sont accessibles après le déploiement
                    sh '''
                    sleep 10 # Attendre que les services soient opérationnels
                    # Test des endpoints pour vérifier que les services répondent
                    curl -f http://192.168.1.4:31000/api/v1/movies || exit 1
                    curl -f http://192.168.1.4:32000/api/v1/casts || exit 1
                    '''
                }
            }
        }

        // Déploiement dans le namespace QA avec mise à jour des images
        stage('Deploy to QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    // Déploiement dans le namespace qa
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    echo "$KUBECONFIG" > .kube/config

                    # Mise à jour des fichiers YAML pour utiliser les nouveaux tags
                    sed -i "s+image:.*+image: $DOCKER_ID/$MOVIE_IMAGE:$MOVIE_TAG+g" kube/movie-service.yaml
                    sed -i "s+image:.*+image: $DOCKER_ID/$CAST_IMAGE:$CAST_TAG+g" kube/cast-service.yaml
                    
                    # Déploiement dans le namespace QA
                    kubectl apply -f kube/cast-db.yaml -n qa
                    kubectl apply -f kube/movie-db.yaml -n qa
                    kubectl apply -f kube/cast-service.yaml -n qa
                    kubectl apply -f kube/movie-service.yaml -n qa
                    '''
                }
            }
        }

        // Déploiement dans le namespace staging avec mise à jour des images
        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    // Déploiement dans le namespace staging
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    echo "$KUBECONFIG" > .kube/config

                    # Mise à jour des fichiers YAML pour utiliser les nouveaux tags
                    sed -i "s+image:.*+image: $DOCKER_ID/$MOVIE_IMAGE:$MOVIE_TAG+g" kube/movie-service.yaml
                    sed -i "s+image:.*+image: $DOCKER_ID/$CAST_IMAGE:$CAST_TAG+g" kube/cast-service.yaml
                    
                    # Déploiement dans le namespace staging
                    kubectl apply -f kube/cast-db.yaml -n staging
                    kubectl apply -f kube/movie-db.yaml -n staging
                    kubectl apply -f kube/cast-service.yaml -n staging
                    kubectl apply -f kube/movie-service.yaml -n staging
                    '''
                }
            }
        }

        // Déploiement en production avec validation manuelle et mise à jour des images
        stage('Deploy to Production') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    // Demande d'approbation manuelle avant déploiement en production
                    def userInput = input(
                        id: 'userInput', message: 'Déployer en production ?', ok: 'Oui',
                        parameters: [
                            choice(name: 'Approval', choices: ['Yes', 'No'], description: 'Sélectionnez Yes pour valider')
                        ]
                    )

                    if (userInput == 'Yes') {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        echo "$KUBECONFIG" > .kube/config

                        # Mise à jour des fichiers YAML pour utiliser les nouveaux tags
                        sed -i "s+image:.*+image: $DOCKER_ID/$MOVIE_IMAGE:$MOVIE_TAG+g" kube/movie-service.yaml
                        sed -i "s+image:.*+image: $DOCKER_ID/$CAST_IMAGE:$CAST_TAG+g" kube/cast-service.yaml
                        
                        # Déploiement dans le namespace prod
                        kubectl apply -f kube/cast-db.yaml -n prod
                        kubectl apply -f kube/movie-db.yaml -n prod
                        kubectl apply -f kube/cast-service.yaml -n prod
                        kubectl apply -f kube/movie-service.yaml -n prod
                        '''
                    } else {
                        echo "Déploiement en production annulé."
                    }
                }
            }
        }
    }
}
