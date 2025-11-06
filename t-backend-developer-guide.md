---
description: Updated July 2025
---

# T Backend Developer Guide

<figure><img src=".gitbook/assets/agent_t_pfp.png" alt=""><figcaption></figcaption></figure>

**Welcome to T Backend!** Your autonomous multi-agent AI platform is now ready for development. This guide will help you get started with the powerful features available.

### 🎯 What You Can Do

T Backend combines a conversational AI assistant with autonomous agents that can:

* **Research complex topics** and provide detailed analysis
* **Monitor blockchain/crypto markets** automatically
* **Process long-running tasks** in the background
* **Manage knowledge graphs** with temporal memory
* **Generate and edit images** from text descriptions
* **Handle file uploads** and create searchable document collections

### 📜 API Documentation

For complete details on all API endpoints, request formats, and responses, refer to our [Swagger API Docs](https://api.fluxpointstudios.com/docs). This comprehensive documentation will guide you through integrating with the T-Backend efficiently.

### 🔑 Authentication Setup

All API requests require an `api-key` header:

```bash
curl -H "api-key: YOUR_API_KEY_HERE" \
  https://api.fluxpointstudios.com/health
```

**Contact the Flux Point Team** via email: contact@fluxpointstudios.com OR Discord: https://discord.gg/MfYUMnfrJM to get your API key if you don't have one yet.

### 💰 Pricing & Access

**API Access**: 10,000 $AGENT per month

* Payment is processed through **FluidTokens scheduled transactions** via Aquarium
* Subscription includes access to all platform features:
  * Unlimited chat interactions with the AI assistant
  * Background task processing with autonomous agents
  * File upload and vector store creation
  * Image generation and editing capabilities
  * Knowledge graph access and querying
  * Real-time research and blockchain monitoring

**Payment Setup**: Contact the Flux Point Team via email: contact@fluxpointstudios.com OR Discord: https://discord.gg/MfYUMnfrJM for details.

### 🌟 Core Features

#### 1. **Chat with the AI Assistant**

The main chat endpoint provides conversational AI with advanced capabilities:

```python
import requests

response = requests.post(
    "https://api.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY"},
    json={
        "message": "Research the latest developments in blockchain technology",
        "session_id": "my-session-123"  # Optional: maintains conversation context
    }
)

print(response.json()["reply"])
```

**Features:**

* Remembers conversation context within sessions
* Can process images (send `image_data` as base64)
* Uses external tools (web search, data analysis) automatically
* Supports complex research queries

#### 2. **Background Tasks for Heavy Work**

For complex analysis that might take several minutes:

```python
# Start a background task
bg_task = requests.post(
    "https://api.fluxpointstudios.com/background/create",
    headers={"api-key": "YOUR_API_KEY"},
    json={
        "model": "advanced",  # Uses our most powerful AI model
        "input": "Provide a comprehensive analysis of DeFi trends in 2025",
        "analysis_depth": "high"  # For complex analytical tasks
    }
)

task_id = bg_task.json()["id"]

# Poll for results
import time
while True:
    status = requests.get(
        f"https://api.fluxpointstudios.com/background/{task_id}",
        headers={"api-key": "YOUR_API_KEY"}
    ).json()
    
    if status["status"] == "completed":
        print(status["output_text"])
        break
    elif status["status"] == "failed":
        print("Task failed:", status.get("error"))
        break
    
    time.sleep(10)  # Check every 10 seconds
```

#### 3. **Autonomous Agent System**

The platform includes specialized autonomous agents that work behind the scenes:

**Available Agents:**

* **Research Agent**: Web research and fact-finding
* **Blockchain Agent**: Crypto market monitoring
* **Consolidation Agent**: Knowledge graph optimization
* **Maintenance Agent**: System health monitoring

These agents automatically enhance your chat interactions and can be triggered through background tasks for specific analysis types.

#### 4. **File Upload & Search**

Create searchable document collections:

```python
# 1. Create a vector store for your documents
store = requests.post(
    "https://api.fluxpointstudios.com/files/vector-stores",
    headers={"api-key": "YOUR_API_KEY"},
    json={
        "name": "my-documents",
        "description": "My company's documentation"
    }
).json()

store_id = store["id"]

# 2. Upload a file
file_response = requests.post(
    "https://api.fluxpointstudios.com/files/upload-url",
    headers={"api-key": "YOUR_API_KEY"},
    params={"url": "https://example.com/whitepaper.pdf"}
).json()

# 3. Add file to vector store
requests.post(
    f"https://api.fluxpointstudios.com/files/vector-stores/{store_id}/files",
    headers={"api-key": "YOUR_API_KEY"},
    json={"file_id": file_response["file_id"]}
)

# 4. Now you can search across your documents in chat!
search_response = requests.post(
    "https://api.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY"},
    json={"message": "Search my documents for information about tokenomics"}
)
```

#### 5. **Image Generation & Editing**

Generate and edit images from text:

```python
# Generate a new image
image_response = requests.post(
    "https://api.fluxpointstudios.com/images/generate",
    headers={"api-key": "YOUR_API_KEY"},
    json={
        "prompt": "A futuristic AI agent working with blockchain data",
        "size": "1024x1024",
        "quality": "high"
    }
).json()

if image_response["success"]:
    # Images are returned as base64 data
    base64_images = image_response["images"]
    print(f"Generated {len(base64_images)} images!")
```

#### 6. **Knowledge Graph Access**

Explore and query the temporal knowledge graph:

```python
# List accessible graphs
graphs = requests.get(
    "https://api.fluxpointstudios.com/graph/accessible",
    headers={"api-key": "YOUR_API_KEY"}
).json()

# Query a specific graph
if graphs["graphs"]:
    graph_id = graphs["graphs"][0]["id"]
    results = requests.get(
        f"https://api.fluxpointstudios.com/graph/{graph_id}/query?query=blockchain",
        headers={"api-key": "YOUR_API_KEY"}
    ).json()
```

### 🔧 Common Use Cases

#### Research & Analysis

```python
# Deep research on a complex topic
research_task = requests.post(
    "https://api.fluxpointstudios.com/background/create",
    headers={"api-key": "YOUR_API_KEY"},
    json={
        "input": "Research and analyze the impact of AI agents on financial markets",
        "analysis_depth": "high"
    }
)
```

#### Document Q\&A System

```python
# Upload company docs to vector store, then ask questions
response = requests.post(
    "https://api.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY"},
    json={"message": "Based on our uploaded documentation, what is our API rate limiting policy?"}
)
```

#### Market Monitoring

```python
# Ask about crypto trends (uses blockchain agent automatically)
market_response = requests.post(
    "https://api.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY"},
    json={"message": "What are the current market conditions for Cardano and recent developments?"}
)
```

#### Multi-turn Image Editing

```python
# Generate an image, then refine it
initial_image = requests.post(
    "https://api.fluxpointstudios.com/images/generate",
    headers={"api-key": "YOUR_API_KEY"},
    json={"prompt": "A modern office workspace"}
).json()

# Edit the generated image
if initial_image["success"]:
    edited_image = requests.post(
        "https://api.fluxpointstudios.com/images/edit",
        headers={"api-key": "YOUR_API_KEY"},
        json={
            "prompt": "Add some plants and natural lighting",
            "input_images": initial_image["images"]
        }
    )
```

### 📊 Monitoring Your Usage

```python
# Check background task history
tasks = requests.get(
    "https://api.fluxpointstudios.com/background/",
    headers={"api-key": "YOUR_API_KEY"}
).json()

print(f"You have {len(tasks['responses'])} background tasks")

# View your vector stores
stores = requests.get(
    "https://api.fluxpointstudios.com/files/vector-stores",
    headers={"api-key": "YOUR_API_KEY"}
).json()

for store in stores:
    print(f"Store: {store['name']} (ID: {store['id']})")

# Check system health
health = requests.get(
    "https://api.fluxpointstudios.com/health",
    headers={"api-key": "YOUR_API_KEY"}
).json()
```

### 🛠️ Advanced Features

#### Background Task Types

You can create specialized background tasks for different agent types:

```python
# Research-focused task
research_task = requests.post(
    "https://api.fluxpointstudios.com/background/create",
    headers={"api-key": "YOUR_API_KEY"},
    json={
        "input": "research latest blockchain trends",
        "task_type": "research"
    }
)

# Consolidation task for knowledge optimization
consolidation_task = requests.post(
    "https://api.fluxpointstudios.com/background/create", 
    headers={"api-key": "YOUR_API_KEY"},
    json={
        "input": "consolidate recent knowledge and optimize storage",
        "task_type": "consolidation"
    }
)
```

#### Streaming Results

For real-time updates on long-running tasks:

```python
import requests

# Create a background task
task = requests.post(
    "https://api.fluxpointstudios.com/background/create",
    headers={"api-key": "YOUR_API_KEY"},
    json={"input": "Analyze market trends", "stream": True}
).json()

# Stream the results
response = requests.get(
    f"https://api.fluxpointstudios.com/background/{task['id']}/stream",
    headers={"api-key": "YOUR_API_KEY"},
    stream=True
)

for line in response.iter_lines():
    if line:
        print(line.decode('utf-8'))
```

### 🚨 Important Notes

* **Rate Limits**: Be mindful of API usage, especially with background tasks
* **Session IDs**: Use consistent session IDs to maintain conversation context
* **Background Tasks**: Use for analysis that takes more than 30 seconds
* **File Limits**: Maximum 25MB per file upload
* **Model Choice**: Use "advanced" model for complex reasoning tasks
* **API Keys**: Keep your API keys secure and never commit them to version control

### 🆘 Troubleshooting

**Common Issues:**

1. **401 Unauthorized**: Check your API key is correct and active
2. **403 Forbidden**: You may not have access to that graph/resource
3. **408 Timeout**: Use background tasks for long-running operations
4. **413 File Too Large**: Files must be under 25MB
5. **422 Unprocessable Entity**: Check your request format matches the API specification

**Getting Help:**

* Check the interactive API docs at `/docs` endpoint
* Review system health at `GET /health`
* Contact the Flux Point Team via email: contact@fluxpointstudios.com OR Discord: https://discord.gg/MfYUMnfrJM for API key issues or permission problems

**Debug Tips:**

```python
# Always check response status
response = requests.post(url, headers=headers, json=data)
if response.status_code != 200:
    print(f"Error {response.status_code}: {response.text}")
else:
    result = response.json()
```

### 📚 Additional Resources

* **API Documentation**: Visit `/docs` for interactive API explorer

### 🎉 You're Ready!

Your T Backend autonomous agent platform is ready for:

* **Intelligent conversations** with memory
* **Deep research and analysis**
* **Automated monitoring** and insights
* **Document processing** and search
* **Image generation** and editing
* **Knowledge graph exploration**

Start with simple chat requests and gradually explore the advanced features. The autonomous agents work behind the scenes to enhance your interactions automatically!

**Quick Start Checklist:**

* [ ] Get your API key from administrator
* [ ] Test with `GET /health` endpoint
* [ ] Try a simple chat request
* [ ] Upload a document and create a vector store
* [ ] Create your first background task
* [ ] Generate an image
* [ ] Explore the knowledge graphs

Welcome to the future of autonomous AI development! 🚀
