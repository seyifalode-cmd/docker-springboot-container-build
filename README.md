# Docker Spring Boot Container Build

Oluwaseyi Michael Falode | Cloud & DevOps Engineering | Toronto, ON

A Jenkins CI pipeline that compiles a Spring Boot application with Maven and packages it into a Docker container image — automating the path from source code to a deployable container artifact.

---

## Project at a Glance

| | |
|---|---|
| **Tools Used** | Jenkins · Docker · Maven · Spring Boot |
| **Platform** | Jenkins CI server with Docker installed |
| **Languages** | Java (Spring Boot) · Groovy (Jenkinsfile) · Dockerfile |
| **What It Does** | Two-stage pipeline: Maven builds the JAR, Docker wraps it into a container image ready for deployment |

## The Problem This Project Solves

A Spring Boot application that runs on a developer's laptop is not the same thing as a Spring Boot application ready for production. The developer's machine has a specific Java version, specific environment variables, specific ports configured. A teammate's machine is different. A production server is different again. This inconsistency is the root cause of "works on my machine" failures.

Docker eliminates the inconsistency by packaging the application with everything it needs — the JRE, the JAR, the configuration — into a single image. That image runs identically on any machine with Docker installed, whether that is a developer laptop, a test environment, or a production Kubernetes cluster.

Jenkins automates the two-step process: compile the code with Maven, then build the Docker image. Every commit to the repository can trigger this pipeline, ensuring the latest code is always available as a deployable container without anyone manually running build commands.

## Pipeline Architecture

```
SOURCE CODE (Spring-Boot-Sample/)
       |
       | trigger (manual or SCM poll)
       v
JENKINS PIPELINE (Jenkinsfile)
       |
       ├── Stage 1: env
       │     mvn clean install -f ./Spring-Boot-Sample/pom.xml
       │     → compiles Java source
       │     → runs unit tests
       │     → packages as .jar file
       │
       └── Stage 2: build
             docker build -t springboot ./Spring-Boot-Sample/
             → reads Dockerfile
             → copies .jar into image
             → produces: springboot:latest image
                    |
                    v
             DEPLOYABLE CONTAINER IMAGE
             (push to registry or deploy directly)
```

## Repository Structure

```
docker-springboot-container-build/
├── Jenkinsfile                     # Two-stage CI pipeline definition
└── Spring-Boot-Sample/
    ├── Dockerfile                  # Container image definition
    ├── pom.xml                     # Maven build configuration
    └── src/                        # Spring Boot application source
```

## Key Files Explained

### `Jenkinsfile` — Pipeline Definition

```groovy
pipeline {
    agent any
    stages {
        stage('env') {
            steps {
                sh 'mvn clean install -f ./Spring-Boot-Sample/pom.xml'
            }
        }
        stage('build') {
            steps {
                sh "docker build -t springboot ./Spring-Boot-Sample/"
            }
        }
    }
}
```

Stage 1 (`env`) compiles the application and runs tests via Maven. Stage 2 (`build`) calls Docker to create the container image using the Dockerfile in the Spring-Boot-Sample directory.

### `Spring-Boot-Sample/Dockerfile` — Container Definition

Defines how to wrap the compiled Spring Boot JAR into a runnable container image, specifying the base JRE image, copying the artifact, and setting the entry point.

## How to Reproduce

**Prerequisites:** Jenkins server with Maven and Docker installed and configured

```bash
# 1. Clone the repository into Jenkins or configure as SCM source
git clone https://github.com/seyifalode-cmd/docker-springboot-container-build.git

# 2. In Jenkins: New Item → Pipeline → Pipeline script from SCM
#    Point to this repository, branch: main, script path: Jenkinsfile

# 3. Build Now — Jenkins runs both stages

# 4. Verify the image was created on the Jenkins agent
docker images | grep springboot

# 5. Run locally to test
docker run -p 8080:8080 springboot
```

## What This Demonstrates

- Jenkins declarative pipeline syntax
- Maven build lifecycle integration in CI
- Docker image creation as a pipeline stage
- The foundation of a full CI/CD pipeline — this build stage precedes a push-to-registry and deploy stage in a complete pipeline

---

*Oluwaseyi Michael Falode · Cybersecurity & Cloud Security Engineer · Toronto, ON*
