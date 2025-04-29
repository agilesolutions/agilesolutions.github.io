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
- Put both type of tests on DevOps pipelines and let them run on demand or on pull requests....

[<img src="../images/back.png">](../presentation)
