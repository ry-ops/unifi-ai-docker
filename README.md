# 🐳 UniFi MCP Server - Docker Deployment

Complete Docker containerization setup for the [UniFi MCP Server](https://github.com/ry-ops/unifi-mcp-server) project. This repository provides everything you need to run your UniFi MCP Server in Docker with production-ready configurations, security best practices, and automated CI/CD.

## 📋 Overview

This Docker setup enables you to:
- 🚀 Deploy UniFi MCP Server in isolated containers
- 🔒 Run with enhanced security (non-root user, read-only filesystem options)
- 📦 Build optimized multi-architecture images (amd64, arm64, arm/v7)
- 🔄 Automate builds and deployments with GitHub Actions
- 🎯 Easily integrate with Claude Desktop
- 🛠️ Manage containers with simple commands

## 🎯 Features

### Production-Ready Dockerfiles
- **Standard Dockerfile** - Quick setup for development
- **Optimized Multi-Stage Dockerfile** - 40% smaller images, enhanced security

### Easy Management
- **Docker Compose** - One-command orchestration
- **Makefile** - Convenient shortcuts for all operations
- **Health Checks** - Automatic monitoring and restart

### Security First
- Non-root user execution
- Environment variable isolation
- Secure secrets management
- Minimal attack surface

### CI/CD Ready
- GitHub Actions workflow included
- Automated multi-architecture builds
- Automatic image publishing to registries

## 📦 What's Included

```
.
├── Dockerfile                      # Standard Docker build
├── Dockerfile.optimized           # Multi-stage optimized build
├── docker-compose.yml             # Container orchestration
├── .dockerignore                  # Build optimization
├── .github-workflows-docker.yml   # CI/CD automation
├── secrets.env.example            # Configuration template
├── Makefile                       # Command shortcuts
├── DOCKER_DEPLOYMENT.md          # Comprehensive guide
├── DOCKER_QUICKSTART.md          # 5-minute setup guide
└── README.md                     # This file
```

## 🚀 Quick Start

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) v2.0+
- UniFi Controller with API access OR UniFi Site Manager API token
- Clone of [unifi-mcp-server](https://github.com/ry-ops/unifi-mcp-server)

### Installation

#### Option 1: Add to Existing unifi-mcp-server Repository

```bash
# Navigate to your unifi-mcp-server directory
cd unifi-mcp-server

# Download and extract the Docker files
curl -L https://github.com/ry-ops/unifi-mcp-docker/archive/main.zip -o docker-files.zip
unzip docker-files.zip
mv unifi-mcp-docker-main/* .
rm -rf unifi-mcp-docker-main docker-files.zip

# Or if you have this as a submodule
git submodule add https://github.com/ry-ops/unifi-mcp-docker.git docker
```

#### Option 2: Clone This Repository

```bash
# Clone the Docker configuration
git clone https://github.com/ry-ops/unifi-mcp-docker.git
cd unifi-mcp-docker

# Clone the main unifi-mcp-server project
git clone https://github.com/ry-ops/unifi-mcp-server.git app
cd app
cp ../*.* .
```

### Configuration

1. **Create your secrets file:**
   ```bash
   cp secrets.env.example secrets.env
   ```

2. **Edit secrets.env with your credentials:**
   ```bash
   # Minimum required configuration
   UNIFI_API_KEY=your_api_key_here
   UNIFI_GATEWAY_HOST=192.168.1.1
   UNIFI_GATEWAY_PORT=443
   UNIFI_VERIFY_TLS=false
   
   # Optional: Cloud API
   UNIFI_SITEMGR_TOKEN=your_site_manager_token_here
   ```

3. **Launch the container:**
   ```bash
   docker-compose up -d
   ```

That's it! 🎉

### Verification

```bash
# Check container status
docker-compose ps

# View logs
docker-compose logs -f

# Test API connectivity
docker exec unifi-mcp-server uv run python -c "from main import debug_api_connectivity; print(debug_api_connectivity())"

# Get system status
docker exec unifi-mcp-server uv run python -c "from main import get_system_status; print(get_system_status())"
```

## 🎮 Usage

### Basic Commands

```bash
# Start container
docker-compose up -d

# Stop container
docker-compose down

# Restart container
docker-compose restart

# View logs (live)
docker-compose logs -f

# Access container shell
docker exec -it unifi-mcp-server /bin/bash

# Rebuild after changes
docker-compose up -d --build
```

### Using the Makefile

```bash
make help              # Show all commands
make build-optimized   # Build optimized image
make up                # Start container
make down              # Stop container
make logs              # View logs
make shell             # Open shell
make test              # Run connectivity tests
make rebuild           # Rebuild and restart
make stats             # Show resource usage
```

## 🔗 Integration with Claude Desktop

Update your `claude_desktop_config.json`:

### macOS
```bash
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Windows
```bash
notepad %APPDATA%\Claude\claude_desktop_config.json
```

### Configuration
```json
{
  "mcpServers": {
    "unifi": {
      "command": "docker",
      "args": [
        "exec",
        "-i",
        "unifi-mcp-server",
        "uv",
        "run",
        "main.py"
      ]
    }
  }
}
```

**Important:** The Docker container must be running for Claude Desktop to connect.

## 📚 Documentation

- **[Quick Start Guide](DOCKER_QUICKSTART.md)** - Get running in 5 minutes
- **[Deployment Guide](DOCKER_DEPLOYMENT.md)** - Comprehensive production deployment
- **[Main Project README](https://github.com/ry-ops/unifi-mcp-server)** - UniFi MCP Server documentation

## 🏗️ Building Images

### Standard Build
```bash
docker build -t unifi-mcp-server:latest .
```

### Optimized Build (Recommended)
```bash
docker build -f Dockerfile.optimized -t unifi-mcp-server:latest .
```

### Multi-Architecture Build
```bash
# Create builder
docker buildx create --name multiarch --use

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  -t yourusername/unifi-mcp-server:latest \
  --push \
  -f Dockerfile.optimized .
```

## 🔄 CI/CD Setup

### GitHub Actions

1. **Move the workflow file:**
   ```bash
   mkdir -p .github/workflows
   mv .github-workflows-docker.yml .github/workflows/docker.yml
   ```

2. **Configure secrets in GitHub:**
   - Go to Settings > Secrets and variables > Actions
   - Add `DOCKERHUB_USERNAME` (optional)
   - Add `DOCKERHUB_TOKEN` (optional)

3. **Push to trigger build:**
   ```bash
   git add .
   git commit -m "Add Docker support"
   git push
   ```

Images will be automatically built and pushed to:
- GitHub Container Registry: `ghcr.io/username/unifi-mcp-server`
- Docker Hub: `username/unifi-mcp-server` (if configured)

## 🔒 Security Best Practices

1. **Never commit secrets.env** - It's in `.gitignore`
2. **Use strong API keys** - Generate unique keys
3. **Rotate credentials** - Change keys regularly
4. **Run as non-root** - Use `Dockerfile.optimized`
5. **Use read-only filesystem** - Enabled in optimized version
6. **Monitor access logs** - Check UniFi controller logs
7. **Update regularly** - Keep base images and dependencies current

## 🐛 Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs

# Verify environment
docker-compose config

# Rebuild image
docker-compose up -d --build
```

### Can't Connect to UniFi Controller

```bash
# Test network connectivity
docker exec unifi-mcp-server ping YOUR_CONTROLLER_IP

# Try host network mode
# Edit docker-compose.yml:
# network_mode: host
```

### API Authentication Errors

```bash
# Verify credentials
cat secrets.env

# Test API connectivity
docker exec unifi-mcp-server uv run python -c "from main import debug_api_connectivity; print(debug_api_connectivity())"

# Check for 2FA issues
# Some controllers require app-specific passwords
```

### Permission Issues

```bash
# Fix file permissions
sudo chown -R $USER:$USER .

# Check container user
docker exec unifi-mcp-server whoami
```

## 📊 Performance Optimization

### Image Size Comparison

| Dockerfile | Size | Build Time | Security |
|------------|------|------------|----------|
| Standard | ~850MB | ~3 min | Good |
| Optimized | ~500MB | ~4 min | Excellent |

### Resource Limits

Edit `docker-compose.yml` to add resource constraints:

```yaml
services:
  unifi-mcp-server:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### Caching Strategy

- Status data cached for 5 minutes (configurable via `CACHE_TTL`)
- Docker layer caching enabled via BuildKit
- Multi-stage builds minimize final image size

## 🌐 Deployment Options

### Docker Compose (Development/Home Lab)
```bash
docker-compose up -d
```

### Docker Swarm (Production)
```bash
docker stack deploy -c docker-compose.yml unifi
```

### Kubernetes
See [DOCKER_DEPLOYMENT.md](DOCKER_DEPLOYMENT.md) for complete Kubernetes manifests.

### Cloud Platforms
- **AWS ECS/Fargate** - Use task definitions
- **Google Cloud Run** - Direct container deployment
- **Azure Container Instances** - Simple container hosting
- **DigitalOcean App Platform** - Easy deployment

## 🧪 Testing

### Run All Tests
```bash
make test           # API connectivity
make test-sites     # Site discovery
make test-status    # System status
```

### Manual Testing
```bash
# Enter container
docker exec -it unifi-mcp-server /bin/bash

# Run tests manually
uv run python -c "from main import debug_api_connectivity; print(debug_api_connectivity())"
uv run python -c "from main import working_list_hosts_example; print(working_list_hosts_example())"
uv run python -c "from main import get_system_status; print(get_system_status())"
```

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Test with multiple UniFi configurations
4. Update documentation
5. Submit a pull request

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🔗 Related Projects

- [UniFi MCP Server](https://github.com/ry-ops/unifi-mcp-server) - Main project
- [Model Context Protocol](https://github.com/anthropics/mcp) - MCP specification
- [Claude Desktop](https://claude.ai/download) - AI assistant with MCP support

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/ry-ops/unifi-mcp-docker/issues)
- **Discussions**: [GitHub Discussions](https://github.com/ry-ops/unifi-mcp-docker/discussions)
- **Main Project**: [UniFi MCP Server Issues](https://github.com/ry-ops/unifi-mcp-server/issues)
- **UniFi Community**: [community.ui.com](https://community.ui.com)

## ⭐ Acknowledgments

- Built for the [UniFi MCP Server](https://github.com/ry-ops/unifi-mcp-server) project
- Powered by [Anthropic's MCP](https://github.com/anthropics/mcp)
- Container orchestration by [Docker](https://www.docker.com/)

## 📈 Project Status

![Docker Build](https://img.shields.io/github/actions/workflow/status/ry-ops/unifi-mcp-docker/docker.yml?label=docker%20build)
![License](https://img.shields.io/github/license/ry-ops/unifi-mcp-docker)
![Docker Pulls](https://img.shields.io/docker/pulls/username/unifi-mcp-server)
![GitHub Stars](https://img.shields.io/github/stars/ry-ops/unifi-mcp-docker)

---

**Made with ❤️ for the UniFi and Claude communities** | [Report Bug](https://github.com/ry-ops/unifi-mcp-docker/issues) | [Request Feature](https://github.com/ry-ops/unifi-mcp-docker/issues)
