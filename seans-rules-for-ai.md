# Sean's Rules for AI
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
The one-test-per-class pattern ties tests to implementation details, making code harder to change—the opposite of what tests should do.
Tests should enable refactoring, not prevent it.

AI's ability to manage complexity and generate comprehensive fakes lets us test entire behaviors at application boundaries.
We fake external dependencies (files, clock, network) but test through the full internal dependency chain.
Internal structure—whether one class or twenty—is implementation detail that tests should ignore.

The "unit" in unit testing is a behavioral unit, not a compilation unit, so what you are really trying to do is test a coherent piece of behavior in isolation from external dependencies.
Testing the full behavior through a real internal dependency chain with fake external dependencies is not BDD—it is simply correct unit testing.
BDD has similarities to correct unit testing, but BDD typically refers to a more heavyweight methodology involving natural language specifications and special tooling.

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
    val files: Files
    val emitLine: (String) -> Unit
    val exitCode: ExitCode
}

// Entry point orchestrates: wire -> work -> wire -> work
fun runApplication(integrations: Integrations): Int {
    // Stage 1: Bootstrap - WIRING
    val bootstrapDeps = BootstrapDependencies(integrations)
    val configuration = bootstrapDeps.bootstrap.loadConfiguration()  // WORK

    // Stage 2: Schema - WIRING
    val schemaDeps = SchemaDependencies(integrations, configuration)
    val schema = schemaDeps.schemaLoader.loadSchema()  // WORK

    // Stage 3: Application - WIRING
    val appDeps = ApplicationDependencies(integrations, configuration, schema)
    appDeps.reportGenerator.generate()  // WORK

    return integrations.exitCode.value
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
    emitLine = { line -> capturedOutput.add(line) },
    exitCode = ExitCodeImpl()
)
val exitCode = runApplication(testIntegrations)
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
class ApplicationTester {
    private val fakeFileContents = mutableMapOf<String, List<String>>()
    private val capturedOutput = mutableListOf<String>()

    // Setup methods - configure fake behavior
    fun setupConfigFile(fileName: String, csvPath: String, columns: String, format: String) {
        val lines = listOf(
            "csv-path=$csvPath",
            "columns=$columns",
            "format=$format"
        )
        fakeFileContents[fileName] = lines
    }

    fun setupCsvFile(fileName: String, header: List<String>, rows: List<List<String>>) {
        val headerLine = header.joinToString(",")
        val dataLines = rows.map { row -> row.joinToString(",") }
        fakeFileContents[fileName] = listOf(headerLine) + dataLines
    }

    // Action method - returns exit code
    fun runApplication(configFileName: String): Int {
        val fakeFiles = FakeFiles(fakeFileContents)
        val exitCode = ExitCodeImpl()
        val testIntegrations: Integrations = TestIntegrations(
            commandLineArgs = arrayOf(configFileName),
            files = fakeFiles,
            emitLine = { line -> capturedOutput.add(line) },
            exitCode = exitCode
        )
        return com.seanshubin.csv.report.console.runApplication(testIntegrations)
    }

    // Query methods - for assertions
    fun outputContains(text: String): Boolean {
        return capturedOutput.any { it.contains(text) }
    }

    fun getOutputLineCount(): Int = capturedOutput.size
}

// Test using orchestrator
@Test
fun `full application flow with fake integrations`() {
    // given
    val tester = ApplicationTester()
    tester.setupConfigFile("test-config.txt",
        csvPath = "test-data.csv",
        columns = "name,department",
        format = "TABLE"
    )
    tester.setupCsvFile("test-data.csv",
        header = listOf("name", "age", "department", "salary"),
        rows = listOf(
            listOf("Alice", "28", "Engineering", "75000"),
            listOf("Bob", "35", "Marketing", "82000")
        )
    )

    // when
    tester.runApplication("test-config.txt")

    // then
    assertTrue(tester.outputContains("Alice"))
    assertTrue(tester.outputContains("Engineering"))
    assertTrue(!tester.outputContains("salary"))  // Filtered out
    assertEquals(5, tester.getOutputLineCount())
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
