# About my background and expertise in Software Engineering and System Integration
With 40 years experience in Software Engineering and System Integration across diverse industries—including Logistics, Telephony, Banking, Insurance. I have built a career as a results-driven consultant. My expertise spans roles with organizations such as ITT, DEC, Philips, Shell, Samsung, KPN Royal Dutch Telecom, Friesland Bank, UBS, Credit Suisse, Bank Julius Baer, and Swiss RE.
Specialized in optimizing software development processes to enhance productivity and quality across all facets of development.
With focus on Spring and JEE development, build and release management, testing, issue tracking, continuous integration and delivery (CI/CD), and improving code quality.
Accelerating productivity and code quality by integrating AI-powered tools like GitHub Copilot, [GitHub Spec Kit](https://developer.microsoft.com/blog/spec-driven-development-spec-kit) Spec-Driven development into the software development lifecycle. This includes code generation, code review, testing, and documentation.
See more about my backgrounds on [Linkedin account](https://www.linkedin.com/in/robert-rong-agile-solutions/)
## ShowCasing Agentic AI with Spring AI Embabel running on Azure AKS and AI Foundry
Currently, my  main focus is on implementing AI Agentic behavior on existing businesses implemented with Spring Framework and Boot utilizing the new [Embabel Agent Framework](https://github.com/embabel/embabel-agent), running on Azure Kubernetes and connecting to Azure AI Foundry. Everything related to AI development that I have studied so far is worked into [one Github showcase project](https://github.com/agilesolutions/spring-ai-embabel).  

### Why Embabel?
Embabel is an Agentic AI framework for the JVM. Python has long been the go-to for machine learning experiments, thanks to its ease, ecosystem richness, and data scientist-friendly tools.
However, it often struggles when you try to move from experiment to real-world, production-scale AI.

As AI adoption becomes more mission critical, what matters is not just model training or computation.
It is context, reliability, type safety, performance, observability, and integration with existing enterprise systems.
Java and Kotlin offer these strengths. Strong typing, mature tooling, and proven track records in resilient, scalable systems make them a compelling choice for serious AI. It is time to move beyond proofs of concept.

**The Team Behind Embabel:**

Alongside Spring Framework founder Rod Johnson and other alumni, Embabel is built by a team of high-achieving engineers with a proven record not only in applied AI, but also in designing, scaling, and delivering large, complex systems.

### Embabel Core Philosophy and features
Embabel distinguishes itself from Python-based frameworks by emphasizing deterministic orchestration and deep integration with existing enterprise systems.

- **GOAP Planning:** Uses Goal-Oriented Action Planning (GOAP)—a pathfinding algorithm common in gaming—to dynamically determine the best sequence of actions to reach a goal.
- **OODA Loop:** Agents operate in a continuous loop of Observe, Orient, Decide, and Act, reassessing their plan after every step.
- **Type Safety:** Heavily leverages Kotlin and Java's strong typing to ensure that LLM inputs and outputs are grounded in typed domain objects rather than raw text.
- **Spring Integration:** Built on top of Spring AI, it allows developers to use familiar dependency injection and configuration patterns.§
- **Model Context Protocol (MCP):** Supports the Model Context Protocol for standardizing how agents interact with external tools and data sources.

## Full End-to-End, from Development, CI/CD, Deployment to Provisioning
This project is a full end-to-end demonstration of how to develop an AI application with Spring AI Embabel, and then use Azure DevOps pipelines to CI/CD deliver that logic on Azure ACR container registry and finally use Terraform to provision Azure AKS cluster, Azure Flexible server and Azure AI Foundry to provision everything needed to run this process.
- **SpringBoot** application Build process managed with **Gradle**.
- **Azure DevOps pipelines** to build, run Spring AI JUnit Relevancy Factuality AI tests, HELM package and deploy application on Azure AKS cluster.
- **HELM** charts to package and deploy Spring AI application
- **Terraform** manifests:
  - Build AKS Cluster.
  - Provision AD Extra ID managed identities and RBAC rules.
  - Provision MySQL Flexible Server and Database to store stock portfolio records.
  - Provision Azure AI Foundry project, Storage Accounts and Containers and AZ key vaults and policies.

## About the AI Trader Use Case
This Use Case is about an AI Agent that is autonomously doing the following:
- **Fetches** [real-time Stock Price Trends](https://twelvedata.com/) and [real-time Crypto Trends](https://finnhub.io/)
- **Follows** the latest [real-time Financial Economic worldwide developments](https://www.marketaux.com/)
- **Interprets** these real-time developments Stock Market Price forecasts. 
 
**In a few words:** 
The AI Stock Market Trader continuously monitors the stock market, analyzing news, economic data, company performance, and other factors that can influence stock prices, and then makes informed decisions to buy and sell stocks in the stock market, typically to profit from price fluctuations.<br><br>
**[See the presentation](ppt/ai-trader.pdf)**
## What is Agentic AI and how does it work...
Nowadays, we are in the era of Generative AI, where the focus is on generating content and engaging in conversations with humans. However, the next frontier of AI is Agentic AI, which involves creating agents that can plan and act autonomously without user intervention.
Agentic AI is designed to automate repetitive tasks, freeing up human resources to focus on more strategic and creative tasks, such as innovation. By delegating routine tasks to AI agents, organizations can improve overall performance and efficiency. For example, AI agents can be responsible for data entry, data analysis, inventory management, and other tasks that would otherwise require human intervention or explicit programming. This is a significant shift from the traditional AI we are familiar with, which primarily focuses on content generation and back-and-forth conversations with humans.
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
2. [AZ commands](presentations/az.md)
3. [AZ VM's and Networking](presentations/az-networking.md)