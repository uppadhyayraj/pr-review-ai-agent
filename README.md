# GitHub PR AI Agent

[![PyPI version](https://badge.fury.io/py/ghprai.svg)](https://badge.fury.io/py/ghprai)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An AI-powered GitHub Pull Request reviewer that uses local LLMs via Ollama for intelligent code analysis, test generation, and code review automation.

## Architecture

### High-Level System Design

```
+------------------+    +------------------+    +------------------+
|  GitHub Webhook  |--->|    AI Agent      |--->|   Test Runner    |
|   (PR Events)    |    |(Code Analysis &  |    |  (pytest/jest)   |
+------------------+    | Test Generation) |    +------------------+
                        +------------------+              |
                               |                          |
                       +------------------+               |
                       |   Local Git +    |               |
                       |   GitHub API     |<--------------+
                       +------------------+
                               |
                       +------------------+    +------------------+
                       |  Ollama (Local   |    |  Branch Manager  |
                       |      LLM)        |    | (New Test Branch)|
                       +------------------+    +------------------+
                                                        |
                                              +------------------+
                                              |   PR Comments    |
                                              | (AI Analysis &   |
                                              |  Test Results)   |
                                              +------------------+
```

### Component Overview

| Component | Purpose | Technology |
|-----------|---------|------------|
| **GitHub Webhook** | Receives PR events and triggers analysis | Flask REST API |
| **AI Agent** | Core intelligence engine for code analysis | Python + Ollama API |
| **Local Git** | Repository cloning and file management | GitPython |
| **GitHub API** | PR commenting and repository operations | PyGithub |
| **Ollama LLM** | Local AI model for code understanding | CodeLlama/DeepSeek |
| **Test Runner** | Executes generated tests and validates code | pytest/jest/mocha |
| **Branch Manager** | Creates new branches and commits tests | GitPython + GitHub API |
| **PR Comments** | Posts AI analysis and test results | GitHub API |

### Data Flow

1. **PR Event** → GitHub sends webhook to AI Agent
2. **Repository Analysis** → Agent clones repo and analyzes changed files
3. **AI Processing** → Ollama processes code for complexity, risks, and test opportunities
4. **Test Generation** → AI creates comprehensive tests based on analysis
5. **Branch Creation** → New branch created from PR base (e.g., `ai-tests/pr-123`)
6. **Test Commit** → Generated tests committed to new branch
7. **Test Execution** → Generated tests are run to validate functionality
8. **Analysis Report** → AI generates detailed code quality assessment
9. **PR Integration** → Results posted as PR comments with actionable insights
10. **Branch Merge** → Option to merge test branch into PR or create separate PR

### Workflow Details

#### Branch Strategy
```
main branch
├── feature/user-auth (Original PR)
└── ai-tests/pr-123 (AI Generated Tests)
    ├── tests/test_user_auth.py
    ├── tests/test_edge_cases.py
    └── test_report.md
```

#### PR Comment Structure
```markdown
## 🤖 AI Code Analysis Report

### 📊 Code Quality Score: 8.5/10

### 🔍 Analysis Summary
- **Files Analyzed**: 3
- **Functions Found**: 12
- **Complexity Score**: Medium
- **Test Coverage**: 65% → 92% (with generated tests)

### 🧪 Generated Tests
- **Branch**: `ai-tests/pr-123`
- **Test Files**: 4
- **Test Cases**: 23
- **Edge Cases Covered**: 8

### ⚠️ Issues Found
1. **Security**: Potential SQL injection in `user_query.php:45`
2. **Performance**: Inefficient loop in `data_processor.py:78`
3. **Maintainability**: Complex function exceeds 50 lines

### ✅ Recommendations
- [ ] Add input validation for user queries
- [ ] Optimize data processing algorithm
- [ ] Consider breaking down large functions

[View Generated Tests](link-to-test-branch) | [Run Tests Locally](command)
```

## Features

🤖 **Local AI Integration**: Uses Ollama for privacy-focused code analysis without external API calls
🧠 **Intelligent Code Analysis**: AI-powered complexity assessment, security scanning, and optimization suggestions  
🧪 **Smart Test Generation**: Automatic test creation with framework-specific patterns and edge case coverage
📊 **Comprehensive Reports**: Detailed coverage analysis and actionable improvement recommendations
🔄 **Automated Workflow**: Seamless integration with GitHub webhooks for PR automation

## Prerequisites

### System Requirements

- **Operating System**: Linux, macOS, or Windows with WSL2
- **Python**: 3.9 or higher
- **Memory**: Minimum 8GB RAM (16GB recommended for larger models)
- **Storage**: At least 10GB free space for models
- **Network**: Internet connection for initial setup and GitHub integration

### Required Accounts & Tokens

- **GitHub Account** with repository access
- **GitHub Personal Access Token** with the following permissions:
  - `repo` (full repository access)
  - `pull_requests:write`
  - `contents:write`

### Hardware Recommendations

| Model Size | RAM Required | CPU Cores | GPU (Optional) |
|------------|--------------|-----------|----------------|
| 7B models  | 8GB          | 4+ cores  | 4GB VRAM      |
| 13B models | 16GB         | 8+ cores  | 8GB VRAM      |
| 34B models | 32GB         | 16+ cores | 24GB VRAM     |

## Installation & Setup

### Step 1: Install the Package

```bash
# Install from PyPI
pip install ghprai

# Or install from source
git clone https://github.com/yourusername/ai-agent-pr-reviewer
cd ai-agent-pr-reviewer
pip install -e .
```

### Step 2: Install and Configure Ollama

#### On Linux/macOS:
```bash
# Download and install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Start Ollama service
ollama serve
```

#### On Windows:
```bash
# Download Ollama from https://ollama.ai/download
# Run the installer and start Ollama Desktop
```

#### Pull AI Model:
```bash
# Recommended model for balanced performance
ollama pull codellama:13b

# Alternative models:
# ollama pull codellama:7b      # Faster, less capable
# ollama pull codellama:34b     # More capable, requires more resources
# ollama pull deepseek-coder    # Good for code generation
```

### Step 3: Create GitHub Personal Access Token

1. Go to GitHub → Settings → Developer settings → Personal access tokens
2. Click "Generate new token (classic)"
3. Select the following scopes:
   - ✅ `repo` (Full control of private repositories)
   - ✅ `workflow` (Update GitHub Action workflows)
4. Copy the generated token

### Step 4: Configure Environment Variables

Create a `.env` file in your project directory:

```bash
# Required configurations
GITHUB_TOKEN=your_github_personal_access_token
OLLAMA_URL=http://localhost:11434
OLLAMA_MODEL=codellama:13b

# Optional configurations
PORT=5000
LOG_LEVEL=INFO
```

Or export them directly:

```bash
export GITHUB_TOKEN="your_github_token"
export OLLAMA_URL="http://localhost:11434"
export OLLAMA_MODEL="codellama:13b"
```

## How to Use

### Method 1: Webhook Server (Recommended for Automation)

#### Step 1: Start the Server
```bash
# Start the webhook server
ghprai-server 

# With custom configuration
ghprai-server --port 8080 --model codellama:7b
```
### Step 2: Configure Public URL using NGROK
- Install ngrok (this is required for configuring Github Webhook for listening to PR events):
  ```bash
  # macOS with Homebrew
  brew install ngrok

  # Windows with Chocolatey
  choco install ngrok

  # Or download from https://ngrok.com/download
  ```

- In a new terminal, create a public URL with ngrok (note down the public URL):
  ```bash
  ngrok http 5043
  ```
#### Step 3: Configure GitHub Webhook
1. Go to your repository on GitHub
2. Navigate to **Settings** → **Webhooks**
3. Click **Add webhook**
4. Configure the webhook:
   - **Payload URL**: `http://your-domain:5000/webhook`
   - **Content type**: `application/json`
   - **Events**: Select "Pull requests"
   - **Active**: ✅ Checked
5. Click **Add webhook**

#### Step 4: Test the Integration
1. Create a new pull request in your repository
2. Check the webhook deliveries in GitHub settings
3. Verify the agent processes the PR and adds comments

### Method 2: Command Line Interface

#### Analyze a Repository
```bash
# Analyze current directory
ghprai analyze .

# Analyze specific directory
ghprai analyze /path/to/your/repo

# Analyze with specific model
ghprai analyze . --model codellama:7b
```

#### Generate Tests for Specific Files
```bash
# Generate tests for a single file
ghprai generate-tests src/mymodule.py

# Generate tests for multiple files
ghprai generate-tests src/module1.py src/module2.py

# Generate tests with specific test framework
ghprai generate-tests src/mymodule.py --framework pytest
```

#### Health Check
```bash
# Check if services are running
curl http://localhost:5000/health

# Check Ollama connection
curl http://localhost:11434/api/tags
```

### Method 3: Python API

```python
from ghprai import GitHubPRAgent, OllamaAIAgent

# Initialize the agent
agent = GitHubPRAgent(
    github_token="your_token",
    ollama_url="http://localhost:11434",
    model="codellama:13b"
)

# Analyze code
ai_agent = OllamaAIAgent()
analysis = ai_agent.analyze_code_intelligence(code, "example.py")

# Generate tests
test_code = ai_agent.generate_intelligent_tests(code, "example.py", analysis)

# Review pull request
pr_review = agent.review_pull_request(
    repo_name="owner/repository",
    pr_number=123
)
```

### Method 4: Docker Deployment (COMING SOON...)

#### Using Docker Compose (Easiest)
```bash
# Clone the repository
git clone https://github.com/uppadhyayraj/ai-agent-pr-reviewer
cd ai-agent-pr-reviewer

# Copy environment template
cp .env.example .env
# Edit .env with your GitHub token

# Start services
docker-compose up --build

# Run in background
docker-compose up -d --build
```

#### Using Docker
```bash
# Build the image
docker build -t ghprai:latest .

# Run the container
docker run -d \
  -e GITHUB_TOKEN=your_token \
  -e OLLAMA_URL=http://host.docker.internal:11434 \
  -p 5000:5000 \
  ghprai:latest
```

## Configuration Options

### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `GITHUB_TOKEN` | GitHub Personal Access Token | - | ✅ |
| `OLLAMA_URL` | Ollama server URL | `http://localhost:11434` | No |
| `OLLAMA_MODEL` | Model to use for analysis | `codellama:13b` | No |
| `PORT` | Server port | `5000` | No |
| `LOG_LEVEL` | Logging level | `INFO` | No |
| `MAX_FILE_SIZE` | Max file size to analyze (bytes) | `1048576` | No |
| `WEBHOOK_SECRET` | GitHub webhook secret | - | No |

### Supported AI Models

| Model | Size | Speed | Quality | Use Case |
|-------|------|-------|---------|----------|
| `codellama:7b` | 7B | ⚡⚡⚡ | ⭐⭐ | Development/Testing |
| `codellama:13b` | 13B | ⚡⚡ | ⭐⭐⭐ | Production (Recommended) |
| `codellama:34b` | 34B | ⚡ | ⭐⭐⭐⭐ | High-quality analysis |
| `deepseek-coder` | 6.7B | ⚡⚡⚡ | ⭐⭐⭐ | Code generation focused |
| `magicoder:7b` | 7B | ⚡⚡ | ⭐⭐⭐ | Specialized for code |

## Troubleshooting

### Common Issues

#### 1. Ollama Connection Error
```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Restart Ollama
ollama serve

# Check available models
ollama list
```

#### 2. GitHub Authentication Error
```bash
# Test GitHub token
curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/user

# Verify token permissions in GitHub settings
```

#### 3. Memory/Performance Issues
```bash
# Use smaller model
export OLLAMA_MODEL=codellama:7b

# Increase system resources or use GPU acceleration
export OLLAMA_GPU_LAYERS=35
```

#### 4. Webhook Not Triggering
- Verify webhook URL is accessible from internet
- Check webhook delivery logs in GitHub repository settings
- Ensure webhook secret matches if configured
- Verify server is running and accessible

### Getting Help

- **Check server logs**: `docker-compose logs -f` or check console output
- **Verify configurations**: Ensure all environment variables are set correctly
- **Test components individually**: Use health check endpoints and CLI commands
- **Monitor resource usage**: Ensure sufficient RAM and CPU for chosen model

## Development

### Setup Development Environment

```bash
git clone https://github.com/uppadhyayraj/ai-agent-pr-reviewer
cd ai-agent-pr-reviewer
pip install -e ".[dev]"
```

### Running Tests

```bash
pytest tests/
```

### Code Quality

```bash
black ghprai/
flake8 ghprai/
mypy ghprai/
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Support

- 📚 [Documentation](https://github.com/uppadhyayraj/pr-review-ai-agent/blob/main/GUIDE.md)
- 🐛 [Issues](https://github.com/uppadhyayraj/pr-review-ai-agent/issues)
- 💬 [Discussions](https://github.com/uppadhyayraj/pr-review-ai-agent/discussions)

---
# .env (create this file)
GITHUB_TOKEN=your_github_personal_access_token
OLLAMA_URL=http://localhost:11434
OLLAMA_MODEL=codellama:13b

---
# Setup Instructions:

## 1. Install and Setup Ollama

### Local Installation:
```bash
# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Pull CodeLlama model (recommended for code analysis)
ollama pull codellama:13b

# Alternative models you can use:
# ollama pull codellama:7b      # Faster, less capable
# ollama pull codellama:34b     # More capable, slower
# ollama pull deepseek-coder    # Good for code generation
# ollama pull magicoder         # Specialized for code

# Start Ollama server
ollama serve
```

## 2. Deploy the AI Agent

### Option A: Docker Compose (Recommended)
```bash
docker-compose up --build

# This will:
# - Start Ollama server with CodeLlama model
# - Build and run the PR agent
# - Set up networking between services
```

### Option B: Local Development
```bash
# Start Ollama first
ollama serve

# In another terminal
python pr_agent.py
```

## 3. Test AI Agent Functionality

# Test Ollama connection
curl http://localhost:11434/api/tags

# Test AI agent
curl -X POST http://localhost:5000/test-ai \
  -H "Content-Type: application/json" \
  -d '{"code": "def calculate(a, b): return a + b"}'

## 4. Configure GitHub Webhook
#### Repository Settings → Webhooks → Add webhook
#### Payload URL: https://your-domain.com/webhook
#### Content type: application/json
#### Events: Pull requests
#### Active: ✓

---
# AI Agent Features:

## 🤖 Ollama Integration
- Uses local LLM via Ollama API
- No external API costs or rate limits
- Privacy-focused (code never leaves your infrastructure)
- Supports multiple models (CodeLlama, DeepSeek-Coder, etc.)

## 🧠 Intelligent Code Analysis
- AI-powered complexity assessment
- Function and class identification
- Edge case detection
- Security vulnerability scanning
- Performance optimization suggestions

## 🧪 Smart Test Generation
- Context-aware test creation
- Framework-specific test patterns
- Edge case coverage
- Mocking suggestions for dependencies
- Comprehensive assertion generation

## 📊 AI-Powered Code Review
- Quality assessment scoring
- Specific improvement recommendations
- Best practice enforcement
- Testability analysis

## 🔄 Automated Workflow
1. PR created/updated → Webhook triggered
2. Repository cloned and analyzed
3. AI analyzes changed files for complexity/risks
4. Existing tests discovered and executed
5. Missing tests generated using AI
6. New tests committed to PR branch
7. Comprehensive AI report posted as PR comment

---
# Model Recommendations:

## For Production (High Quality):
OLLAMA_MODEL=codellama:13b        # Best balance of speed/quality
OLLAMA_MODEL=codellama:34b        # Highest quality, slower

## For Development/Testing (Fast):
OLLAMA_MODEL=codellama:7b         # Fastest, lower quality
OLLAMA_MODEL=deepseek-coder:6.7b  # Good alternative

## Specialized Models:
OLLAMA_MODEL=magicoder:7b         # Code-focused
OLLAMA_MODEL=starcoder:15b        # Multi-language support

---
# Advanced Configuration:

## Performance Tuning
#### For faster responses:
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_MAX_LOADED_MODELS=2

## For better GPU utilization:
export OLLAMA_GPU_LAYERS=35

##  Required GitHub Token Permissions:
#### - repo (full repository access)
#### - pull_requests:write
#### - contents:write

##  Test the Setup
#### Create a new PR in your repository
#### Check the webhook delivery in GitHub settings
#### Verify the agent processes the PR and adds comments

---
# Enhanced Features (Optional Extensions):

## Integration with CI/CD
## Monitoring & Logging
#### Add structured logging:
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

## Database Integration

