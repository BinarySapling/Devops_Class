# GitHub Container Registry (GHCR) Guide: Publishing an Nginx Image

GitHub Container Registry (GHCR) is a package hosting service provided by GitHub that allows developers to host and manage Docker container images and other OCI-compatible images. It is tightly integrated with GitHub repositories and GitHub Actions, offering granular permissions and seamless CI/CD workflows.

The registry domain for GHCR is `ghcr.io`.

In this tutorial, we will learn how to build, tag, push, and pull a custom Nginx Docker image using GHCR.

## Prerequisites
1. A GitHub account.
2. Docker installed on your local machine.

---

## Step 1: Generate a Personal Access Token (PAT)
To authenticate with `ghcr.io` from your local Docker CLI, you need a Personal Access Token (Classic).

1. Go to your GitHub account **Settings**.
2. Scroll down on the left sidebar and click on **Developer settings**.
3. Click on **Personal access tokens** -> **Tokens (classic)**.
4. Click **Generate new token (classic)**.
5. Give it a descriptive note (e.g., "GHCR Docker Login").
6. Select the following scopes:
   * `write:packages` (this will automatically select `read:packages`)
   * `delete:packages` (optional, if you want to be able to delete images)
7. Click **Generate token** and **copy the token immediately**. (Store it securely; you won't see it again!).

---

## Step 2: Authenticate Docker with GHCR
On your local machine, use the `docker login` command to authenticate with the GitHub Container Registry. Replace `USERNAME` with your GitHub username.

```bash
# Export your PAT as an environment variable (Linux/macOS)
export CR_PAT="YOUR_PERSONAL_ACCESS_TOKEN"

# Log in to ghcr.io
echo $CR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

*For Windows PowerShell:*
```powershell
$env:CR_PAT="YOUR_PERSONAL_ACCESS_TOKEN"
$env:CR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```
If successful, you will see a `Login Succeeded` message.

---

## Step 3: Create a Custom Nginx Image
Let's create a custom Nginx image with a modified `index.html` simply to demonstrate pushing a unique image.

1. Create a new directory and navigate into it:
   ```bash
   mkdir ghcr-nginx && cd ghcr-nginx
   ```

2. Create a custom `index.html` file:
   ```html
   <!-- index.html -->
   <!DOCTYPE html>
   <html>
   <head>
       <title>My GHCR Nginx App</title>
   </head>
   <body>
       <h1>Hello from GitHub Container Registry!</h1>
   </body>
   </html>
   ```

3. Create a `Dockerfile`:
   ```dockerfile
   # Dockerfile
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/index.html
   EXPOSE 80
   ```

---

## Step 4: Build the Docker Image
Build the Docker image locally.

```bash
docker build -t custom-nginx .
```

---

## Step 5: Tag the Image for GHCR
To push an image to GHCR, it must be tagged with the specific registry URL format:
`ghcr.io/<OWNER>/<IMAGE_NAME>:<TAG>`

Replace `<OWNER>` with your GitHub username (in lowercase).

```bash
docker tag custom-nginx ghcr.io/YOUR_GITHUB_USERNAME/custom-nginx:1.0.0
docker tag custom-nginx ghcr.io/YOUR_GITHUB_USERNAME/custom-nginx:latest
```

---

## Step 6: Push the Image to GHCR
Now that the image is properly tagged and you are authenticated, push it to GitHub Container Registry.

```bash
docker push ghcr.io/YOUR_GITHUB_USERNAME/custom-nginx:1.0.0
docker push ghcr.io/YOUR_GITHUB_USERNAME/custom-nginx:latest
```

Once the push is complete, go to your GitHub Profile. Click on the **Packages** tab. You should see your newly uploaded `custom-nginx` image holding the tags `1.0.0` and `latest`.

---

## Step 7: Manage Package Visibility (Optional)
By default, images published via personal accounts are Private. If you want others to be able to pull your image without authenticating:

1. Click on your package in the GitHub UI.
2. Go to **Package settings** (usually found on the right sidebar).
3. Scroll down to the **Danger Zone**.
4. Click **Change visibility** and set it to **Public**.

---

## Step 8: Pull and Run the Image from GHCR
To verify everything works, remove your local images and run the container directly from GHCR.

```bash
# Remove local images to prove we are pulling from GHCR
docker rmi custom-nginx ghcr.io/YOUR_GITHUB_USERNAME/custom-nginx:latest ghcr.io/YOUR_GITHUB_USERNAME/custom-nginx:1.0.0

# Run the container from GHCR
docker run -d -p 8080:80 --name my-ghcr-app ghcr.io/YOUR_GITHUB_USERNAME/custom-nginx:latest
```

Now, open your browser and navigate to `http://localhost:8080`. You should see the custom "Hello from GitHub Container Registry!" message.

---

## Summary of Key GHCR Commands

| Action | Command |
| :--- | :--- |
| **Login** | `docker login ghcr.io -u <username>` |
| **Tag** | `docker tag <local-image> ghcr.io/<username>/<image-name>:<tag>` |
| **Push** | `docker push ghcr.io/<username>/<image-name>:<tag>` |
| **Pull** | `docker pull ghcr.io/<username>/<image-name>:<tag>` |
