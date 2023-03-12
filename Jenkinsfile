

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
''') {
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
            
            
            stage("Unit test") {
            
        if (env.BRANCH_NAME == "master")
            {
           echo "I am the ${env.BRANCH_NAME} branch"
                   try {
                      sh '''
                      pwd
                      cd Chapter08/sample1
                      chmod +x gradlew
                      ./gradlew test
                       '''
              }

             catch (Exception E) {
               echo 'Failure detected'
           }
             }
        else if (env.BRANCH_NAME == "feature") {
           echo "I am the ${env.BRANCH_NAME} branch"
                   try {
                      sh '''
                      pwd
                      cd Chapter08/sample1
                      chmod +x gradlew
                      ./gradlew test
                       '''
             }   
           
                  catch (Exception E) {
                  echo 'Failure detected'
               }
             }
           else {
           echo "I am the ${env.BRANCH_NAME} branch, I am jumping Unit test stage"
         }
        }
        stage("Code coverage") {
            echo "I am the ${env.BRANCH_NAME} branch"
        if  (env.BRANCH_NAME == "master")
            {
                    try {
                      sh '''
                        pwd
                        cd Chapter08/sample1
                         chmod +x gradlew
                         ./gradlew jacocoTestCoverageVerification
                         ./gradlew jacocoTestReport '''
                       } 
          
             catch (Exception E) {
                        echo 'Failure detected'
                      }
             }
        }


        stage("Code checkstyle") {
           echo "I am the ${env.BRANCH_NAME} branch"
       if (env.BRANCH_NAME == "master") {
               try {
                 sh '''
                 pwd
                 cd Chapter08/sample1
                 chmod +x gradlew
                 ./gradlew checkstyleMain
                 ./gradlew jacocoTestReport '''
            }   
           
                catch (Exception E) {
                  echo 'Failure detected'
               }
       }
       else if (env.BRANCH_NAME == "feature") {
               try {
                 sh '''
                 pwd
                 cd Chapter08/sample1
                 chmod +x gradlew
                 ./gradlew checkstyleMain
                 ./gradlew jacocoTestReport '''
            }   
           
                catch (Exception E) {
                  echo 'Failure detected'
                }
         }
       else {
        echo "I am the ${env.BRANCH_NAME} branch, I am jumping Code checkstyle stage"
        }
     }          

} 
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
