# Overview
This project presents an AI-powered stock market data AI powered application built with Spring Boot and Spring AI, designed to run as a service on Azure AKS kubernetes, connecting to Azure AI OpenAI models provisioned on Azure AI Foundry.
1. This solution fetches share prices from a public API ([twelvedata.com](https://support.twelvedata.com/)) and stores it on Azure AI Search to support RAG (Retrieval Augmented Generation).
2. [AI Agent](https://www.pega.com/agentic-ai?utm_source=google&utm_medium=cpc&utm_campaign=G_DACH_NonBrand_AgenticAI_CE_Exact_(CPN-111052)_EN&utm_term=agentic%20ai&gloc=9189123&utm_content=pcrid%7C731149222736%7Cpkw%7Ckwd-1490950831424%7Cpmt%7Ce%7Cpdv%7Cc%7C&gad_source=1&gclid=Cj0KCQjwkZm_BhDrARIsAAEbX1HB619ps6TWXRDu9QIvvLbQJ98faUewIqpHV6y2beDb_ayi1qLpbNoaAvxMEALw_wcB&gclsrc=aw.ds#p-c6e30b6a-0e50-4df1-bc14-357576efd817) functionality to letting the AI model autonomously perform tasks, like sending out messages without user intervention, depending on the inquiry and AI response. However, it’s important to differentiate between agentic AI and AI agents. Essentially, agentic AI is the framework; AI agents are the building blocks within the framework.
3. **NEW** : Introduce [Model Context Protocol](https://modelcontextprotocol.io/introduction) (MCP), introduce an agent and complex workflows on top of LLMs. Go beyound general knowledge-based question-answering pattern and enhance AI LLM agent connect to search engines, databases and file systems.

- **NOTE!!!** *development ongoing*
## Why RAG
Large language models (LLMs) like ChatGPT are trained on public internet data that was available at the point in time when they were trained ([See the knowledge cut-off dates for various large language models (LLM)](https://github.com/HaoooWang/llm-knowledge-cutoff-dates?tab=readme-ov-file)). They can answer questions related to the data they were trained on.
Retrieval Augmented Generation (RAG) is a smart way to improve how AI systems answer questions or create content by combining two steps: retrieving useful information and generating responses. 
Instead of just relying on what the AI knows, RAG pulls in extra data that helps the system understand the question better and provide more accurate, context-aware answers.
This demo provides that extra data by pulling stocks, forex and other financial assets from [twelvedata.com](https://twelvedata.com/) over REST API and stores that data as 
vectors (text data converted to number sequences using an embedding model) on [Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search). 


## Benefits and Applications of RAG
- Improved privacy: You can use data that the AI wasn’t trained on, meaning you don’t have to worry about the AI knowing sensitive information beforehand.
- Better context: The system can pull in relevant information to understand the user’s question more deeply. 
- Higher accuracy: By looking up information, RAG helps reduce mistakes (when AI makes up things) by using real facts.
- Flexible applications: It can be used for various tasks like answering questions and more importantly autonomously performing tasks on behalf of a user, aka the [AI Agent pattern](https://blog.whiteprompt.com/mastering-ai-patterns-architectures-knowledge-systems-and-multi-agent-approaches-386064c4bc5a). Tools in this category can be used to take action in a software system, such as sending an email, creating a new record in a database, submitting a form, or triggering a workflow.
  The goal is to automate tasks that would otherwise require human intervention or explicit programming.

<img title="Retrieval Augmented Generation (RAG) technique" alt="Alt text" src="/images/rag.png">

## What is on this project
- Terraform configuration files to provisioning all Azure infrastructure components needed to run this demo in its full context. That includes a full Azure Kubernetes AKS cluster and [Azure AI](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-ai-foundry) Foundry HUB, project and finally deploying gpt-4 LLM model.
- [SpringBoot AI](https://docs.spring.io/spring-ai/reference/index.html) empowered SpringBoot application:
  - Connecting to Azure AI Foundry
  - Connecting to Azure Search VectorStore to loading stock prices growth from [TwelveData](https://twelvedata.com/) to supporting AI RAG.
  - Exposing REST API endpoints to invoking AI ChatBot logic underneath.
  - Misc integration tests to accessing Semantic searches and evaluating LLM responses on [Relevancy and Factual Accuracy](https://www.evidentlyai.com/llm-guide/llm-as-a-judge). 
- Gradle scripts build to compile, test and package to a deployable archive.
- Docker and HELM chart artifacts to providing Kubernetes manifests and deploy this solution onto Azure AKS. 
- CI/CD ADO Azure DevOps Pipeline - Fully automated build package and deployment with Azure DevOps.
- Enterprise-Grade Security - Uses OAuth2 and API Gateway for authentication.
- Scalable & Cloud-Native - Deployable on Azure App Service or Kubernetes.
- Centralized Logging (Azure Monitor, Application Insights)
- Monitoring & Alerts (Azure Monitor, Prometheus/Grafana)
- Security Best Practices (Key Vault for secrets, RBAC, Network Policies)

## Terraform IaC provision Azure AKS cluster and Azure AI HUB and project
1. Create an Azure account with an active subscription. You can [create an account for free](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account?icid=azurefreeaccount&WT.mc_id=A261C142F).
2. [Install and configure Terraform](https://learn.microsoft.com/en-us/azure/developer/terraform/quickstart-configure)
3. Run the following commands...
```
cd ./terraform-manifests
terraform init
terraform validate
terraform plan -out tfplan
terraform apply "tfplan"
```
Check the outputs and assign the Azure OpenAI key, endpoint and model on application.yml.

## Architecture
### Tech Stack
- Spring Boot (Microservices)
- Spring AI (Azure OpenAI API integration)
- Azure AI foundry OpenAI (GPT-based chat responses)
- Terraform (Infrastructure as Code)
- Azure DevOps Pipelines (CI/CD automation)
- Helm & Kubernetes (Optional deployment)
## GitHub repository structure, including:
- Java components (Spring Boot chatbot API)
- Terraform scripts (to create an AKS cluster)
- Gradle build file (for dependencies and build tasks)
- Azure DevOps pipeline (CI/CD for build, package, and deploy)
- Kubernetes manifests (Deployment and LoadBalancer Service)

```
spring-azure-ai/
│── .github/                                # GitHub workflows for CI/CD (optional)
│── terraform-manifests/                    # Terraform configurations for AKS
│   ├── 01-versions.tf                      # Terraform providers and versions
│   ├── 02-variables.tf                     # Configurable input variables
│   ├── 03-locals.tf                        # Local value blocks for composing names and common labeling
│   ├── 04-resource-group.tf                # Master Azure ARM resource group to hosting all Azure infrastructure components
│   ├── 05-aks-versions-datasource.tf       # Datasource to get Latest Azure AKS latest Version
│   ├── 06-aks-administrators-azure-ad.tf   # Create Azure AD Group in Active Directory for AKS Admins
│   ├── 07-log-analytics-workspace.tf       # Log Analytics workspace to integrate Application insights telemetry data: enabling comprehensive observability of springboot applications running on AKS. 
│   ├── 08-external-datasource.tf           # Call SSH key generator script to providing AKS PKI pub and private key pair.
│   ├── 09-aks-cluster.tf                   # Provision AKS cluster, node pool and network profile.
│   ├── 10-ai-foundry-project.tf            # Provision Azure AI Foundry HUB, project and deploy LLM model.
│   ├── 11-outputs.tf                       # Capture all outputs from Terraform provisioning process, like Azure AI Service API keys and endpoints get configured on SpringBoot application properties.
│── src/
│   ├── main/java/com/agilesolutions/chatbot/
│   │   ├── config/                # Spring Boot configuration (AI, security, etc.)
│   │   ├── dto/                   # Data Transfer Objects
│   │   ├── model/                 # Domain models (Stock, Share, etc.)
│   │   ├── repository/            # H2 in memory DB repository
│   │   ├── rest/                  # REST API controllers (chat and AI agent endpoints)
│   │   ├── schedule/              # Scheduled jobs to load share prices on Azure Search VectorDatabase
│   │   ├── service/               # Business logic for AI processing
│   │   ├── tools/                 # AI autonomously invoked tools like sending out WhatsUp message, inquiring actual stock prices
│   ├── resources/
│   │   ├── application.yml        # Spring Boot configurations
│   │   ├── prompts/               # AI prompts for chatbot
│── test/java/com/agilesolutions/chatbot/
│   ├── service/                   # Unit tests for services
│   ├── controller/                # API integration tests
│── helm/                          # Helm charts for Kubernetes deployment
│   ├── Chart.yaml                 # Helm metadata
│   ├── values.yaml                # Configurable Helm values
│── kubernetes/                    # Kubernetes manifests
│   ├── deployment.yaml            # Deployment configuration
│   ├── service.yaml               # LoadBalancer Service
│── build.gradle                   # Gradle build file for dependencies and packaging
│── azure-pipelines.yml            # Azure DevOps pipeline for CI/CD
│── Dockerfile                      # Docker configuration
│── docker-compose.yml              # Local environment setup
│── README.md                       # Project overview & setup instructions
│── LICENSE                         # Open-source license
```

# Logging, Monitoring (*Azure AI search production-ready*)
Collect telemetry data for search traffic analytics. Search traffic analytics is a pattern for collecting telemetry about user interactions with your Azure AI Search application, such as user-initiated clickstream events and keyboard inputs. Using this information, you can determine the effectiveness of your search solution, including clickthrough rate and which query inputs yield zero results.

- Add a telemetry client
- Modify a search request to include a correlation Id that maps search results to user actions
- Create and send a custom event to [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/create-workspace-resource?tabs=portal) and use the visualization and reporting tools to view event data. Application Insights integrates with Log Analytics and sends telemetry to a common Log Analytics workspace. This setup provides full access to Log Analytics features, consolidates logs in one location.
- Add Gradle dependency Application Insights to instrument application code. 
- Add Application Insights resource with Terraform and let it integrate with the Log Analytics workspace configured by Terraform.

## Expose Prometheus Metrics in Spring Boot
- Add Actuator + Micrometer dependencies

## Accessing metrics
```
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090
open http://localhost:9090
```

## Secure IA ChatBot
- Store API keys, DB credentials securely.
- Enable Key Vault in Spring Boot.
```
az keyvault create --name "springAIKeyVault" --resource-group "spring-ai-chatbot-rg"
az keyvault secret set --vault-name "springAIKeyVault" --name "OpenAIKey" --value "<your-key>"
```

## Implement Role-Based Access Control (RBAC)
Restrict chatbot API access using Azure Managed Identity.

Assign RBAC roles:
```
az role assignment create --assignee <service-principal-id> --role "Reader" --scope /subscriptions/<subscription-id>

Update ChatController.java to restrict access:
@PreAuthorize("hasAuthority('SCOPE_chatbot.read')")
@PostMapping
public String chat(@RequestBody String userInput) {
    // Chat logic here
}
```

##  Deploy CI/CD Updates
Modify azure-pipelines.yml to include security scans & Prometheus monitoring:
- AI Chatbot with Logging, Monitoring & Security
- Azure DevOps CI/CD with Terraform & Helm
- Observability with Prometheus, Grafana, and Azure Monitor
```
steps:
  - script: echo "Running security scan..."
  - script: ./gradlew check
  - script: |
      az aks get-credentials --resource-group spring-ai-chatbot-rg --name spring-ai-aks
      helm upgrade --install spring-ai-chatbot helm/
  - script: echo "Deploying monitoring..."
  - script: helm upgrade --install prometheus prometheus-community/kube-prometheus-stack
```


# How This Works:
- User sends a chat request to the Spring Boot Chatbot API.
- API forwards the request to Azure OpenAI (GPT) for intelligent response generation.
- The response is stored in Azure Cosmos DB for context-awareness.
- Azure Functions handle event-driven automation (e.g., analytics, logging).
- The chatbot API is deployed on Azure App Service or Kubernetes for scalability.
- CI/CD Pipeline automates builds, tests, and deployments using GitHub Actions, Azure DevOps, and Terraform.

## start ollama locally
- docker compose -f docker-compose.yml
- docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
- docker logs -f ollama
- docker exec -ti ollama ollama run llama2
- Ask some question: >>>> who is the current PM in Netherlands
- curl -X POST http://localhost:11434/api/generate -d '{ "model": "llama2","prompt":"List down 5 best java frameworks"

## Evalutation testing
Evaluating the generated content to ensure the AI models do not generate hallucinated responses. Spring AI Evaluators to test LLM help checking the relevance and factual accuracy of the LLM response, namely RelevanceEvaluator and FactCheckingEvaluator.
- RelevancyEvaluator: JUnit test that performs a RAG query over a Twelvedata share document loaded into a Vector Store and then evaluates if the response is relevant to the user text
- FactCheckingEvaluator: Junit tests to detect and reduce hallucinations in AI outputs by verifying if a given statement (claim) is logically supported by the provided context (document).

## References
The Spring AI project aims to streamline the development of applications that incorporate artificial intelligence functionality without unnecessary complexity.
Azure AI Foundry provides a unified platform for enterprise AI operations, model builders, and application development. 
- [TwelveData: Financial Data that drives your success](https://twelvedata.com/)
- [About Spring AI](https://docs.spring.io/spring-ai/reference/index.html)
- [About Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-ai-foundry)
- [Azure AI Foundary Retrieval augmented generation and indexes](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/retrieval-augmented-generation)
- [How to build and consume vector indexes in Azure AI Foundry portal](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/index-add)
- [What's Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
- [Create an Azure AI Foundry hub with Terraform](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/create-hub-terraform?tabs=azure-cli)
- [Quickstart: Create an Azure AI services resource using Terraform](https://learn.microsoft.com/en-us/azure/ai-services/create-account-terraform?tabs=azure-cli)
- [Quickstart: Deploy Azure AI Search service using Terraform](https://learn.microsoft.com/en-us/azure/search/search-get-started-terraform)
- [Terraform Azure AI Hub module](https://registry.terraform.io/modules/Azure/avm-res-machinelearningservices-workspace/azurerm/latest/examples/private_ai_studio)
- [Terraform Application Insights in workspace mode](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/application_insights#example-usage---workspace-mode)
- [Azure AI Search traffic analytics](https://learn.microsoft.com/en-us/previous-versions/azure/search/search-traffic-analytics?tabs=visual-studio-telemetry-client%2Cdotnet-correlation%2Cdotnet-properties%2Cdotnet-custom-events)
- [Azure App insights for Java Springboot application](https://learn.microsoft.com/en-us/answers/questions/464921/azure-app-insights-for-java-springboot-application)
- [Enhancing Search and Insights with Azure AI Search](https://medium.com/@ilakk2023/329-enhancing-search-and-insights-with-azure-ai-search-features-benefits-and-use-cases-2a2954178ffd)

## What is next
Convert this solution into an AI Agent, a system that is using LLM (combined in RAG) and performs actions. aka [Function call API pattern](It enables applications to perform a complex collection of precise tasks and make decisions independently.). It enables applications to perform a complex collection of precise tasks and make decisions independently, 
like automatically raising orders in a legacy OMS order management system. By defining the algorithm in a LLM prompt specifying the function schema. This schema identifies the correct function to performing that specific legacy OMS order entry API endpoint.
- [Ai Agent Patterns with Spring AI](https://dev.to/lucasnscr/ai-agent-patterns-with-spring-ai-43gl)
- [Building a custom AI agent in Java](https://vaadin.com/blog/building-a-custom-ai-agent-in-java-a-hands-on-guide)
- [AI Agents with Spring AI](https://www.youtube.com/watch?v=d2p97gV-kHY)
- 
### Spring AI Tool calling
The goal is to automate tasks that would otherwise require human intervention or explicit programming. For example, a tool can be used to book a flight for a customer interacting with a chatbot, to fill out a form on a web page, or
automatically generate an order in an OMS order management system. (*to be continued*)

- [Spring AI Tool Calling](https://docs.spring.io/spring-ai/reference/api/tools.html) 



