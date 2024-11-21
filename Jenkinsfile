node {
    stage('Checkout') {
        checkout scm
    }
    stage('Verify Test Plan') {
        // List files to verify the presence of my_test_plan.jmx
        sh 'ls -la jmeter/my_test_plan.jmx'
    }
    stage('Build') {
        try {
            docker.withServer('unix:///var/run/docker.sock') {
                docker.image('maven:3.9.9').inside {
                    sh 'mvn clean package'
                }
            }

        } catch (Exception e) {
            echo "Build failed: ${e.getMessage()}"
            error("Build stage failed")
        }
    }
    stage('Test') {
        try {
            parallel (
                "unit-tests": {
                    docker.image('maven:3.9.9').inside {
                        sh 'mvn test'
                    }
                },
                "performance-tests": {
                    echo "skip"
                    // docker.image('justb4/jmeter:5.5').inside {
                    //     sh 'jmeter -n -t jmeter/my_test_plan.jmx -l result.jtl'
                    // }
                    // archiveArtifacts artifacts: 'result.jtl', allowEmptyArchive: true
                    // perfReport sourceDataFiles: 'result.jtl'
                }
            )
        } catch (Exception e) {
            echo "Test stage failed: ${e.getMessage()}"
            error("Test stage failed")
        }
    }
    stage('Deploy') {
        try {
            withCredentials([sshUserPrivateKey(credentialsId: '4486cd8b-9fdc-43c2-98b3-de74c2bd5c28', keyFileVariable: 'SSH_KEY')]) {
                // Define variables
                def ec2User = 'ubuntu'          // Replace with your EC2 instance username
                def ec2Host = '3.88.167.27'       // Replace with your EC2 instance public IP
                def appJar = 'target/my-app-1.0-SNAPSHOT.jar' // Path to the built application JAR
                def remotePath = '/home/ubuntu/simple-java-maven-app' // Remote directory on EC2
                
                // Transfer the JAR file to the EC2 instance
                sh """
                scp -i $SSH_KEY ${appJar} ${ec2User}@${ec2Host}:${remotePath}
                """

                // SSH into EC2 and restart the application
                sh """
                ssh -i $SSH_KEY ${ec2User}@${ec2Host} << EOF
                pkill -f my-app-1.0-SNAPSHOT.jar || true   # Stop any running instance of the app
                nohup java -jar ${remotePath}my-app-1.0-SNAPSHOT.jar > app.log 2>&1 &
                EOF
                """
            }
        } catch (Exception e) {
            echo "Deployment failed: ${e.getMessage()}"
            error("Deploy stage failed")
        }
    }
    // stage('Deploy') {
    //     withCredentials([usernamePassword(credentialsId: 'my-ansible-credentials', usernameVariable: 'ANSIBLE_USER', passwordVariable: 'ANSIBLE_PASSWORD')]) {
    //         sh "ansible-playbook -i my-inventory -u ${ANSIBLE_USER} -k -v deploy.yml"
    //     }
    // }

}