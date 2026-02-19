# Sean's Rules for AI
- Constraints
  - Actively engage with AI so that you know what it is actually doing
  - Don't trust anything an AI says
  - Don't let the AI make decisions
- Changes
  - Use deep tests instead of shallow tests
  - Use the test orchestrator pattern
  - Smoke test a single happy path scenario
- Benefits
  - Research assistant
  - Document organizer
  - Mechanical multiplier

## Details

### Actively engage with AI so that you know what it is actually doing
You need to establish a feedback loop.
You may think that the AI is taking care of a lot for you, but the AI is exceptionally capable of giving you something that very plausibly looks like what you asked for, but is in fact, flat wrong.
The only way to get AI to give you what you wanted, instead of plausibly imitating what you asked for, to keep challenging the code it generates until the code matches what you actually wanted.
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
As the AI has access to a vast breadth of knowledge, it is good at suggesting you would not have otherwise thought of.
The AI can be guided to the right decision eventually, but only after the human challenges its reasoning and its beliefs about what is true.
Once you both fully understand and agree with the AI's proposal, it is no longer an AI decision, you have taken ownership of the decision.

### Use deep tests instead of shallow tests
The AI's ability to manage complexity in code, perform large refactorings, not miss details, and generate comprehensive fakes, greatly reduces the need to break up tests around implementation detail boundaries.
This allows us to push our testing all the way up to the application boundaries while still maintaining determinism by faking the foundational integration points.
Use the staged dependency injection pattern and create fakes for all your integrations, for example, in kotlin, this might look like:
```kotlin
// Service class - does work via methods
class Bootstrap(
    private val integrations: Integrations  // Constructor-injected dependencies
) {
    private val argsParser = ArgsParser

    fun loadConfiguration(): Configuration {  // Work happens in methods
        val configBaseName = argsParser.parseConfigBaseName(integrations.commandLineArgs)
        val loader = ConfigurationLoader(integrations, configBaseName)
        return loader.load()
    }

    fun loadConfiguration(configBaseName: String): Configuration {  // Overload for external config source
        val loader = ConfigurationLoader(integrations, configBaseName)
        return loader.load()
    }
}

// Composition root - only wiring
class BootstrapDependencies(
    integrations: Integrations
) {
    val bootstrap: Bootstrap = Bootstrap(integrations)  // Only constructor calls
}

// Composition root - only wiring
class ApplicationDependencies(
    integrations: Integrations,
    configuration: Configuration
) {
    private val files = integrations.files
    private val clock = integrations.clock
    // ... more wiring ...
    val runner: Runnable = Runner(clock, /* ... */)
    val errorMessageHolder: ErrorMessageHolder = ErrorMessageHolderImpl()
}

// Entry point orchestrates: wire -> work -> wire -> work
fun execute(args: Array<String>): Int {
    val integrations = ProductionIntegrations(args)           // Stage 1: WIRING

    val bootstrapDeps = BootstrapDependencies(integrations)   // Stage 2: WIRING
    val configuration = bootstrapDeps.bootstrap.loadConfiguration()  // Stage 2: WORK ←

    val appDeps = ApplicationDependencies(integrations, configuration)  // Stage 3: WIRING
    appDeps.runner.run()  // Stage 3: WORK ←

    return if (appDeps.errorMessageHolder.errorMessage == null) 0 else 1
}
```

### Use the test orchestrator pattern
At some point, the behavior of AI generated code has to be held to account to human understanding.
We need a feedback loop.
This is where the test orchestrator pattern comes in.
The test orchestrator handles all the test infrastructure details, freeing up the test to focus on human auditable behavior.
This frees up the AI to vastly change implementation details, including the corresponding fakes and stubs, while still proving the code behaves to human specification.
Here is an example of what the test orchestrator pattern looks like:
```
TODO: Fix up examples in other rules projects, then add them here
```

### Smoke test a single happy path scenario
We can test all logic inside our application with deterministic unit tests
We can test the behavior of things we depend on outside our application with integration tests
The only remaining places to hide are how we hook up everything together and differences in our production environment.
Hooking up everything together can be tested with a smoke test.
Differences in environments can be tested by running that smoke test on a staging enviromnent that is identical to production.
The smoke test is not for testing logic, so it only needs a single happy path that exercises all the integration points.


### Research Assistant
I don't have to pour over documentation, follow links, search for articles, etc.
The AI does all of this mechanical and menial labor of figuring out what information is relevant and what is not, then summarizes the information with code examples specifically tailored to the problem I am trying to solve.
I can learn and understand new technology much faster with AI than I could on my own.

### Document organizer
I can document all of my research, values, and decisions.
I can make changes without worrying about forgetting to update a reference somewhere and introduce lack of internal consistency.

### Mechanical Multiplier
Once I have made all the decisions, the AI can handle the tedious mechanical details.
I don't have to worry that this massive change requires me to update the code in hundreds of places, the AI just does that for me.
I still review every line of code to make sure the AI does not do a single thing I would not have done, but every time I catch something I create a new rule for the AI to follow, so this manual review becomes faster over time, and importantly, much faster than me doing the typing.
