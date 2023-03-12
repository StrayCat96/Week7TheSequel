

podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt        
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''')  {
  node(POD_LABEL) {
    stage('Build a gradle project') {
      git 'https://github.com/StrayCat96/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
      container('gradle') {
        stage('Build a gradle project') {
          sh '''
          cd Chapter08/sample1
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
        }
      }
    }

    stage('Build Java Image') {
      container('kaniko') {
        stage('Build a gradle project') {
          sh '''
          echo 'FROM openjdk:8-jre' > Dockerfile
          echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
          echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
          mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
          /kaniko/executor --context `pwd` --destination straycat96/calculator:1.0
          '''
        }
      }
    }

  }
}


{

    node(POD_LABEL) {
        stage('Run pipeline against a gradle project') {
            // "container" Selects a container of the agent pod so that all shell steps are executed in that container.
            container('gradle') {
                stage('Build a gradle project') {
                    // from the git plugin
                    // https://www.jenkins.io/doc/pipeline/steps/git/
                    git 'https://github.com/StrayCat96/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
                    sh '''
                    cd Chapter08/sample1
                    chmod +x gradlew
                    ./gradlew test
                    '''
                }
            }
                stage("Code coverage") {
                    when {
                        env.BRANCH_NAME != "feature"
                    }
                    try {
                        sh '''
        	            pwd
               		    cd Chapter08/sample1
                	    ./gradlew jacocoTestCoverageVerification
                        ./gradlew jacocoTestReport
                        '''
                    } catch (Exception E) {
                        echo 'Failure detected'
                    }

                    // from the HTML publisher plugin
                    // https://www.jenkins.io/doc/pipeline/steps/htmlpublisher/
                    publishHTML (target: [
                        reportDir: 'Chapter08/sample1/build/reports/tests/test',
                        reportFiles: 'index.html',
                        reportName: "JaCoCo Report"
                    ])                       
                }
           }
        }
    }
