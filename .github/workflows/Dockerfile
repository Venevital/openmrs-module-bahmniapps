# Use Ubuntu as the base image
FROM ubuntu:latest

# Set environment variables
ENV HELM_CHART_PATH=/package/helm
ENV NODE_VERSION=14.x
ENV RUBY_VERSION=3.1

# Install dependencies
RUN apt-get update && apt-get install -y \
  wget \
  curl \
  git \
  build-essential \
  ruby \
  python3 \
  python3-pip \
  nodejs \
  npm \
  && rm -rf /var/lib/apt/lists/*

# Install Node.js (specific version 14.x)
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash - \
  && apt-get install -y nodejs

# Install Ruby (specific version 3.1)
RUN gem install --no-document compass

# Install Bower, Grunt CLI, and Yarn
RUN npm install -g bower grunt-cli yarn

# Set up the working directory
WORKDIR /app

# Copy project files to the working directory
COPY . .

# Install Node.js dependencies for micro-frontends
WORKDIR /app/micro-frontends
RUN yarn install --frozen-lock-file

# Run tests
RUN yarn test:ci

# Build micro-frontends
RUN CI=true yarn build || true

# Package the UI
WORKDIR /app/ui
RUN yarn cache clean && /bin/bash ./scripts/package.sh

# Set up Docker Buildx
RUN curl -sL https://github.com/docker/buildx/releases/download/v0.7.1/buildx-v0.7.1.linux-amd64 > /usr/local/bin/buildx && chmod +x /usr/local/bin/buildx

# Set up QEMU for multi-platform build
RUN apt-get update && apt-get install -y qemu-user-static

# Login to DockerHub (pass through the Docker credentials)
# (DOCKER_HUB_USERNAME and DOCKER_HUB_TOKEN will be passed from your CI/CD environment)
ARG DOCKER_HUB_USERNAME
ARG DOCKER_HUB_TOKEN

# Build the Docker image and tag it
RUN docker login -u $DOCKER_HUB_USERNAME -p $DOCKER_HUB_TOKEN && \
    docker buildx create --use && \
    docker buildx build --platform linux/amd64,linux/arm64 -f package/docker/Dockerfile -t venevital/bahmni-web:${ARTIFACT_VERSION} -t venevital/bahmni-web:latest .

# Final step: store built image in .git/build directory (locally)
RUN mkdir -p /app/.git/build && \
    docker save venevital/bahmni-web:${ARTIFACT_VERSION} > /app/.git/build/bahmni-web-${ARTIFACT_VERSION}.tar

# Clean up unnecessary files to reduce image size
RUN rm -rf /app/micro-frontends /app/ui /app/scripts
