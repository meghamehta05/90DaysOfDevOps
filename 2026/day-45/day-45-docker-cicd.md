.github/workflows/docker-publish.yml
name: Docker Build & Push

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/myapp:latest .

      - name: Tag image with commit SHA
        run: |
          docker tag ${{ secrets.DOCKER_USERNAME }}/myapp:latest \
          ${{ secrets.DOCKER_USERNAME }}/myapp:sha-${{ github.sha }}

      - name: Push latest tag
        run: docker push ${{ secrets.DOCKER_USERNAME }}/myapp:latest

      - name: Push SHA tag
        run: docker push ${{ secrets.DOCKER_USERNAME }}/myapp:sha-${{ github.sha }}
 day-45-docker-cicd.md
# Day 45 – Docker Build & Push CI/CD

---

## 1. What I Built

I created a full CI/CD pipeline that:
- Builds a Docker image on every push to main
- Tags the image with latest + commit SHA
- Pushes it automatically to Docker Hub

---

## 2. Workflow Summary

### Trigger
- Runs only on push to `main`

### Steps
- Checkout code
- Login to Docker Hub using secrets
- Build Docker image
- Tag image (latest + sha)
- Push to Docker Hub

---

## 3. Secrets Used

- DOCKER_USERNAME → Docker Hub username
- DOCKER_TOKEN → Docker Hub access token

### Why secrets matter:
- Keeps credentials secure
- Never exposed in logs

---

## 4. Image Tags

- `latest` → always newest version
- `sha-xxxxx` → version-specific build

---

## 5. CI/CD Flow


Git Push
↓
GitHub Actions Trigger
↓
Build Docker Image
↓
Tag Image (latest + sha)
↓
Push to Docker Hub
↓
Image available for deployment


---

## 6. Key Learning

- CI builds automatically
- Docker images are versioned using SHA
- GitHub Actions + Docker Hub = real DevOps pipeline
- Everything is automated end-to-end

---

## 7. Real World Insight

This is exactly how production systems work:
- Developers push code
- CI builds container
- Registry stores image
- CD systems deploy it

---

## Final Outcome

✔ Docker image automatically built  
✔ Docker image pushed to Docker Hub  
✔ Versioned deployments enabled  
✔ Full CI/CD pipeline working 