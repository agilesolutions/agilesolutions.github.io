# About my experience so far
With 40 years experience in Software Engineering and System Integration across diverse industries—including Logistics, Telephony, Banking, Insurance. I have built a career as a results-driven consultant. My expertise spans roles with organizations such as ITT, DEC, Philips, Shell, Samsung, KPN Royal Dutch Telecom, Friesland Bank, UBS, Credit Suisse, Bank Julius Baer, and Swiss RE.
Specialized in optimizing software development processes to enhance productivity and quality across all facets of development.
With focus on Spring and JEE development, build and release management, testing, issue tracking, continuous integration and delivery (CI/CD), and improving code quality.
See more about my backgrounds on [Linkedin account](https://www.linkedin.com/in/robert-rong-agile-solutions/)
## ShowCasing Agentic AI with Spring AI, running on Azure AKS and AI Foundry
At the moment my main focus goes out to experimenting with AI Agent enabling existing business running with SpringBoot on Azure kubernetes and connecting to Azure AI Foundry. Everything related to AI development I studied so far is worked into [one Github showcase project](https://github.com/agilesolutions/poc-mcp-client/).
## What is Agentic AI and how does it work...
Next frontier of AI world is Agentic AI with Agents that plan and act autonomously without user intervention. Most common scenario is automating repetitive tasks to freeing up human resources to focus on more strategic and creative tasks like innovation. 
This will at the end improve the overall organization performance. You can think about letting Agents being responsible for data entry, data analysis and inventory management and so on.
This is entirely different from the generic AI we are used to, which is primarily focused on content generation and back and forth conversations with humans.

### Agentic AI goes through four-step approach for doing its things:
1. **Preceive** : sensing for additional data on top of what it was learned on from various sources, such as proprietary database, digital interfaces, REST API and so on.
2. **Reason** : interacting with LLM as the reasoning engine to understand and come to a conclusion. Taking in the prompt it was provided (constrained to the relevant business logic), alternatively using techniques like RAG (retrieval-augmented generation) to enriching with proprietary data sources and deliver accurate relevant outputs. 
3. **Act** : executing tasks by calling external tools (that is what my ShowCase project is about), like opening new DB orders on an OMS system (Order Management System). Guardrails can get provided to constrain the amount of impact on your systems.
4. **Learn**: Continuously improving through feedback loops (data flywheel) by feeding back data into its system to enhance its model, ability to adapt and improve it effectiveness over time.

<img title="The data Flywheel of adaptivity" alt="Alt text" src="/images/agentic.png">


## Objectives line out
[Spring AI](https://docs.spring.io/spring-ai/reference/index.html) provides abstractions that serve as the foundation for developing AI applications. I am particularly focusing on the following areas:
1. [Chat Client API](https://docs.spring.io/spring-ai/reference/api/chatclient.html) offering a fluent API for communicating with an AI Model. It supports both a synchronous and streaming programming model.
2. [Azure OpenAI Embeddings](https://docs.spring.io/spring-ai/reference/api/embeddings/azure-openai-embeddings.html) extends the OpenAI capabilities, offering safe text generation and Embeddings computation models for various task
3. [Azure AI Service](https://docs.spring.io/spring-ai/reference/api/vectordbs/azure.html) setting up the AzureVectorStore to store document embeddings and perform similarity searches using the [Azure AI Search](https://azure.microsoft.com/en-us/products/ai-services/ai-search/) Service.
4. [Tool Calling](https://docs.spring.io/spring-ai/reference/api/tools.html) common pattern in AI applications allowing a model to interact with a set of APIs, or tools, augmenting its capabilities as a basis to build SpringBoot oriented AI agents.
5. [Model Context Protocol (MCP)](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-overview.html) enabling standardized protocol that enables AI models to interact with external tools and resources in a structured way. It supports multiple transport mechanisms to provide flexibility across different environments.
6. [Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-ai-foundry) providing me an enterprise-grade platform to building and running generative AI applications on an enterprise-grade platform.
7. [Application Insight and Telemetry](https://learn.microsoft.com/en-us/previous-versions/azure/search/search-traffic-analytics?tabs=visual-studio-telemetry-client%2Cdotnet-correlation%2Cdotnet-properties%2Cdotnet-custom-events) for [Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search) traffic analytics.
8. [Hashicorp Terraform](https://learn.microsoft.com/en-us/azure/ai-services/create-account-terraform?tabs=azure-cli) to deploy Azure AI services resource using Terraform
9. [Hashicorp Terraform](https://learn.microsoft.com/en-us/azure/search/search-get-started-terraform) to deploy Azure AI Search service using Terraform

## Technology stack
- SpringBoot application microservice exposing REST API end-points to triggering demonstrated logic.
- Spring AI
  - Connecting Azure Search Vector database to demonstrating RAG
  - Tool Calling, demonstrating Agentic AI. The goal is to automate that would otherwise require human intervention or explicit programming. For example, a tool can be used to book a flight for a customer interacting with a chatbot, to fill out a form on a web page, or automatically generate an order in an OMS order management system.
- Terraform configuration files to provisioning all Azure infrastructure components needed to run this demo in its full context. That includes a full Azure Kubernetes AKS cluster and Azure AI Foundry HUB, project and finally deploying gpt-4 LLM model.
- Gradle scripts build to compile, test and package to a deployable archive.
- Docker and HELM chart artifacts to providing Kubernetes manifests and deploy this solution onto Azure AKS.
- CI/CD ADO Azure DevOps Pipeline - Fully automated build package and deployment with Azure DevOps.
- Scalable & Cloud-Native - Deployable on Azure App Service or Kubernetes.
- Centralized Logging (Azure Monitor, Application Insights)
- Monitoring & Alerts (Azure Monitor, Prometheus/Grafana)
- Security Best Practices (Key Vault for secrets, RBAC, Network Policies)
- 

# Table of contents
1. [RAG](pages/rag.md)
2. [AI Agentic Tool Calling](pages/tools.md)
3. [Model Context Protocol (MCP) to implementing Agentic AI services](pages/mcp.md)

# Presentations
1. [From Generative to vertical Agentic AI](presentations/presentation.md)
2. [Strategy Seminar for Carreer Management](presentations/seminar.md)
3. [Letter of Application](presentations/letterapplication.md)
4. [Developer skills](presentations/skills.md)
5. [P.I.E. Method Performance, Image, and Exposure](presentations/pie.md)
6. [The STAR method to preparing Interviews and structure your CV](presentations/star.md)
