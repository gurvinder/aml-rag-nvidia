# Streamline AML Investigation Workflows with RAG and NVIDIA

Build anti-money laundering investigation workflows with RAG and NVIDIA models on Red Hat AI, with built-in inference, governance, observability, and more.

## Table of contents

- [Detailed Description](#detailed-description)
  - [See it in Action](#see-it-in-action)
  - [Architecture Diagrams](#architecture-diagrams)
- [Requirements](#requirements)
  - [Minimum Hardware Requirements](#minimum-hardware-requirements)
  - [Minimum Software Requirements](#minimum-software-requirements)
  - [Required User Permissions](#required-user-permissions)
- [Deploy](#deploy)
  - [Prerequisites](#prerequisites)
  - [Install](#install)
  - [Delete](#delete)
- [References](#references)
- [Tags](#tags)

## Detailed description

Anti-money laundering (AML) is the set of processes financial institutions use to detect suspicious activity, investigate potential financial crime, and support regulatory compliance. AML investigations require analysts to move quickly through large volumes of information while still following strict internal procedures and compliance requirements. Investigators often need to pull together context from multiple sources, including case notes, internal policies, sanctions guidance, procedural documentation, and other enterprise records. As a result, even routine investigative tasks can become time-consuming because the work depends on finding and connecting the right information across fragmented systems.

This quickstart demonstrates how a Retrieval-Augmented Generation (RAG) application can help accelerate anti-money laundering investigation workflows by grounding AI responses in trusted enterprise content. Rather than generating answers from model knowledge alone, the application retrieves relevant context at query time and uses it to support more informed, document-based responses. This can help analysts more quickly review policies, gather investigation context, summarize relevant materials, and support follow-up work around suspicious activity cases.

Built as a customized version of the NVIDIA RAG blueprint for Red Hat AI, this application shows how AML investigation workflows can run with NVIDIA models on an enterprise-ready AI platform. It highlights how teams can adapt a proven RAG pattern for hybrid cloud environments while aligning with operational needs such as governance, observability, and scalable model serving.

### See it in action 

### Architecture diagrams

## Requirements

### Minimum hardware requirements 

#### vLLM Model Requirements

- Llama-3_3-Nemotron-Super-49B-v1_5 (LLM): 4 NVIDIA GPUs with 48GB VRAM (see NVIDIA docs for more information)
- NVIDIA-Nemotron-Nano-12B-v2-VL-BF16 (VLM): 1 NVIDIA GPU with 48GB VRAM (see NVIDIA docs for more information)
- llama-nemotron-embed-1b-v2 (Embedding): 1 NVIDIA GPU with 8GB VRAM (see NVIDIA docs for more information)
- llama-nemotron-rerank-1b-v2 (Reranking): 1 NVIDIA GPU with 8GB VRAM (see NVIDIA docs for more information)

**Note**: This quickstart was tested on a single node with 8 NVIDIA H100 GPUs.

#### Storage

- At least 200 GB of disk space is needed across node(s) for model downloads, images, and vector data.

### Minimum software requirements

- Red Hat OpenShift Container Platform v4.21
- Red Hat OpenShift AI v3.3
- NVIDIA GPU Operator (tested w/ v25.10.)
- S3-compatible object storage
- Helm CLI: 3.x
- OpenShift Client CLI
- NGC API key
  - With appropriate access to hosted models leveraged in deployment
- HuggingFace token for model weight downloads

### Required user permissions

- cluster-admin or SCC management rights
  - Required to grant the anyuid Security Context Constraint to three service accounts: default, <release>-nv-ingest, and ingestor-server as part of the helm deployment.

## Deploy

The following instructions will easily deploy the quickstart to your Red Hat AI environment using helm charts. 

### Prerequisites

- OpenShift cluster
- OpenShift cluster has GPUs available
- OpenShift AI has a datasciencecluster resource with kserve and dashboard resources, at minimum, set to managed.
- The NVIDIA GPU Operator is installed and configured with a ClusterPolicy to configure the driver
- OpenShift Data Foundation: NooBaa object storage with openshift-storage.noobaa.io StorageClass

### Install

1. git clone quickstart repository
```
git clone https://github.com/rh-ai-quickstart/aml-rag-nvidia
```

2. cd into the quickstart directory
```
cd aml-rag-nvidia
```

3. Ensure you are logged into your OpenShift AI cluster as a cluster-admin user, such as `kube:admin` or `system:admin`:
```
oc whoami
```

4. Set environment variables for secrets:

```
export HF_TOKEN={insert_token}
export NGC_API_KEY=”nvapi-...”
```

5. Run helm install commands

Deploy model-serving chart
```
helm install model-serving ./charts/model-serving --set secret.hf_token=$HF_TOKEN --namespace rag --create-namespace
```

Deploy ingest chart
```
helm dependency update ./charts/ingest
helm install ingest ./charts/ingest --set nvidiaApiKey.password=$NGC_API_KEY --namespace rag
```
Deploy rag-server chart
```
helm install rag-server ./charts/rag-server --namespace rag
```

Deploy frontend
```
helm install frontend ./charts/frontend --namespace rag
```

#### Verify installation

Check all deployed pods are running
```
oc get pods -n rag
```

### Usage



### Delete

Delete all helm deployments
```
helm uninstall ingest model-serving rag-server frontend -n rag
```

Delete all PVCs in rag namespace
```
oc get pvc -n rag
oc delete pvc --all -n rag
```

(Optional) Delete the entire namespace
```
oc delete namespace rag
```

## References 

- [vLLM](https://vllm.ai/): The High-Throughput and Memory-Efficient inference and serving engine for LLMs.
- [NVIDIA Nemotron](https://developer.nvidia.com/nemotron): a family of open models with open weights, training data, and recipes, delivering leading efficiency and accuracy for building specialized AI agents.
- [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html): uses the operator framework within Kubernetes to automate the management of all NVIDIA software components needed to provision
  GPU.
- [NVIDIA RAG Blueprint](https://github.com/NVIDIA-AI-Blueprints/rag): The official RAG blueprint from which this quickstart is based from.

## Tags

- **Product**: Red Hat AI Enterprise
- **Use case**: Anti-money laundering
- **Industry**: Adopt and scale AI