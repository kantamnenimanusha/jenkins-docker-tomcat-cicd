\# CI/CD Architecture



\## 1. Overview



This project uses Jenkins to automate the build and deployment of a Java web application to an Apache Tomcat server running inside a Docker container on AWS EC2.



The workflow uses two EC2 instances:



\- \*\*Jenkins Server:\*\* Performs source checkout, Maven build, artifact verification, and deployment orchestration.

\- \*\*Docker Host:\*\* Receives deployment files, builds the Docker image, and runs the Tomcat application container.



\## 2. Architecture Diagram



```mermaid

flowchart TD

&#x20;   A\["GitHub Application Repository"] --> B\["Jenkins Server"]

&#x20;   B --> C\["Checkout"]

&#x20;   C --> D\["Check Java and Maven"]

&#x20;   D --> E\["Maven Build"]

&#x20;   E --> F\["Generate WAR"]

&#x20;   F --> G\["Verify WAR"]

&#x20;   G --> H\["SSH / Publish Over SSH"]

&#x20;   H --> I\["Docker Host - AWS EC2"]

&#x20;   I --> J\["Copy deployment files"]

&#x20;   J --> K\["Docker Image Build"]

&#x20;   K --> L\["Tomcat Container"]

&#x20;   L --> M\["Deploy WAR"]

&#x20;   M --> N\["HTTP Health Check"]

&#x20;   N --> O\["Application Available"]

```



\## 3. Pipeline Execution



| Stage | Responsibility |

|---|---|

| Checkout | Retrieves application source from GitHub |

| Check Tools | Checks Java and Maven |

| Build WAR | Runs Maven package lifecycle |

| Verify WAR | Confirms the WAR artifact exists |

| Transfer and Deploy | Transfers files and deploys the container |



\## 4. Communication



1\. Jenkins retrieves source code over HTTPS.

2\. Jenkins connects to the Docker host using SSH.

3\. Publish Over SSH transfers the WAR and Docker configuration.

4\. The Docker host builds and starts the Tomcat container.

5\. The deployment script checks the application locally using HTTP.



\## 5. Deployment Configuration



| Component | Configuration |

|---|---|

| Docker image | `regapp:v2` |

| Container | `registryapp` |

| Tomcat port | `8080` |

| Host port | `8088` |

| Application context | `/webapp/` |

| Health check | `http://localhost:8088/webapp/` |



\## 6. Security Notes



\- SSH authentication uses a private key configured on Jenkins.

\- Private keys and real Tomcat credentials are excluded from source control.

\- The actual Tomcat users configuration is provisioned separately on the Docker host.

\- The current demonstration Dockerfile copies Tomcat credentials into the image. This is a known security limitation and must be improved before production use.

\- The lab uses a direct container replacement strategy rather than a zero-downtime deployment strategy.



\## 7. Current Verification Status



The AWS lab deployment was successfully tested:



\- Jenkins pipeline completed successfully.

\- WAR artifact was generated.

\- SSH transfer completed.

\- Docker image was built.

\- Tomcat container started.

\- Application endpoint returned HTTP 200.



The EC2 infrastructure may be stopped when the lab is not in use. A fresh deployment test requires the instances and their network connectivity to be available.

