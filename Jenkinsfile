node {
    
    stage ('Clone Master') {
        git credentialsId: 'git-hub-credentials', url: 'https://github.com/jtb75/GoCICD.git'
    }
   
    stage ('Build image') {
        container('build') {
            echo 'Building..'
            sh """
            chmod 777 /var/run/docker.sock
            docker build -t webapps/gocicd:$BUILD_NUMBER .
            """
        }
    }
    
    stage ('Scan Image') {
        echo 'Scanning..'
        prismaCloudScanImage ca: '',
                    cert: '',
                    dockerAddress: 'unix:///var/run/docker.sock',
                    ignoreImageBuildTime: true,
                    image: 'webapps/gocicd:$BUILD_NUMBER',
                    key: '',
                    logLevel: 'info',
                    podmanPath: '',
                    project: '',
                    resultsFile: 'prisma-cloud-scan-results.json'
    }
    
    stage ('Publish Scan Results') {
        echo 'Publishing Scan Results..'
        prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
    } 
    
    stage ('Push Image') {
        withCredentials([usernamePassword(credentialsId: 'harbor_cred', passwordVariable: 'HARBOR_PW', usernameVariable: 'HARBOR_USER')]) {
            container('build') {
                echo 'Pushing..'
                sh """
                docker tag webapps/gocicd:$BUILD_NUMBER 192.168.1.211:80/webapps/gocicd:$BUILD_NUMBER
                docker tag webapps/gocicd:$BUILD_NUMBER 192.168.1.211:80/webapps/gocicd:latest
                docker login --username ${HARBOR_USER} --password ${HARBOR_PW} 192.168.1.211:80
                docker push 192.168.1.211:80/webapps/gocicd:$BUILD_NUMBER
                docker push 192.168.1.211:80/webapps/gocicd:latest
                """
            }
        }
    }

    stage ('Cleanup') {
            container('build') {
            echo 'Pushing..'
            sh """
            docker rmi 192.168.1.211:80/webapps/gocicd:latest
            docker rmi 192.168.1.211:80/webapps/gocicd:$BUILD_NUMBER
            docker rmi webapps/gocicd:$BUILD_NUMBER
            """
        }
    }
        
}
