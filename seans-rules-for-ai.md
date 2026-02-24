# Sean's Rules for AI
- Constraints
  - Actively engage with AI so that you know what it is actually doing
  - Don't trust anything an AI says
  - Don't let the AI make decisions
- Testing
  - Use deep tests instead of shallow tests
  - Use the staged dependency injection and test orchestrator patterns
  - Smoke test a single happy path scenario
- Benefits
  - Research assistant
  - Document organizer
  - Mechanical multiplier

## Details

### Actively engage with AI so that you know what it is actually doing
You need to establish a feedback loop.
You may think that the AI is taking care of a lot for you, but the AI is exceptionally capable of giving you something that very plausibly looks like what you asked for, but is in fact, flat wrong.
The only way to get AI to give you what you wanted, instead of plausibly imitating what you asked for, is to keep challenging the code it generates until the code matches what you actually wanted.
A surprising behavior that I get with AI and not with humans, is that when I ask it to explain its architectural or design decisions, it will without ego tell me that its decisions are terrible.
The AI can understand that code is bad, but does not have the initiative to avoid bad code in the first place.
You have to actively engage to call out the bad code until you can see for yourself that it is fixed.

### Don't trust anything an AI says
The AI is optimized for generating plausible sounding answers, not true answers.
This does not mean that the AI can't get to the right answer eventually.
It means that the AI has no way to tell the difference between correct and incorrect answers without a human challenging it.
Keep pressing the AI to explain itself until you can tell independently of the AI that the final answer is true.

### Don't let the AI make decisions
The AI's decisions will look plausible, but it can only respond probabilistically based on your prompt, it can't know your full context.
As the AI has access to a vast breadth of knowledge, it is good at suggesting things you would not have otherwise thought of.
The AI can be guided to the right decision eventually, but only after the human challenges its reasoning and its beliefs about what is true.
Once you both fully understand and agree with the AI's proposal, it is no longer an AI decision, you have taken ownership of the decision.

### Use deep tests instead of shallow tests
The AI's ability to manage complexity in code, perform large refactorings, not miss details, and generate comprehensive fakes, greatly reduces the need to break up tests around implementation detail boundaries.
This allows us to push our testing all the way up to the application boundaries while still maintaining determinism by faking the foundational integration points.

The staged dependency injection pattern enables deep testing by bundling all external interactions into Integrations.
In production: `ProductionIntegrations(args)` provides real files, clock, network, etc.
In tests: `TestIntegrations(testArgs, fakeFiles, fakeClock, capturedOutput)` provides fakes.

Because you swap a single object at the boundary, the entire application runs with fakes—you test through the full dependency chain without mocking internal collaborators.
This is practical because AI can manage the complexity of comprehensive fakes.
See the staged dependency injection pattern in the "Patterns" section below.

### Use the test orchestrator pattern
At some point, the behavior of AI generated code has to be held to account to human understanding.
We need a feedback loop.
This is where the test orchestrator pattern comes in.
The test orchestrator handles all the test infrastructure details, freeing up the test to focus on human auditable behavior.

The orchestrator has three types of methods:
- **Setup methods**: Configure fake behavior (setupConfigFile, setupCsvFile)
- **Action methods**: Represent user/system actions (runApplication)
- **Query methods**: For assertions (outputContains, getOutputLineCount)

You review the orchestrator's API to verify it matches your mental model.
AI changes everything behind that API—fakes, stubs, implementation—while the test stays the same.
If the test still passes, behavior is preserved.
This frees up the AI to vastly change implementation details, including the corresponding fakes and stubs, while still proving the code behaves to human specification.
See the test orchestrator pattern in the "Patterns" section below.

### Smoke test a single happy path scenario
We can test all logic inside our application with deterministic unit tests.
We can test the behavior of things we depend on outside our application with integration tests.
The only remaining places to hide are how we hook up everything together and differences in our production environment.
Hooking up everything together can be tested with a smoke test.
Differences in environments can be tested by running that smoke test on a staging environment that is identical to production.
The smoke test is not for testing logic, so it only needs a single happy path that exercises all the integration points.

### Research Assistant
I don't have to pore over documentation, follow links, search for articles, etc.
The AI does all of this mechanical and menial labor of figuring out what information is relevant and what is not, then summarizes the information with code examples specifically tailored to the problem I am trying to solve.
I can learn and understand new technology much faster with AI than I could on my own.

### Document organizer
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

You swap a single object at the boundary. The entire application runs with fakes - you test through the full dependency chain without mocking internal collaborators. This is practical because AI can manage the complexity of comprehensive fakes.

### Test Orchestrator Pattern

**Purpose:** Hide infrastructure complexity, expose domain-focused test API.

The orchestrator makes tests readable, maintainable, and resilient to implementation changes. You review the orchestrator's API to verify it matches your mental model. AI changes everything behind that API - fakes, stubs, implementation - while the test stays the same. If the test still passes, behavior is preserved.

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
- You only review the orchestrator's API, not every line of fake implementation
