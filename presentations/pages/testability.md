# AI - Test Automation more important than ever
[<img src="../images/back.png">](../presentation)

### The need to continuously Integrate test AI
- While LLMs are powerful, they are prone to generating hallucinations
- Their responses may not always be relevant, appropriate and or factual accurate
- While Generative AI you still have things visible and in control;... AI Agents act autonomously ...potentially harming your systems...

## Testing LLM Responses using Spring AI Evaluators
- Evaluating LLM responses using an LLM itself, preferably a separate one
- **RelevanceEvaluator**
  - Checking LLM response is relevant to the user's query and retrieved context from vector store and/or tools
- **FactCheckingEvaluator**
  - Validate the factual accuracy of the LLM response against the retrieved context.
- Use current Industry standard [minicheck](https://ollama.com/library/bespoke-minicheck) LLM Model to evaluate your LLM.
  - Open-Source model specifically trained for evaluation testing by [Bespoke Labs](https://www.bespokelabs.ai/); Open-Source Data Curator for 
    - Agentic traces
    - Model Fine-tuning
    - RAG evaluation
    - Improving Retrieval (tools)
  - ranks at the top of the [LLM-AggreFact leaderboard](https://llm-aggrefact.github.io/) and only produces a yes/no response.
  - LLM fact-checkers models should be small and cheap to run...
  - LLM-AggreFact is a fact-checking benchmark that aggregates 11 of the most up-to-date publicly available datasets on grounded factuality (i.e., hallucination) evaluation.
- Put both type of tests on DevOps pipelines and let them run on demand or on pull requests....

### References
- [Spring AI Model Evaluation and Testing](https://docs.spring.io/spring-ai/reference/api/testing.html)

[<img src="../images/back.png">](../presentation)
