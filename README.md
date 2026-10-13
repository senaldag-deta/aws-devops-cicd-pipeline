# Docker CI/CD Pipeline Using GitHub Actions

## About This Project

I built this project to get hands-on experience with Docker and CI/CD pipelines rather than just learning the concepts.

I started by creating a simple Python Flask application and running it locally. After getting the application working, I used Docker to package it into a container so it could run consistently without having to set up Python and its dependencies each time.

The next step was automating the process. I used GitHub Actions to build the Docker image and check whether the Flask application was running correctly. Once the health check passes, the workflow automatically publishes the image to GitHub Container Registry (GHCR).

I also added Docker image versioning using Git commit IDs, making it easier to identify different builds and retrieve a specific version when needed.

The main goal of this project was to understand how code moves from a GitHub repository into a tested, reusable Docker image.

## Tools and Technologies

| Technology | How I Used It |
|---|---|
| Python | Developed the application |
| Flask | Created a simple web application with a health-check endpoint |
| Docker | Containerized the application |
| Git | Tracked changes to the project |
| GitHub | Hosted the source code |
| GitHub Actions | Automated Docker builds, testing, and image publishing |
| GitHub Container Registry | Stored the Docker images |
| YAML | Configured the GitHub Actions workflow |
| VS Code | Developed and managed the project |

## How the CI/CD Pipeline Works

The pipeline starts whenever I push changes to the `main` branch.

GitHub Actions builds the Docker image, starts a container, and checks the Flask application's `/health` endpoint.

If the application responds correctly, the workflow logs in to GitHub Container Registry and publishes the Docker image.

If the build or health check fails, the workflow stops without publishing a new image.

```text
        Push Code to GitHub
                 |
                 v
       GitHub Actions Starts
                 |
                 v
          Build Docker Image
                 |
                 v
        Start Flask Container
                 |
                 v
        Check /health Endpoint
                 |
          +------+------+
          |             |
        Pass           Fail
          |             |
          v             v
     Log in to GHCR   Stop Pipeline
          |
          v
     Publish Docker Image
          |
          v
       Image Tags
       - latest
       - Git Commit ID
```

The workflow also runs its build and health-check steps for pull requests targeting `main`, but image publishing happens only on pushes.

## Project Structure

```text
aws-devops-cicd-pipeline/
|
|-- .github/
|   |-- workflows/
|       |-- docker-build.yml
|
|-- app.py
|-- Dockerfile
|-- requirements.txt
|-- .gitignore
|-- README.md
```

The main files are:

- **app.py:** Contains the Flask application and its health-check endpoint.
- **Dockerfile:** Defines how the Docker image is built.
- **requirements.txt:** Lists the Python dependencies.
- **docker-build.yml:** Contains the GitHub Actions CI/CD workflow.
- **.gitignore:** Prevents unnecessary local files from being tracked by Git.
- **README.md:** Explains the project and how to run it.

## Running the Application Locally

To run this project, you'll need Docker installed and running on your computer.

### 1. Clone the Repository

```bash
git clone https://github.com/senaldag-deta/aws-devops-cicd-pipeline.git
cd aws-devops-cicd-pipeline
```

### 2. Build the Docker Image

```bash
docker build -t aws-devops-flask .
```

This creates a Docker image using the instructions in the Dockerfile.

### 3. Start the Container

```bash
docker run -d -p 5000:5000 --name flask-app aws-devops-flask
```

This starts the Flask application inside a Docker container and makes it accessible through port 5000.

### 4. Check Whether the Application Is Running

Open the following address in your browser:

http://localhost:5000/health

If everything is working correctly, you'll see:

```json
{"status":"healthy"}
```

This is the same health-check endpoint used in the GitHub Actions pipeline.

### 5. Stop the Container

```bash
docker stop flask-app
```

## Using the Published Docker Image

One of the things I wanted to test was whether the image created by GitHub Actions could be downloaded and run separately from my local build.

After the pipeline published the image to GHCR, I downloaded it using:

```bash
docker pull ghcr.io/senaldag-deta/aws-devops-cicd-pipeline:latest
```

I then started a container using the downloaded image:

```bash
docker run -d -p 5001:5000 --name flask-ghcr ghcr.io/senaldag-deta/aws-devops-cicd-pipeline:latest
```

I opened:

http://localhost:5001/health

The application returned:

```json
{"status":"healthy"}
```

This confirmed that the image published through the pipeline could be downloaded and run successfully.

## Docker Image Versioning

Initially, my workflow published Docker images using only the `latest` tag.

I later updated the pipeline to publish two tags for each successful build:

- **latest:** Identifies the most recently published image.
- **Git commit ID:** Identifies the image created from a particular commit.

For example, one of my successful builds was published with the tag `d9e9b2b`.

```bash
docker pull ghcr.io/senaldag-deta/aws-devops-cicd-pipeline:d9e9b2b
```

This is useful because the `latest` tag changes with new builds, while a commit-based tag helps identify and retrieve a specific image version.

## GitHub Actions Workflow

The CI/CD workflow is configured in:

`.github/workflows/docker-build.yml`

It performs the following steps:

1. Checks out the repository.
2. Builds the Flask application's Docker image.
3. Starts the application in a Docker container.
4. Tests the `/health` endpoint.
5. Logs in to GitHub Container Registry.
6. Publishes the image with `latest` and commit-based tags.

The workflow runs automatically when changes are pushed to `main` or when a pull request targets `main`. The publishing steps run only for push events.

## What I Learned

Working on this project helped me understand how the different parts of a CI/CD pipeline fit together.

Some of the main things I learned were:

- How to write a Dockerfile and package a Python application into a container.
- How Docker images and containers are different.
- How to use Git and GitHub to manage code changes.
- How GitHub Actions workflows are written using YAML.
- How to automate Docker builds and application health checks.
- How to publish and download Docker images using GitHub Container Registry.
- Why image versioning is useful when managing application releases.

I also gained experience troubleshooting YAML indentation issues, checking GitHub Actions workflow results, and verifying that a published image runs as expected.

## Challenges I Faced

One challenge was getting the YAML indentation correct when adding new steps to the GitHub Actions workflow. Because YAML depends on spacing, I had to make sure each step was placed at the correct level.

I also worked through the Git process of staging, committing, and pushing changes, and learned how to check whether files had actually been included in a commit.

These were small issues, but resolving them helped me become more comfortable working with Git, Docker, and CI/CD configurations.

## What I Plan to Improve Next

The current project covers automated building, testing, and publishing of Docker images. The published application has been tested locally, but it has not yet been deployed to a cloud server.

My next improvements will be:

- Deploy the Docker application to AWS when a suitable environment is available.
- Explore automated deployment after a successful build.
- Add more application tests.
- Introduce security scanning into the pipeline.
- Learn more about monitoring and logging.

## Project Repository

GitHub: https://github.com/senaldag-deta/aws-devops-cicd-pipeline