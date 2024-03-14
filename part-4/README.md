# How to Create a Jenkins CI CD Pipeline

In this section, our goal is to replicate the CI/CD pipeline using Groovy scripting. It is essential to highlight our focus on scripting within Jenkins, showcasing the full potential of Jenkins' capabilities. To illustrate this feature of Jenkins, we will utilize a Jenkinsfile with the Declarative method written in Groovy, which serves as the foundation for creating Jenkinsfiles, especially in Declarative Pipeline syntax. Groovy is a versatile programming language. Designed to be compatible with Java syntax, Groovy is specifically tailored for the Java platform.

From the practical exercises in the lab where we set up node agents for Jenkins, you likely observed the strong reliance of Jenkins on Java. There are multiple reasons for this dependency. To begin with, Jenkins itself is crafted in Java, establishing a natural affinity for supporting Java projects within the platform. When it comes to building Java projects in Jenkins, popular tools like Apache Maven or Gradle are frequently employed. These tools seamlessly integrate with Jenkins, facilitating efficient project builds and automation tasks. The deep connection between Jenkins and Java is rooted in historical factors and the practical advantages of utilizing Java for project development and management within the Jenkins environment.

While we haven't delved into Maven exercises yet, you likely encountered it during your exploration of Jenkins. Jenkins and Maven are interconnected tools in software development. Maven serves as a build tool primarily for handling dependencies and software lifecycle management, while Jenkins specializes in Continuous Integration (CI) for automating tasks such as building, testing, and deploying applications which happens to be the focus of this lab. Jenkins can incorporate Maven as its build tool, facilitating smooth integration of Maven projects into the Jenkins pipeline. Maven's functions in dependency management and automated builds synergize with Jenkins' role in managing the CI/CD pipeline, leading to their frequent joint usage in software development workflows. Enough discussion on the brief history of integration between these technologies; now, let's delve into our lab exercise.


Prerequisites:
- An AWS account with an existing S3 bucket.
- AWS credentials configured in Jenkins (previously covered).
- Necessary Plugins: AWS Steps Plugin, Pipeline Plugin, Pipeline Utility Steps Plugin, and Credentials Binding Plugin. If you installed the suggested plugins during Jenkins setup, you should already have these in place except for  AWS Steps Plugin.

To execute this code, when creating a job or project, navigate to the Dashboard, select "New Item," name your item, and instead of choosing "Freestyle project," opt for "Pipeline." Scroll down to the "Pipeline" section, where you can input or paste the script. The remaining steps remain unchanged. Assuming familiarity with the Jenkins Dashboard; if not, a review of previous labs is recommended.



<img src="/part-4/pics/jp3-4.png" width="500" />

Paste the following code in there.


```
pipeline {
    agent any

    environment {
        TEMP_DIR = '/tmp/deployment'
        HOME_DIR = '/home/cent1-user'
        PIPELINE_FOLDER = "${HOME_DIR}/pipeline_folder"
        AWS_S3_BUCKET = 'jenkins-banana'
    }

    stages {
        stage('Initialize') {
            steps {
                echo 'Initializing the pipeline...'
            }
        }

        stage('Build') {
            steps {
                echo 'Creating files in temporary directory...'
                sh '''
                    mkdir -p $TEMP_DIR
                    touch $TEMP_DIR/source_code.txt
                    touch $TEMP_DIR/library.io
                    pwd
                    cat /etc/os-release
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Checking if files exist in temporary directory...'
                sh '''
                    if [ -f $TEMP_DIR/source_code.txt ] && [ -f $TEMP_DIR/library.io ]; then
                        echo "Files exist, proceeding to next stage."
                    else
                        echo "Files do not exist, exiting pipeline."
                        currentBuild.result = 'FAILURE'
                        error 'Files not found in temporary directory.'
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying files to pipeline folder...'
                sh '''
                    mkdir -p $PIPELINE_FOLDER
                    cp $TEMP_DIR/* $PIPELINE_FOLDER/
                '''
            }
        }

        stage('Backup') {
            steps {
                echo 'Uploading files to AWS S3 bucket...'
                script {
                    withAWS(region: 'eu-west-1', credentials: 'aws_ec2_ireland') {
                        PIPELINE_FILES = sh(script: "ls $PIPELINE_FOLDER", returnStdout: true).trim().split()
                        PIPELINE_FILES.each { file ->
                            s3Upload acl: 'Private', bucket: AWS_S3_BUCKET, file: "$PIPELINE_FOLDER/$file"
                        }
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning up workspace...'
                sh 'rm -rf $TEMP_DIR'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}

```

The commands ` pwd `  and  ` cat /etc/os-release ` can assist you if you have multiple controllers set up and are unsure which one Jenkins has selected.


You can see the bucket before executing this job.



<img src="/part-4/pics/jp3-1.png" width="500" />

After executing the build, here are my stats.



<img src="/part-4/pics/jp3-2.png" width="500" />

After running the build here is the bucket.



<img src="/part-4/pics/jp3-3.png" width="500" />

In Continuous Integration and Continuous Deployment (CI/CD) pipelines or software development processes in general, there are commonly three fundamental stages: building, testing, and deployment. However, additional stages like monitoring and backup can also be integrated to enhance the pipeline's robustness and completeness. By incorporating these extra stages, we not only demonstrate a comprehensive CI/CD workflow but also explore additional functionalities of Jenkins, such as uploading files to S3 storage.

To kick off the process, we begin with the 'Initialize' stage where we outline our intentions and plan for the pipeline execution.

During the 'Build' stage, we replicate the generation of files in a temporary directory, resembling the early phases of software development when coding or scripting begins. This step reflects the process of compiling code to produce an executable result.

Moving on to the 'Test' stage, this is where thorough testing occurs. Following code compilation, testers validate its quality by checking for bugs and compliance with requirements. Although we verify code existence at this point, it represents the testing phase within the software development lifecycle.

In the 'Deploy' stage, following successful testing, we proceed with deployment. Although file manipulation occurs in this stage, it represents the essence of real-world deployment processes.

After extensive effort and collaboration across teams, safeguarding our code becomes paramount. Hence, we introduce a 'Backup' stage to ensure data preservation in a robust location like S3 storage.

Finally, in the 'Clean' stage, we perform post-execution cleanup to eliminate unnecessary artifacts and sensitive data. Making sure we get rid of any leftover sensitive information from our work that  accumulated during development.

#### I trust you found this journey through Jenkins pipelines insightful and enriching. Thank you for accompanying me on this exploration of CI/CD practices within Jenkins.

