---
title: Deploying an ML Model to a Production Website with Containers and GKE
date: February 2024
---

## Deploying a Machine Learning Model to Production Website using Containers (GKE): A Step-by-Step Guide

## **Introduction:**

Ever trained an awesome ML model that just sits collecting dust? Frustrated you can’t bring its powers to the real world? Don’t worry, you’re not alone! Deploying models to production can feel like a black box, especially for beginners. But fear not! This guide is your secret weapon to break down the barrier and unleash your model’s potential.

Forget complex jargon and confusing steps. We’ll walk you through deploying your ML model on a production website, using Docker and Kubernetes (GKE), in a way that’s easy to understand and implement, even if you’re new to these tools.

Ready to see your model make a real impact? Let’s dive in! (And don’t worry, if you need to brush up on Docker or [Kubernetes](https://medium.com/@shreyjain203/gke-and-kubernetes-basic-knowledge-b4155a57f488), I’ve got separate guides to help).

## Setting Up Your Playground: Creating a Google Account and Project for Your ML Deployment

Before we embark on your exciting ML deployment journey, let’s get the essential tools ready! First things first, we’ll need a Google account and a Google Cloud project to utilize the powerful Google Kubernetes Engine (GKE) for deployment. Don’t worry, even if you’re new to Google Cloud, these steps are simple and beginner-friendly!

**Step 1: Create a Google Account (if you don’t have one already):**

Head over to [Google’s Cloud Computing Services](https://cloud.withgoogle.com/) and click “Create Account”. Fill in the basic information like your name and email address. Choose a strong password and you’re almost there! Verify your email and log in to your shiny new Google account.

**Step 2: Create a Google Cloud Project:**

- Visit the Google Cloud Platform Console at [https://console.cloud.google.com/](https://console.cloud.google.com/).
- If you’re new to the platform, click “Create project” and choose a unique name for your project. This name will identify your deployments and resources.
- For advanced users, you can explore different project options in the “Create project” menu.

**Pro Tip:** Remember, this project will store your deployment resources, so choose a descriptive name that reflects your specific ML application.

**Step 3: Enable Billing (Optional, but recommended):**

While you can explore GKE for free with limited resources, enabling billing gives you access to more robust functionalities for your deployment. To activate billing, follow the on-screen instructions in the console. Don’t worry, you can set spending limits to control your costs.

## **Deployment Roadmap**:

![](https://cdn-images-1.medium.com/max/1024/1*NwQchWzlwo-451ISZLU__w.png)

- **Prepare your environment:**

- (Optional) Set up a virtual environment for project isolation.
- Create a dedicated repository with separate folders for front-end and back-end elements.

2.** Containerize your application:**

- Build a Docker image containing your back-end code and dependencies.
- Tag the image for easy identification.

3.** Connect to Google Cloud:**

- Establish a connection between your project and Google Cloud Platform (GCP).

4.** Store your image:**

- Push the Docker image to the Google Artifact Registry for secure storage.

5.** Deploy using Kubernetes:**

- Set up a Kubernetes cluster on GCP to manage your containerized application.
- Deploy your image to the cluster, leveraging Kubernetes for scalability and efficiency.

6. **Connect and access:**

- Retrieve the endpoint IP address from the Kubernetes load balancer.
- Integrate this IP address into your front-end code to enable communication.

Overall, your flow should be somewhat like the image below:

![](https://cdn-images-1.medium.com/max/1024/1*1RoGx7PieFDF2pyObgAP3A.png)

## 1. Prepare your environment:

**1.1 Set up a virtual environment for project isolation:**

Setting up a virtual environment is a crucial step in Python development, particularly for project isolation and managing dependencies. It allows you to create an isolated environment where you can install specific versions of packages without affecting other projects or the system-wide Python installation. To set up a virtual environment for your project, follow these steps:

**Install Virtualenv (if not already installed): **Virtualenv is a tool used to create isolated Python environments. You can install it using pip:

```
pip install virtualenv
```

**Create a Virtual Environment: **Navigate to your project directory in the terminal and create a virtual environment. Replace env_name with the name you want to give your environment:

```
virtualenv env_name
```

This command will create a new directory with the specified name (env_name) containing the isolated Python environment.

**Activate the Virtual Environment: **Activate the virtual environment using the appropriate command for your operating system:

- **On Windows:**

```
env_name\Scripts\activate
```

- **On macOS and Linux:**

```
source env_name/bin/activate
```

After activation, you will see the name of your virtual environment in the terminal prompt.

**Install Dependencies: **With the virtual environment activated, you can now install the required dependencies for your project using pip. For example:

```
pip install package_name
```

This will install the package in the isolated environment, ensuring that it does not affect other projects or the system-wide Python installation.

**Deactivate the Virtual Environment:** Once you’re done working in the virtual environment, you can deactivate it using the following command:

```
deactivate
```

This will return you to the global Python environment.

Setting up a virtual environment for your project is a best practice in Python development, as it helps keep your project dependencies organized and isolated from other projects, reducing potential conflicts and ensuring a consistent development environment.

**1.2 Create a dedicated repository with separate folders for front-end and back-end elements (for Front-end):**

> **Pro Tip:** Use Cloud Shell (Google) instead of Local machine. Even if you have your repository local machine, push it to Google’s Cloud Shell, it will come in handy in the third step (Connect to Google Cloud).

There are 3 main files (apart from the others) to consider here:

![](https://cdn-images-1.medium.com/max/584/1*OOcUmiplTK6idM0WGg8M2g.png)

- requirements.txt
- Dockerfile (no Extension)
- app.py

**requirements.txt:** Imagine this file as your shopping list. It contains all the external libraries and dependencies your application needs to function, ensuring consistency and reproducibility. Each line specifies a library and its version.

**Example:**

```
# Web DevelopmentFlask==2.2.2# Data Analysispandas==1.5.2numpy==1.23.4# Machine Learningtorch==1.13.1
```

**Dockerfile:**

- **Multi-stage Build:** Consider using a multi-stage build for smaller image sizes and improved security. Separate the base image for building dependencies from the final runtime image.
- **Best Practices:** Adhere to Dockerfile best practices like using COPY only for static assets and leveraging multi-line commands for readability.

**Example:**

```
# Use an official Python runtime as a base imageFROM python:3.8-slim# Set the working directory in the containerWORKDIR /app# Copy the current directory contents into the container at /appCOPY . /app# Install any needed packages specified in requirements.txt# For this simple example, no additional packages are neededRUN pip install -r requirements.txt# Run app.py when the container launchesCMD ["python", "./app.py"]
```

**app.py: **This file serves as the main entry point for our Kubernetes setup and should match the name specified in the Dockerfile (in this case, line #14). It’s recommended to designate this file as the primary Flask application that will run on the cluster. The crucial aspect to note here is the host and port number configuration, which will be utilized when creating the cluster. In this example, the** host is set to 0.0.0.0**, representing the global host, and the **port number is set to 80**.

![](https://cdn-images-1.medium.com/max/1024/1*kn9HQpUGKYPM9m7Cqwc20Q.png)

By incorporating these files, you can create deployment files that are robust, reliable, and professional, ensuring smooth deployments and easy collaboration. Remember, these are just starting points, so feel free to customize them based on your specific project needs and best practices in your field.

## 2. **Containerize your application**:

cd to your folder then run the following command:

### 2.2 Create Docker Image:

To create a docker image, run the following code in your cloud shell terminal:

```
docker build -t <registry_domain>/<project_id>/<artificial_registry_repository_name>/<docker_image>:<version_tag> .
```

You can find the registry_domain name by going to blah blah blah

For example:

```
docker build -t us-west2-docker.pkg.dev/project-name-123456/docker-demo/demo-app:v1 .
```

where,

docker build: This is the command used to build a Docker image. It tells Docker to build an image based on the instructions provided in a Dockerfile.

-t flag is used to set the tag. In this case, the image will be tagged as us-west2-docker.pkg.dev/project-name-123456/docker-demo/demo-app:v1. This is a typical format for naming Docker images.

us-west2-docker.pkg.dev is the registry domain, project-name-123456 is the project name, docker-demo is the Artifact Registry repository name, demo-app is the name of the Docker image, and v1 is the version tag.

.: This specifies the build context. It tells Docker to look for a Dockerfile in the current directory (.) and use it to build the Docker image.

You can also use your local system and then create a connection to Google Cloud using Google Cloud SDK to push the image to the Artifact Registry (AR). It’s a little bit complicated to authenticate and everything, but in the end, it doesn’t matter as long as there’s a Docker image in your Artifact Registry with a proper **TAG**.

On the Google Console, enable the Artifact Registry API.

![](https://cdn-images-1.medium.com/max/1024/1*uPQOlBv8UOsE-4kF_4EPyw.png)

After enabling the Artifact Registry API, Create a repository with any name you want as long as it is separated by a hyphen (-) and that will be your Artificial Registry repository name.

![](https://cdn-images-1.medium.com/max/1024/1*gYcj6vHHunuq6wmhCcGv-g.png)

## 5. Create Docker Image:

To create a docker image, run the following code in your cloud shell terminal:

```
docker build -t <registry_domain>/<project_id>/<artificial_registry_repository_name>/<docker_image>:<version_tag> .
```

You can find the registry_domain name by going to blah blah blah

For example:

```
docker build -t us-west2-docker.pkg.dev/project-name-123456/docker-demo/demo-app:v1 .
```

where,

docker build: This is the command used to build a Docker image. It tells Docker to build an image based on the instructions provided in a Dockerfile.

-t flag is used to set the tag. In this case, the image will be tagged as us-west2-docker.pkg.dev/project-name-123456/docker-demo/demo-app:v1. This is a typical format for naming Docker images.

us-west2-docker.pkg.dev is the registry domain, project-name-123456 is the project name, docker-demo is the Artifact Registry repository name, demo-app is the name of the Docker image, and v1 is the version tag.

.: This specifies the build context. It tells Docker to look for a Dockerfile in the current directory (.) and use it to build the Docker image.

## 5. Pushing Docker Image to Artificial Registry (AR)

First, you’ve to activate the Artifact Registry APi if it's not activated.

![](https://cdn-images-1.medium.com/max/1024/1*uPQOlBv8UOsE-4kF_4EPyw.png)

Artifact Registry is a place where something something and something about Artificial Registry.

To push the docker image to Artificial Registry, use this command:

```
docker push <registry_domain>/<project_id>/<repository_name>/<docker_image>:<version_tag> 
```

For Example:

```
docker push us-west2-docker.pkg.dev/project-name-123456/docker-demo/demo-app:v1
```

This command will push the Docker Image to the Artifact Registry.

## 6. Create a cluster

Create a Kubernetes cluster by going to the Kubernetes Engine. This cluster will help bla bla bla

![](https://cdn-images-1.medium.com/max/1024/1*Zvh1XBuX53StQj4mf33g9g.png)

Go there and create a cluster, choose all the default settings, and if you want to change the default setting and want to know more about those, go check out my other [blog](https://medium.com/@shreyjain203/gke-and-kubernetes-basic-knowledge-b4155a57f488).

![](https://cdn-images-1.medium.com/max/1024/1*ajZitUSP7S-cGBrSvpbPIA.png)

![](https://cdn-images-1.medium.com/max/1024/1*rYIaFKjKIqJs791TL9xfWQ.png)

![](https://cdn-images-1.medium.com/max/1024/1*cLL5nyPJamY6mAVcdJ5qvQ.png)

Click create.

![](https://cdn-images-1.medium.com/max/1024/1*4NaFBbEkrXlmM9_4TiWzQQ.png)

Creating a cluster would require a lot of time (20–30 minutes or maybe more), so be patient.

## 7. Deploy cluster on Kubernetes

Once your cluster is created, go inside the cluster and click on Deploy. Choose the location of your image (The one you saved in step 5). Click Continue. In the Configuration tab, choose the cluster in which you want to deploy this image. Click Continue. Then In the expose tab, set the target port the same as the one you put in your app.py file. Copy the exposed API and all set.

## 8. Frontend code for calling the API

In your Python code, use the POST method to the API you just copied from the above step, hit the post request to that API, and all set.
