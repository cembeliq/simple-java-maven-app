node {
    stage('Checkout') {
        checkout scm
    }
    stage('Verify Test Plan') {
        // List files to verify the presence of my_test_plan.jmx
        sh 'ls -la jmeter/my_test_plan.jmx'
    }
    // stage('Build') {
    //     try {
    //         docker.image('maven:3.9.9').inside {
    //             sh 'mvn clean package'
    //         }
    //     } catch (Exception e) {
    //         echo "Build failed: ${e.getMessage()}"
    //         error("Build stage failed")
    //     }
    // }
    // stage('Test') {
    //     try {
    //         parallel (
    //             "unit-tests": {
    //                 docker.image('maven:3.9.9').inside {
    //                     sh 'mvn test'
    //                 }
    //             },
    //             "performance-tests": {
    //                 echo "skip"
    //                 // docker.image('justb4/jmeter:5.5').inside {
    //                 //     sh 'jmeter -n -t jmeter/my_test_plan.jmx -l result.jtl'
    //                 // }
    //                 // archiveArtifacts artifacts: 'result.jtl', allowEmptyArchive: true
    //                 // perfReport sourceDataFiles: 'result.jtl'
    //             }
    //         )
    //     } catch (Exception e) {
    //         echo "Test stage failed: ${e.getMessage()}"
    //         error("Test stage failed")
    //     }
    // }
    stage('Deploy') {
        try {
            withCredentials([sshUserPrivateKey(credentialsId: '6883e1b1-dd40-41fd-8cc1-561e37027b10', keyFileVariable: 'SSH_KEY')]) {
                def keyFile = '/tmp/temp_ssh_key'
                writeFile file: keyFile, text: ${SSH_KEY}
                sh "chmod 400 ${keyFile}"

                def ec2User = 'ubuntu'
                def ec2Host = '3.88.167.27'
                def appJar = 'target/my-app-1.0-SNAPSHOT.jar'
                def remotePath = '/home/ubuntu/simple-java-maven-app/'

                // Debugging: Check key properties
                sh """
                echo "$SSH_KEY" > /tmp/temp_ssh_key_debug
                chmod 600 /tmp/temp_ssh_key_debug
                head -n 2 /tmp/temp_ssh_key_debug
                """

                // Transfer the JAR file
                sh """
                scp -i ${keyFile} -o StrictHostKeyChecking=no ${appJar} ${ec2User}@${ec2Host}:${remotePath}
                """

                // SSH into EC2
                sh """
                ssh -i ${keyFile} -o StrictHostKeyChecking=no ${ec2User}@${ec2Host} << EOF
                pkill -f my-app-1.0-SNAPSHOT.jar || true
                nohup java -jar ${remotePath}my-app-1.0-SNAPSHOT.jar > app.log 2>&1 &
                EOF
                """

                // Clean up the key file
                sh "rm -f ${keyFile}"
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