# End-to-End Chest Cancer Classification using MLflow, DVC & Azure

A deep learning project for chest cancer classification deployed on Azure Cloud with CI/CD pipeline using GitHub Actions.


## 📋 Table of Contents
- [Project Workflows](#project-workflows)
- [MLflow Integration](#mlflow-integration)
- [DVC Integration](#dvc-integration)
- [Azure Cloud Deployment](#azure-cloud-deployment)
- [Local Development](#local-development)
- [Technologies Used](#technologies-used)

## 🔄 Project Workflows

1. Update `config.yaml`
2. Update `secrets.yaml` [Optional]
3. Update `params.yaml`
4. Update the entity
5. Update the configuration manager in `src/config`
6. Update the components
7. Update the pipeline 
8. Update `main.py`
9. Update `dvc.yaml`

## 📊 MLflow Integration

### What is MLflow?
- Production-grade experiment tracking
- Comprehensive logging & tagging
- Model versioning and management

### Resources
- [MLflow Documentation](https://mlflow.org/docs/latest/index.html)
- [MLflow Tutorial Playlist](https://youtube.com/playlist?list=PLkz_y24mlSJZrqiZ4_cLUiP0CBN5wFmTb&si=zEp_C8zLHt1DzWKK)

### DagHub Setup
```bash
# Export environment variables
export MLFLOW_TRACKING_URI=https://dagshub.com/pantpujan017/Chest-Cancer-Classification-using-MLFlow-and-DVC.mlflow
export MLFLOW_TRACKING_USERNAME=pantpujan017
export MLFLOW_TRACKING_PASSWORD=<your-dagshub-token>
```

### Run MLflow UI Locally
```bash
mlflow ui
```

## 📦 DVC (Data Version Control)

### What is DVC?
- Lightweight experiment tracker
- Pipeline orchestration
- Data versioning

### DVC Commands
```bash
# Initialize DVC
dvc init

# Reproduce pipeline
dvc repro

# View pipeline DAG
dvc dag
```

## ☁️ Azure Cloud Deployment

### Architecture Overview
```
GitHub Repository → GitHub Actions → Azure Container Registry → Azure VM (Self-hosted Runner)
```

### Prerequisites
- Azure Account
- GitHub Account
- Docker installed locally

---

## 🛠️ Azure Deployment Setup

### Step 1: Create Azure Container Registry (ACR)

1. **Login to Azure Portal**
```bash
   az login
```

2. **Create Resource Group**
```bash
   az group create --name chest-cancer-rg --location southeastasia
```

3. **Create Azure Container Registry**
```bash
   az acr create --resource-group chest-cancer-rg \
     --name chestcanceracr --sku Basic \
     --location southeastasia
```

4. **Enable Admin Access**
```bash
   az acr update -n chestcanceracr --admin-enabled true
```

5. **Get ACR Credentials**
```bash
   az acr credential show --name chestcanceracr
```

   Save these values:
   - Login Server: `chestcanceracr.azurecr.io`
   - Username: `chestcanceracr`
   - Password: (from output)

---

### Step 2: Create Azure Virtual Machine

1. **Create Ubuntu VM**
```bash
   az vm create \
     --resource-group chest-cancer-rg \
     --name chest-cancer-vm \
     --image Ubuntu2204 \
     --size Standard_B2s \
     --admin-username azureuser \
     --generate-ssh-keys \
     --public-ip-sku Standard
```

2. **Open Required Ports**
```bash
   # Allow SSH (22)
   az vm open-port --port 22 --resource-group chest-cancer-rg --name chest-cancer-vm --priority 1000
   
   # Allow HTTP (80)
   az vm open-port --port 80 --resource-group chest-cancer-rg --name chest-cancer-vm --priority 1001
   
   # Allow HTTPS (443)
   az vm open-port --port 443 --resource-group chest-cancer-rg --name chest-cancer-vm --priority 1002
   
   # Allow Application (8080)
   az vm open-port --port 8080 --resource-group chest-cancer-rg --name chest-cancer-vm --priority 1003
```

3. **Get VM Public IP**
```bash
   az vm show -d -g chest-cancer-rg -n chest-cancer-vm --query publicIps -o tsv
```
   Public IP: `20.189.120.210`

---

### Step 3: Configure VM

1. **SSH into VM**
```bash
   ssh azureuser@20.189.120.210
```

2. **Install Docker**
```bash
   # Update system
   sudo apt-get update -y
   
   # Install Docker
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   
   # Add user to docker group
   sudo usermod -aG docker $USER
   newgrp docker
   
   # Verify installation
   docker --version
```

---

### Step 4: Setup GitHub Self-Hosted Runner

1. **In GitHub Repository:**
   - Go to `Settings` → `Actions` → `Runners`
   - Click `New self-hosted runner`
   - Select `Linux` and `x64`

2. **On Azure VM, run the provided commands:**
```bash
   # Create directory
   mkdir actions-runner && cd actions-runner
   
   # Download runner
   curl -o actions-runner-linux-x64-2.XXX.X.tar.gz -L https://github.com/actions/runner/releases/download/vX.XXX.X/actions-runner-linux-x64-2.XXX.X.tar.gz
   
   # Extract
   tar xzf ./actions-runner-linux-x64-2.XXX.X.tar.gz
   
   # Configure (use token from GitHub)
   ./config.sh --url https://github.com/pantpujan017/Chest-Cancer-Classification-using-MLFlow-and-DVC --token YOUR_TOKEN
   
   # Install as service
   sudo ./svc.sh install
   sudo ./svc.sh start
```

3. **Verify runner is online** in GitHub Settings → Actions → Runners

---

### Step 5: Configure GitHub Secrets

Go to your repository → `Settings` → `Secrets and variables` → `Actions`

Add these secrets:

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `AZURE_REGISTRY_LOGIN_SERVER` | `chestcanceracr.azurecr.io` | ACR Login Server |
| `AZURE_REGISTRY_USERNAME` | `chestcanceracr` | ACR Username |
| `AZURE_REGISTRY_PASSWORD` | `<from Step 1>` | ACR Password |
| `AZURE_VM_IP` | `20.189.120.210` | VM Public IP |

---

### Step 6: GitHub Actions Workflow

The `.github/workflows/main.yaml` file contains:

**Pipeline Stages:**
1. **Continuous Integration** - Lint & Test
2. **Continuous Delivery** - Build & Push Docker image to ACR
3. **Continuous Deployment** - Deploy to Azure VM

**Workflow triggers on:**
- Push to `main` branch
- Excludes `README.md` changes

---

## 💻 Local Development

### Prerequisites
```bash
# Python 3.8+
python --version

# Install dependencies
pip install -r requirements.txt
```

### Run Locally
```bash
# Run the application
python app.py
```

Access at: `http://localhost:8080`

### Build Docker Image Locally
```bash
# Build image
docker build -t chest-cancer-app .

# Run container
docker run -p 8080:8080 chest-cancer-app
```

---

## 🧪 Testing the Deployment

1. **Check if container is running:**
```bash
   ssh azureuser@20.189.120.210
   docker ps
```

2. **View container logs:**
```bash
   docker logs chest-cancer-app
```

3. **Access the application:**
```
   http://20.189.120.210:8080
```

---

## 🔧 Troubleshooting

### Pipeline Failures
```bash
# Check GitHub Actions logs
# Go to Actions tab → Click on failed workflow

# Check runner status
ssh azureuser@20.189.120.210
cd ~/actions-runner
./run.sh
```

### Docker Issues on VM
```bash
# Restart Docker
sudo systemctl restart docker

# Check Docker status
sudo systemctl status docker

# View container logs
docker logs chest-cancer-app
```

### Container Not Starting
```bash
# Remove old container
docker stop chest-cancer-app
docker rm chest-cancer-app

# Pull latest image
docker pull chestcanceracr.azurecr.io/chest-cancer:latest

# Run new container
docker run -d -p 8080:8080 --name chest-cancer-app chestcanceracr.azurecr.io/chest-cancer:latest
```

---

## 📁 Project Structure
```
├── .github/
│   └── workflows/
│       └── main.yaml          # CI/CD pipeline
├── config/
│   └── config.yaml            # Configuration files
├── model/
│   └── model.h5               # Trained model
├── research/                  # Jupyter notebooks
├── src/
│   └── cnnClassifier/         # Source code
├── templates/                 # HTML templates
├── app.py                     # Flask application
├── Dockerfile                 # Docker configuration
├── requirements.txt           # Python dependencies
├── dvc.yaml                   # DVC pipeline
└── README.md                  # Documentation
```

---

## 🛠️ Technologies Used

- **Machine Learning:** TensorFlow, Keras
- **Experiment Tracking:** MLflow, DagHub
- **Data Versioning:** DVC
- **Web Framework:** Flask
- **Containerization:** Docker
- **Cloud Platform:** Microsoft Azure (ACR, VM)
- **CI/CD:** GitHub Actions
- **Version Control:** Git, GitHub

---

## 📊 Model Information

- **Architecture:** CNN (Convolutional Neural Network)
- **Task:** Binary Classification (Normal vs Adenocarcinoma)
- **Input:** Chest CT Scan Images
- **Framework:** TensorFlow/Keras
- **Model Size:** ~56 MB

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Pujan Pant**
- GitHub: [@pantpujan017](https://github.com/pantpujan017)
- Repository: [Chest-Cancer-Classification-using-MLFlow-and-DVC](https://github.com/pantpujan017/Chest-Cancer-Classification-using-MLFlow-and-DVC)

---

## 🙏 Acknowledgments

- MLflow documentation and community
- DVC for data versioning
- Azure for cloud infrastructure
- TensorFlow/Keras for deep learning framework

---

## 📞 Support

For issues and questions:
- Open an issue on GitHub
- Check the [Troubleshooting](#-troubleshooting) section

---

**⭐ If you find this project helpful, please give it a star!**
