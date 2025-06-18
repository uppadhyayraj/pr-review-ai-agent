# AI-Powered PR Reviewer - User Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [Usage](#usage)
5. [Features](#features)
6. [Troubleshooting](#troubleshooting)

## Prerequisites

Before setting up the AI-powered PR reviewer, ensure you have:

- Python 3.9 or higher installed
- Git installed
- A GitHub account with permissions to create Personal Access Tokens
- Node.js and npm (for JavaScript/TypeScript test execution)
- Docker and Docker Compose (optional, for containerized deployment)

## Installation

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd ai-agent-pr-reviewer
```

### 2. Install Ollama

```bash
# For macOS/Linux
curl -fsSL https://ollama.ai/install.sh | sh

# Verify installation
ollama --version
```

### 3. Set Up Python Environment

```bash
# Create a virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
.\venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 4. Pull the CodeLLama Model

```bash
# Pull the recommended model
ollama pull codellama:13b

# Start Ollama server
ollama serve
```

## Configuration

### 1. Create GitHub Personal Access Token

1. Go to GitHub Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
2. Generate new token with these permissions:
   - `repo` (full repository access)
   - `pull_requests:write`
   - `contents:write`
3. Copy the generated token

### 2. Set Up Environment Variables

Create a `.env` file in the project root:

```env
GITHUB_TOKEN=your_github_personal_access_token
OLLAMA_URL=http://localhost:11434
OLLAMA_MODEL=codellama:13b
PORT=5000
```

## Usage

### Starting the Service

#### Option 1: Local Development

```bash
# Start Ollama server (in a separate terminal)
ollama serve

# Start the PR reviewer service
python main.py
```

#### Option 2: Using Docker Compose

```bash
docker-compose up --build
```

### Setting Up GitHub Webhook

1. Go to your GitHub repository settings
2. Navigate to Webhooks → Add webhook
3. Configure webhook:
   - Payload URL: `http://your-domain:5000/webhook`
   - Content type: `application/json`
   - Events: Select "Pull requests"
   - Enable SSL verification if using HTTPS
4. Click "Add webhook"

## Features

### Automated PR Analysis
- Code complexity assessment
- Security vulnerability scanning
- Performance optimization suggestions
- Code quality recommendations

### Test Generation & Execution
- Automatic test generation for new code
- Execution of existing tests
- Coverage analysis
- Edge case detection

### AI-Powered Code Review
- Best practices enforcement
- Style guide compliance
- Potential bug detection
- Improvement suggestions

## Troubleshooting

### Common Issues

1. **Ollama Connection Failed**
   ```
   Solution: Check if Ollama server is running:
   ps aux | grep ollama
   ```

2. **GitHub Webhook Not Triggering**
   ```
   Solution: Check webhook delivery logs in GitHub repository settings
   ```

3. **Model Loading Issues**
   ```
   Solution: Verify model download:
   ollama list
   ```

4. **Permission Errors**
   ```
   Solution: Verify GitHub token permissions and environment variables
   ```

### Health Check

Test the service health:
```bash
curl http://localhost:5000/health
```

Test AI functionality:
```bash
curl -X POST http://localhost:5000/test-ai \
  -H "Content-Type: application/json" \
  -d '{"code": "def example(): pass"}'
```

## Performance Optimization

For better performance, you can configure Ollama with:

```bash
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_MAX_LOADED_MODELS=2
export OLLAMA_GPU_LAYERS=35  # If using GPU
```

## Security Notes

- Keep your GitHub token secure and never commit it to version control
- Regularly rotate your GitHub token
- Monitor webhook access logs
- Consider implementing rate limiting for the webhook endpoint
- Use HTTPS in production environments

## Support

For issues and feature requests, please create an issue in the GitHub repository.
