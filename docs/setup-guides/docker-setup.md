# Docker Setup Guide

This guide provides clear instructions on how to set up and run ZeroClaw using Docker. There are two primary ways to run ZeroClaw with Docker: using the **One-Click Installer** (interactive, guided) and using **Docker Compose** (declarative, background service).

## Method 1: The One-Click Installer (Recommended for First-Time Setup)

The ZeroClaw `install.sh` script includes a dedicated `--docker` mode. This is the fastest way to get started if you want an interactive wizard to configure your provider and API keys while running ZeroClaw inside a container.

### What it does:
1. Builds the local Docker image (or pulls the official image if `--skip-build` is used).
2. Creates a local directory (`.zeroclaw-docker` by default) to persist your configuration and workspace data.
3. Runs the interactive `zeroclaw onboard` command inside the container to configure your AI provider and model.

### Steps:

1. **Clone the repository:**
   ```bash
   git clone https://github.com/zeroclaw-labs/zeroclaw.git
   cd zeroclaw
   ```

2. **Run the installer in Docker mode:**
   ```bash
   ./install.sh --docker
   ```

3. **Follow the interactive prompts:**
   The wizard will ask you to select your AI provider (e.g., OpenRouter, OpenAI, Anthropic) and enter your API key.

4. **Verify it's working:**
   After the installer finishes, it provides instructions on how to run commands. The containerized data is persisted under `./.zeroclaw-docker`.
   You can run commands using the container image, for example:
   ```bash
   docker run --rm -it -v $(pwd)/.zeroclaw-docker/.zeroclaw:/zeroclaw-data/.zeroclaw -v $(pwd)/.zeroclaw-docker/workspace:/zeroclaw-data/workspace ghcr.io/zeroclaw-labs/zeroclaw:latest status
   ```

### Non-Interactive One-Click Install
If you already have your API key, you can skip the prompts:
```bash
./install.sh --docker --api-key "sk-..." --provider openrouter
```

## Method 2: Docker Compose (Recommended for Background Services)

If you want to run the ZeroClaw gateway as a persistent background service, Docker Compose is the ideal choice. This method uses the `docker-compose.yml` file provided in the repository.

### What it does:
1. Starts the ZeroClaw gateway as a persistent container running in the background.
2. Automatically restarts the service if it stops.
3. Exposes the gateway API on port `42617`.
4. Persists your data in a named Docker volume (`zeroclaw-data`).

### Steps:

1. **Clone the repository (if you haven't already):**
   ```bash
   git clone https://github.com/zeroclaw-labs/zeroclaw.git
   cd zeroclaw
   ```

2. **Set your API key and Provider as environment variables:**
   You can either export them in your shell or create a `.env` file in the same directory as the `docker-compose.yml` file.
   ```bash
   export API_KEY="sk-your-api-key"
   export PROVIDER="openrouter" # Optional, defaults to openrouter
   ```

3. **Launch the service:**
   Run the following command to start the container in detached mode (background):
   ```bash
   docker compose up -d
   ```

4. **Verify the service is running:**
   Check the logs to ensure the gateway started successfully:
   ```bash
   docker compose logs -f
   ```

5. **Access the Gateway:**
   The ZeroClaw gateway is now available at `http://localhost:42617`.
   You can test it by checking the health endpoint (if your image includes curl):
   ```bash
   curl -f http://localhost:42617/health
   ```
   Or by checking the container status:
   ```bash
   docker compose exec zeroclaw zeroclaw status
   ```

### Stopping the Service
To stop the background service, run:
```bash
docker compose down
```
This will stop and remove the container, but your configuration and workspace data will remain safely stored in the `zeroclaw-data` Docker volume.

## Which method should I choose?

*   **Use the One-Click Installer (`./install.sh --docker`)** if you want an interactive wizard to configure your settings, or if you prefer keeping your persistent data in a local folder (`./.zeroclaw-docker`) rather than a named Docker volume.
*   **Use Docker Compose (`docker compose up -d`)** if you want to run the ZeroClaw gateway as a long-running, self-healing background service (e.g., on a home server or VPS), and you already know your API keys.
