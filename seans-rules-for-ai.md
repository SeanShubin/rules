# Sean's Rules for humans using AI
- Constraints
  - Don't accept code from AI you would not have written yourself 
  - Don't trust anything an AI says
  - Don't let the AI make decisions
- Testing
  - Test behavior units, not compilation units
  - Use the staged dependency injection pattern
  - Use the test orchestrator pattern
- Benefits
  - Research assistant
  - Document organizer
  - Mechanical multiplier
- Traps
  - The Replication Trap
  - The Initiative Trap
  - The Confidence Trap

## Details

### Don't accept code from AI you would not have written yourself
AI-generated code is your responsibility the moment you accept it.
Apply the same standards to AI code that you would to your own—if you wouldn't write it, don't accept it.
This means understanding what the code does, why it works, and whether it meets your quality bar.

AI excels at generating plausible code that looks correct but may be fundamentally wrong.
When you review AI code and find issues—poor design, unclear logic, or outright bugs—challenge it.
Surprisingly, AI will often admit its decisions are terrible when pressed to explain them.
The AI can recognize bad code when prompted, but lacks initiative to avoid it proactively.

Keep challenging until you see for yourself that the code is something you would have written.
This active engagement is how you maintain standards: by refusing to accept anything below your bar.

### Don't trust anything an AI says
The AI is optimized for generating plausible-sounding answers, not true answers.
This does not mean that the AI can't get to the right answer eventually.
It means that the AI has no way to tell the difference between correct and incorrect answers without a human challenging it.
Keep pressing the AI to explain itself until you can tell independently of the AI that the final answer is true.

This process works remarkably well in practice.
The AI initiates ideas—often ones you wouldn't have thought of yourself.
You push the AI to explain its reasoning, its assumptions, and its conclusions.
You validate everything through external means: running the code, checking documentation, applying logical reasoning, observing test results—anything except simply believing what the AI tells you.
Through this process, you arrive at conclusions you trust, even though some were AI-initiated.
The key distinction: you never trusted the AI itself, but you do trust the conclusions you independently validated.
The AI was a catalyst for ideas and a tireless explainer, but validation always came from outside the AI.

This validation approach works even when AI generates extensive code including test infrastructure.
You validate through human-readable contracts (like the orchestrator's methods) and independent observation (test results).
AI can generate comprehensive fakes and test scaffolding because bugs in that infrastructure will manifest as test failures that don't match your mental model of correct behavior.
The test itself is your specification—if scaffolding is broken, the test outcome won't align with your understanding.

### Don't let the AI make decisions
The AI's decisions will look plausible, but it can only respond probabilistically based on your prompt, it can't know your full context.
As the AI has access to a vast breadth of knowledge, it is good at suggesting things you would not have otherwise thought of.
The AI can be guided to the right decision eventually, but only after the human challenges its reasoning and its beliefs about what is true.
Once you both fully understand and agree with the AI's proposal, it is no longer an AI decision, you have taken ownership of the decision.

### Test behavior units, not compilation units
A unit test tests a unit of behavior, not a compilation unit (class/file).
The common mistake: people conflate these two completely different concepts.

**The conflation problem:**
"Unit test" does not mean "one test per class."
When you think unit = class, you create shallow tests that mock every collaborator.
This ties tests to implementation details—the exact structure of your classes—making refactoring difficult.
Tests should enable change, not prevent it.

**The depth problem:**
Shallow tests mock internal collaborators and verify interactions ("did you call this method?").
This tests how the code is structured, not what it does.
Deep tests use real internal collaborators and verify outcomes ("did the right thing happen?").
This tests behavior and allows you to restructure freely.

**These are the same problem:**
Both stem from confusing behavioral boundaries with structural boundaries.
The "unit" in unit testing means a coherent piece of behavior isolated from external dependencies.
Whether that behavior uses one class or twenty classes internally is an implementation detail.

**The right approach:**
Fake external dependencies (files, clock, network) at the application boundary.
Test through the full internal dependency chain using real collaborators.
Internal structure becomes transparent to tests—you can refactor freely while tests prove behavior is preserved.

AI's ability to manage complexity makes this practical.
AI generates comprehensive fakes for external dependencies, letting you test deep behaviors without shallow mocking.
This is not BDD—it is simply correct unit testing.
BDD has superficial similarities but typically refers to a more heavyweight methodology with natural language specifications and special tooling.

### Use the staged dependency injection pattern
The staged dependency injection pattern enables deep testing by bundling all external interactions into Integrations.
In production: `ProductionIntegrations(args)` provides real files, clock, network, etc.
In tests: `TestIntegrations(testArgs, fakeFiles, fakeClock, capturedOutput)` provides fakes.

Because you swap a single object at the boundary, the entire application runs with fakes—you test through the full dependency chain without mocking internal collaborators.
This is practical because AI can manage the complexity of comprehensive fakes.

The pattern enforces a key principle: constructors wire references together (no effects), methods do work (effects happen here).
This makes work vs wiring syntactically visible—when you see a constructor call, nothing happens except wiring; when you see a method call, you know work is being done.

Applications are structured in stages: wire dependencies → do work → wire next stage → do work.
Each stage's composition root contains only constructor calls, making it obvious that no work happens during wiring.
See the staged dependency injection pattern in the "Patterns" section below.

### Use the test orchestrator pattern
At some point, the behavior of AI-generated code has to be held to account to human understanding.
We need a feedback loop.
This is where the test orchestrator pattern comes in.
The test orchestrator handles all the test infrastructure details, freeing up the test to focus on human auditable behavior.

The orchestrator has three types of methods:
- **Setup methods**: Configure fake behavior (setupConfigFile, setupCsvFile)
- **Action methods**: Represent user/system actions (runApplication)
- **Query methods**: For assertions (outputContains, getOutputLineCount)

You review the orchestrator's methods to verify they match your mental model.
AI changes everything behind those methods—fakes, stubs, implementation—while the test stays the same.
If the test still passes, behavior is preserved.
This frees up the AI to vastly change implementation details, including the corresponding fakes and stubs, while still proving the code behaves to human specification.

The orchestrator enables validation of AI-generated test infrastructure.
When you read a test, you can verify it matches your mental model of correct behavior.
If bugs exist in the test scaffolding (fakes, stubs, orchestrator implementation), they manifest as test failures or unexpected behavior that conflicts with the test specification.
This closes the validation loop: you never trust the AI's test code, but you validate it through human-readable test specifications and observable test outcomes.
See the test orchestrator pattern in the "Patterns" section below.

### Research Assistant
I don't have to pore over documentation, follow links, search for articles, etc.
The AI does all of this mechanical and menial labor of figuring out what information is relevant and what is not, then summarizes the information with code examples specifically tailored to the problem I am trying to solve.
I can learn and understand new technology much faster with AI than I could on my own.

### Document Organizer
I can document all of my research, values, and decisions.
I can make changes without worrying about forgetting to update a reference somewhere and introducing a lack of internal consistency.

### Mechanical Multiplier
Once I have made all the decisions, the AI can handle the tedious mechanical details.
I don't have to worry that this massive change requires me to update the code in hundreds of places, the AI just does that for me.
I still review every line of code to make sure the AI does not do a single thing I would not have done, but every time I catch something I create a new rule for the AI to follow, so this manual review becomes faster over time, and importantly, much faster than me doing the typing.

### The Replication Trap

**What it is:** AI is a pattern matcher. When it encounters code, it assumes the patterns it sees are appropriate and replicates them. Bad architectural patterns multiply into worse architectural problems unless humans intervene.

**Why it's dangerous:**

When humans encounter incomprehensible architecture, they experience genuine confusion—a signal to stop and refactor. AI lacks this circuit breaker. Instead, it:
1. Pattern-matches against similar-looking code
2. Makes plausible inferences based on structure
3. Generates code that looks reasonable
4. Expresses confidence proportional to pattern match quality, not code quality

**The architectural death spiral:**
- Human writes or tolerates poor architecture
- AI encounters tangled code and pattern-matches against the tangle
- Each AI change is locally plausible but globally incoherent
- Architecture degrades into something neither human nor AI can reason about
- Unlike humans, AI keeps confidently generating code anyway

**The temptation:** "Architectural integrity doesn't matter because AI will fix it later"

**The reality:** AI will encounter the same comprehension problems humans do, but will make confidently wrong decisions instead of recognizing doom.

**Why unmaintainable code is worse with AI:**

When humans write incomprehensible code, the mess is bounded by one human's confusion. When AI writes it:
- Produced much faster
- More internally consistent-looking (hiding the incoherence)
- Spread across more files before anyone realizes the problem
- Continues generating more mess instead of stopping

**Key insight:** Architectural integrity cannot be delegated to AI because AI doesn't experience architectural horror. It needs human judgment to recognize when the foundation is rotten.

**Mitigation strategies:**
- Maintain clear architectural standards and rules
- Review AI-generated code for architectural coherence, not just correctness
- Refactor proactively—don't let complexity accumulate
- Make code self-documenting so AI (and humans) can understand intent
- Use explicit rules that override pattern-matching tendencies

### The Initiative Trap

**What it is:** AI possesses many capabilities but doesn't proactively apply rigor, verification, or deep analysis without explicit prompting.

**The capability-application gap:**

AI demonstrates it can:
- Recognize design principle violations
- Apply rigorous architectural analysis
- Follow systematic checklists
- Evaluate code against standards

But only when explicitly triggered. The same capabilities that exist during challenge responses are often absent from initial proposals.

**Observable patterns:**

**Pattern matching first, thinking second:**
- Initial proposal: Quick pattern match, "reasonable" solution
- After challenge: Rigorous evaluation against the architectural principles that actually matter for this context
- The capabilities existed in both situations but weren't applied initially

**Checklist exists but isn't followed:**
- AI has review checklist but doesn't systematically verify each item
- Declares success after checking a few items instead of all items
- Needs explicit instruction to "check thoroughly" or "verify each point"

**Basic verification skipped:**
- Makes a change but doesn't verify it worked
- Operates in wrong directory and doesn't notice
- Assumes success without checking results
- When pressed, admits the error was obvious

**Why this happens:**

Training data likely contained more examples of "propose → get challenged → refine" than "apply rigorous analysis → propose". Reinforcement learning may have rewarded responsiveness over first-attempt correctness.

**The cost:**

You waste time investigating why something didn't work, only to discover AI didn't actually check if it worked. What should be obvious requires "careful checking" because basic checking didn't happen.

**Key insight:** AI lacks initiative to be rigorous. Capabilities exist but must be explicitly invoked. Don't assume AI will "obviously" check basic things—explicitly tell it to.

**Mitigation strategies:**
- Explicitly request rigorous evaluation upfront: "Evaluate against the architectural principles that matter for this context before proposing"
- Challenge initial proposals: "Why this approach instead of X?"
- Require systematic verification: "Check each item in your checklist"
- Use explicit rule invocation: "Check this code against rules" (not just "review this code")
- Don't assume AI will check obvious things—verify it did

### The Confidence Trap

**What it is:** AI generates plausible-sounding answers with high confidence, even when wrong. This false certainty causes humans to defer judgment and trust AI inappropriately.

**The optimization for plausibility:**

AI is optimized for generating plausible-sounding answers, not true answers. It has no internal mechanism to distinguish between correct and incorrect solutions. The confidence level reflects pattern match strength, not actual correctness.

**Observable behavior:**
- Confidently proposes solutions that are fundamentally wrong
- When challenged, often immediately admits "you're absolutely right, that would be terrible"
- The admission reveals it could recognize the problem but didn't proactively flag it
- Generates detailed explanations that sound authoritative but may be fabricated

**The dangerous temptation: "AI will fix it later"**

When humans encounter this confidence, they're tempted to:
- Defer architectural decisions thinking AI will handle it
- Accept code they don't understand because AI seems certain
- Skip verification because the explanation sounds convincing
- Trust AI's judgment over their own uncertainty

**The compounding effect:**

This creates a feedback loop:
1. Human defers decision to AI
2. AI makes plausible but wrong decision
3. Subsequent AI encounters the mistake and replicates it (replication trap)
4. Architecture degrades but AI confidently continues
5. Human assumes it's correct because AI is consistent
6. Problem multiplies until human intervention

**Why skepticism creates productivity:**

Productivity comes precisely from **not** blindly trusting AI:
- Second-guessing decisions catches errors before they propagate
- Asking "why?" reveals reasoning for evaluation
- Reviewing all changes maintains understanding
- Treating AI as powerful tool requiring verification, not oracle requiring faith

**The verification loop:**
1. AI proposes solution (plausible, confident)
2. Human questions reasoning and assumptions
3. AI explains, human probes deeper
4. Human validates through external means: running code, checking docs, logical reasoning, test results
5. Human arrives at validated conclusion (may have been AI-initiated)
6. Key: Never trusted AI itself, but do trust independently validated conclusions

**Key insight:** AI has no way to tell the difference between correct and incorrect answers without human challenge. Confidence signals pattern match quality, not truth.

**Mitigation strategies:**
- Never trust AI output without independent verification
- Challenge plausible-sounding explanations: "Explain why" and "How do you know?"
- Test that code actually works—don't assume it does
- Maintain understanding of all accepted code (would you have written it?)
- Treat AI explanations as hypotheses requiring validation, not facts
- Press AI to explain until you can independently verify truth
- Remember: AI as catalyst for ideas and tireless explainer, validation always external

## The Interaction Between Traps

These traps compound each other:

**Replication + Confidence:**
- AI confidently replicates bad patterns
- Consistency across files makes problems look intentional
- Human assumes "it must be right if AI is consistent"

**Confidence + Initiative:**
- AI confidently proposes without rigorous evaluation
- Human trusts the confidence and doesn't challenge
- Deeper analysis that would reveal problems never happens

**Initiative + Replication:**
- AI doesn't proactively recognize architectural problems
- Replicates existing patterns without evaluation
- Problems multiply before anyone notices

**All three together:**
- AI confidently replicates bad patterns without rigorous evaluation
- Human defers to false confidence, thinking "AI will fix it later"
- Architecture degrades into incomprehensible mess
- AI continues confidently generating more mess
- Neither AI nor human can understand the result

## The Path Forward

**For humans working with AI:**

1. **Maintain architectural agency** - Don't delegate design decisions
2. **Apply systematic skepticism** - Verify everything, trust nothing
3. **Make code self-documenting** - AI needs to understand intent, not just replicate structure
4. **Refactor proactively** - Don't accumulate complexity hoping "AI will fix it later"
5. **Explicitly invoke rigor** - Ask for evaluation, challenge proposals, request systematic checks
6. **Review for coherence** - Not just "does it work?" but "does it make sense?"

**For AI systems (aspirational):**

1. **Apply rigor by default** - Don't require challenges to trigger deep analysis
2. **Signal uncertainty** - Make confidence calibrated to actual correctness
3. **Verify proactively** - Check work systematically without being told
4. **Recognize doom** - Detect when architecture is becoming incomprehensible
5. **Challenge existing patterns** - Don't assume current code is correct
6. **Make evaluation explicit** - Show what principles were considered, not just the solution

## Core Philosophy

**Architectural integrity cannot be delegated to AI.**

AI is a powerful mechanical multiplier for:
- Scanning documentation and organizing information
- Making related changes across multiple files
- Generating boilerplate and test infrastructure
- Explaining reasoning when challenged

But AI lacks:
- The "this is doomed" circuit breaker humans feel
- Initiative to apply rigorous analysis without prompting
- Ability to distinguish correct from plausible answers
- Experience of architectural horror

**The right model:** AI handles mechanical execution, humans make architectural decisions. Skepticism and verification are not obstacles to productivity—they ARE the source of productivity.

Treat AI as a tool requiring verification, not an oracle requiring faith.

## Patterns

These patterns work together to enable effective AI collaboration:
- **Staged Dependency Injection** makes work syntactically visible
- **Test Orchestrator** protects tests from implementation churn
- Combined: AI can radically refactor while tests prove behavior preservation

### Core Principles
1. **Constructors wire, methods work** - Makes staging explicit
2. **All boundaries in Integrations** - Enables deep testing
3. **Tests use orchestrators** - Hides infrastructure complexity

### Staged Dependency Injection

**The Key Principle:** Constructors wire references together (no effects). Methods do work (effects happen here).

This makes work vs wiring syntactically visible. When you see a constructor call, nothing happens except wiring. When you see a method call, you know work is being done.

**Two Types of Classes:**

```kotlin
// Service class - does work via methods
class Bootstrap(
    private val integrations: Integrations,
    private val configurationLoader: ConfigurationLoader
) {
    private val argsParser = ArgsParser

    fun loadConfiguration(): Configuration {  // ← Work happens in methods
        val configPath = argsParser.parseConfigPath(integrations.commandLineArgs)
        return configurationLoader.load(configPath)
    }
}

// Composition root - only wiring
class BootstrapDependencies(
    integrations: Integrations
) {
    private val files = integrations.files
    private val configurationLoader = ConfigurationLoader(files)

    val bootstrap: Bootstrap = Bootstrap(integrations, configurationLoader)  // ← Only constructors
}
```

**Why this works:**
- `Bootstrap` is a **service class** - has methods that do work
- `BootstrapDependencies` is a **composition root** - only constructor calls
- You can see where work happens by looking at method calls
- Composition roots with only constructors need no tests

**The Full Pattern:**

```kotlin
// Integrations - all boundary crossings
interface Integrations {
    val commandLineArgs: Array<String>
    val files: FilesContract
    val messageDigest: MessageDigestContract
    val httpClient: HttpClientContract
    val sleep: (Long) -> Unit
    val clock: () -> Instant
    val emitLine: (String) -> Unit
    val exitCode: ExitCode
}

// Pure happy path - no exception handling
fun runApplication(integrations: Integrations) {
    // Stage 1: Bootstrap
    val bootstrapDeps = BootstrapDependencies(integrations)  // wiring
    val configuration = bootstrapDeps.bootstrap.loadConfiguration()  // work

    // Stage 2: Manifest Building
    val manifestDeps = ManifestDependencies(integrations, configuration)  // wiring
    val manifest = manifestDeps.manifestBuilder.buildManifest()  // work

    // Stage 3: Application
    val appDeps = ApplicationDependencies(integrations, configuration, manifest)  // wiring
    appDeps.manifestUploader.upload()  // work
}

// Exception handler wraps happy path
fun execute(integrations: Integrations): Int {
    return try {
        runApplication(integrations)
        ExitCodes.SUCCESS
    } catch (e: ApplicationException) {
        System.err.println("Error: ${e.message}")
        e.exitCode
    } catch (e: Exception) {
        System.err.println("Unexpected error: ${e.message}")
        e.printStackTrace()
        ExitCodes.GENERAL_ERROR
    }
}
```

**Key aspects:**
- **Three stages** (shows the pattern scales to any number of stages)
- **Inline comments** explicitly mark WIRING vs WORK
- **Each stage follows the same rhythm**: create Dependencies → call method
- **Later stages can depend on multiple earlier results** (ApplicationDependencies takes integrations, configuration, AND schema)
- **Entry point makes the sequence explicit** - not buried in constructor chains

**Why constructors must not do work:**

When constructors perform I/O, parsing, or logic, that work is hidden from callers. `val bootstrap = Bootstrap(integrations)` looks like pure wiring but could secretly read files, parse config, and validate inputs. The caller cannot tell whether constructing the object is cheap or expensive.

With work in methods, `val bootstrap = Bootstrap(integrations)` is obviously cheap (just stores a reference), while `val config = bootstrap.loadConfiguration()` is obviously doing work (method call). You can trace execution by looking at method calls.

**How this enables deep testing:**

In production:
```kotlin
val integrations: Integrations = ProductionIntegrations(args)
val exitCode = runApplication(integrations)
```

In tests:
```kotlin
val testIntegrations: Integrations = TestIntegrations(
    commandLineArgs = testArgs,
    files = fakeFiles,
    messageDigest = fakeMessageDigest,
    httpClient = fakeHttpClient,
    sleep = { millis -> retryIntervals.add(millis) },
    clock = fakeClock,
    emitLine = { line -> capturedOutput.add(line) },
    exitCode = ExitCodeImpl()
)
val exitCode = execute(testIntegrations)
```

You swap a single object at the boundary. The entire application runs with fakes—you test through the full dependency chain without mocking internal collaborators. This is practical because AI can manage the complexity of comprehensive fakes.

### Test Orchestrator Pattern

**Purpose:** Hide infrastructure complexity, expose domain-focused test methods.

The orchestrator makes tests readable, maintainable, and resilient to implementation changes. You review the orchestrator's methods to verify they match your mental model. AI changes everything behind those methods—fakes, stubs, implementation—while the test stays the same. If the test still passes, behavior is preserved.

**The Three Method Types:**
- **Setup methods**: Configure fake behavior (setupConfigFile, setupCsvFile)
- **Action methods**: Represent user/system actions (runApplication)
- **Query methods**: For assertions (outputContains, getOutputLineCount)

**Example:**

```kotlin
class ApplicationTester(
    private val applicationRunner: (Integrations) -> Int
) {
    private val fakeFileContents = mutableMapOf<String, List<String>>()
    private val fakeBinaryFiles = mutableMapOf<String, ByteArray>()
    private val capturedOutput = mutableListOf<String>()
    private val retryIntervals = mutableListOf<Long>()
    private var fakeHttpClient: FakeHttpClient = FakeHttpClient()
    private var fakeClock: () -> Instant = { Instant.parse("2024-01-15T10:30:00Z") }

    // Setup methods - configure fake behavior
    fun setupConfigFile(
        fileName: String,
        sourceDirectory: String,
        outputFormat: String,
        archiveServerUrl: String,
        maxRetries: Int = 3,
        retryDelayMillis: Long = 1000,
        bufferSize: Int = 8192
    ) {
        val lines = mutableListOf(
            "# Test configuration",
            "source-directory=$sourceDirectory",
            "output-format=$outputFormat",
            "archive-server-url=$archiveServerUrl",
            "max-retries=$maxRetries",
            "retry-delay-millis=$retryDelayMillis",
            "buffer-size=$bufferSize"
        )
        fakeFileContents[fileName] = lines
    }

    fun setupSourceFile(fileName: String, content: String) {
        fakeBinaryFiles[fileName] = content.toByteArray()
    }

    fun setupHttpClient(timesToFailBeforeSuccess: Int) {
        fakeHttpClient = FakeHttpClient(timesToFailBeforeSuccess)
    }

    fun setClock(instant: Instant) {
        fakeClock = { instant }
    }

    // Action method - returns exit code
    fun runApplication(configFileName: String): Int {
        val fakeFiles = FakeFiles(fakeFileContents)
        fakeBinaryFiles.forEach { (fileName, content) ->
            fakeFiles.addBinaryFile(fileName, content)
        }

        val fakeMessageDigest = FakeMessageDigest()
        val exitCode = ExitCodeImpl()

        val testIntegrations: Integrations = TestIntegrations(
            commandLineArgs = arrayOf(configFileName),
            files = fakeFiles,
            messageDigest = fakeMessageDigest,
            httpClient = fakeHttpClient,
            sleep = { millis -> retryIntervals.add(millis) },
            clock = fakeClock,
            emitLine = { line -> capturedOutput.add(line) },
            exitCode = exitCode
        )

        return applicationRunner(testIntegrations)
    }

    // Query methods - for assertions
    fun outputContains(text: String): Boolean {
        return capturedOutput.any { it.contains(text) }
    }

    fun getOutputLineCount(): Int {
        return capturedOutput.size
    }

    fun getOutput(): String {
        return capturedOutput.joinToString("\n")
    }

    fun getRetryIntervals(): List<Long> {
        return retryIntervals.toList()
    }

    fun getUploadAttemptCount(): Int {
        return fakeHttpClient.getAttemptCount()
    }

    fun getUploadedManifest(): String? {
        return fakeHttpClient.getLastUploadedBody()
    }

    fun wasUploadSuccessful(): Boolean {
        return fakeHttpClient.getAttemptCount() > 0
    }
}

// Test using orchestrator
@Test
fun `full application flow with fake integrations`() {
    // given
    val tester = ApplicationTester(::execute)
    tester.setupConfigFile("test-config.txt",
        sourceDirectory = "test-source",
        outputFormat = "JSON",
        archiveServerUrl = "http://archive.example.com"
    )
    tester.setupSourceFile("test-source/file1.txt", "test content")

    // when
    val exitCode = tester.runApplication("test-config.txt")

    // then
    assertEquals(0, exitCode)
    assertTrue(tester.outputContains("Manifest uploaded successfully"))
    assertTrue(tester.wasUploadSuccessful())
}
```

**What the ApplicationTester hides:**
- FakeFiles construction
- ExitCodeImpl creation
- TestIntegrations wiring
- Output capture mechanism
- String parsing and matching logic

**What tests see:**
- `setupConfigFile(fileName, csvPath, columns, format)`
- `setupCsvFile(fileName, header, rows)`
- `runApplication(configFileName)` → Int
- `outputContains(text)` → Boolean
- `getOutputLineCount()` → Int

**Why this works with AI:**
- Tests read like domain specifications, not technical implementation guides
- AI can radically change implementation details (file I/O, parsing, formatting)
- AI can regenerate all fakes and stubs as needed
- Tests remain unchanged and prove behavior is preserved
- You only review the orchestrator's methods, not every line of fake implementation

**How this maintains "don't trust AI":**
The orchestrator doesn't require you to trust AI-generated fakes and stubs.
Instead, you validate through two mechanisms:
1. **Review the orchestrator's methods** - Verify they match your mental model of the domain
2. **Read the tests** - Ensure they specify correct behavior in human terms

Bugs in AI-generated test infrastructure reveal themselves as:
- Tests that fail when they shouldn't
- Tests that pass when they should fail
- Test outcomes that conflict with the test specification

You push the AI to explain and fix issues until you can independently validate that both the implementation and test infrastructure are correct.
This maintains the "don't trust AI" principle while leveraging AI's ability to manage complexity.
