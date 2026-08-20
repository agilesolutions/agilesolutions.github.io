# Human-in-the-Loop (HITL) Workflows in Embabel

This guide covers how to implement **Human-in-the-Loop (HITL)** interaction patterns in the Embabel framework using the `WaitFor.formSubmission()` mechanism.

Embabel's Goal-Oriented Action Planning (GOAP) engine allows AI agents to act autonomously. However, enterprise workflows frequently require human intervention for **ambiguity resolution, manual approvals, or quality guardrails**. `WaitFor.formSubmission()` lets you seamlessly pause an agentic thread, collect structured human input, and resume execution deterministically.

---

## ⚙️ How it Works Under the Hood

When an agent invokes `WaitFor.formSubmission()`, Embabel handles the complex distributed systems orchestration required for long-running workflows:

1. **State Serialization:** The framework serializes the agent's context block (the "blackboard") and persists it to your tracking store.
2. **Non-blocking Halt:** The execution engine halts the current evaluation and yields control by bubbling up a `ProcessWaitingException`. This frees up server threads.
3. **Schema Generation:** Embabel automatically inspects the requested target `record` or `class` reflection data and exposes a structured JSON schema representing the expected human input form.
4. **Rehydration & Resumption:** Once a human completes the form submission via your API layer, the engine rehydrates the agent's exact state, binds the payload into a strongly-typed Java/Kotlin object, and resumes execution.

---

## 🛠️ Code Implementation Example

### 1. Define the Human Input Record
Create a standard Java or Kotlin record representing the specific data schema you want the human to fill out.

```java
// Java 17+ Record
public record HumanReview(
    @Description("Whether the generated response is safe and accurate to send to the client.")
    boolean approved,
    
    @Description("Detailed feedback or corrections if rejected.")
    String correctionNotes
) {}
```

### 2. Implement the Action with `WaitFor`
Decorate your agent action method. Use `WaitFor.formSubmission()` to safely pause execution if a condition (like a budget limit or low confidence score) is met.

```java
import com.embabel.agent.annotation.Action;
import com.embabel.agent.workflow.WaitFor;

public class ComplianceAgent {

    @Action
    public ReviewResult verifyLargeTransaction(Transaction tx) {
        // Autonomous guardrail check
        if (tx.amount() < 10000) {
            return new ReviewResult(true, "Auto-approved by policy.");
        }

        // Trigger Human-in-the-Loop workflow for high-value transfers
        HumanReview review = WaitFor.formSubmission(
            "Transaction exceeds $10,000 threshold. Manual compliance sign-off required.",
            HumanReview.class
        );

        if (!review.approved()) {
            return new ReviewResult(false, "Rejected by compliance: " + review.correctionNotes());
        }

        return new ReviewResult(true, "Manually approved by human operator.");
    }
}
```

### 3. Expose the API to Resume Execution
To bridge your UI front-end with Embabel, capture the form submission in your REST controller layer and submit it to the process manager.

```java
@RestController
@RequestMapping("/api/workflows")
public class WorkflowController {

    @Autowired
    private EmbabelProcessManager processManager;

    @PostMapping("/{processId}/submit-form")
    public ResponseEntity<Void> submitHumanForm(
        @PathVariable String processId,
        @RequestBody Map<String, Object> formPayload
    ) {
        // Submits the payload back to the awaiting process block
        processManager.resumeProcess(processId, formPayload);
        return ResponseEntity.accepted().build();
    }
}
```

---

## ⚠️ Important Best Practices

* **Mutate State to Avoid Infinite Loops:** Embabel continuously evaluates action preconditions during its planning cycles. Ensure that when execution resumes, your workflow code changes the underlying domain state (e.g., marks the transaction as `REVIEWED`). If the state remains completely unchanged, the planning engine may re-trigger the action and get stuck in an endless loop of calling `WaitFor.formSubmission()`.
* **Use `@Description` Annotations:** Always use descriptions on your target records. Embabel utilizes this metadata to construct rich, self-documenting UI schemas and provides the LLM with clear instructions on what properties it should expect back from the human.
* **Configure Distributed Persistence:** In standard production environments, avoid relying on the default in-memory state tracker. Configure a persistent tracking store (e.g., Redis or JDBC backend) to ensure that paused agent states survive application restarts.