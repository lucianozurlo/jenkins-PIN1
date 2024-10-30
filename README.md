# Pipeline de Jenkins para Construir y Publicar Imagen de Docker

[![Video Tutorial](https://img.youtube.com/vi/5clu3ngTce0/hqdefault.jpg)](https://www.youtube.com/watch?v=5clu3ngTce0)

Este documento describe el pipeline de Jenkins utilizado para construir y publicar una imagen de Docker para un proyecto. El pipeline se divide en varias etapas, cada una realizando tareas específicas para asegurar una construcción y publicación exitosa de la imagen.

## Estructura del Pipeline

El pipeline se divide en varias etapas, cada una responsable de una parte específica del proceso.

### Etapas

1. **Verificar Instalación de Docker**
    Verificar que Docker esté correctamente instalado en el entorno de Jenkins.
      ```bash
      docker --version
      ```

2. **Limpiar Espacio de Trabajo**
    Limpiar el espacio de trabajo de Jenkins eliminando cualquier archivo o directorio sobrante de ejecuciones anteriores.
      ```groovy
      deleteDir()
      ```

3. **Clonar Repositorio**
    Clonar el repositorio del proyecto desde GitHub.
      ```bash
      git clone https://github.com/EducacionMundose/PIN1.git
      ```

4. **Construir Imagen de Docker**
    Construir la imagen de Docker utilizando el `Dockerfile` presente en el directorio clonado. La imagen se etiqueta como `lucianozurlo/mi-nginx:latest`.
      ```bash
      docker build -t lucianozurlo/mi-nginx:latest .
      ```

5. **Publicar en Docker Hub**
    Iniciar sesión en Docker Hub utilizando las credenciales almacenadas en Jenkins y luego publicar la imagen construida en tu repositorio en Docker Hub.
      ```bash
      echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
      docker push lucianozurlo/mi-nginx:latest
      ```

### Acciones Posteriores

Al final, limpiar el espacio de trabajo para asegurar que no queden residuos de la construcción.
    ```groovy
    deleteDir()
    ```


## Pipeline

```groovy
pipeline {
    agent any
    
    stages {
        stage('Verify Docker Installation') {
            steps {
                script {
                    sh 'docker --version'
                }
            }
        }

        stage('Clean Workspace') {
            steps {
                script {
                    deleteDir()
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    sh 'git clone https://github.com/EducacionMundose/PIN1.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dir('PIN1') {
                        sh 'docker build -t lucianozurlo/mi-nginx:latest .'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push lucianozurlo/mi-nginx:latest'
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo 'Cleaning up...'
                deleteDir()
            }
        }
    }
}
