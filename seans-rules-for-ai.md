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
Use the staged dependency injection pattern and create fakes for all your integrations.
See the example of the staged dependency injection pattern in the "Patterns" section below.

### Use the test orchestrator pattern
At some point, the behavior of AI generated code has to be held to account to human understanding.
We need a feedback loop.
This is where the test orchestrator pattern comes in.
The test orchestrator handles all the test infrastructure details, freeing up the test to focus on human auditable behavior.
This frees up the AI to vastly change implementation details, including the corresponding fakes and stubs, while still proving the code behaves to human specification.
Here is an example of what the test orchestrator pattern looks like:
See the example of the test orchestrator pattern in the "Patterns" section below.

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

### Staged Dependency Injection
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

### Test Orchestrator
```java
public class MessageDigestUtilityInvertedTest {
    @Test
    public void testMessageDigestForDirectory() {
        // given
        String pathName = "the-path";
        int bufferSize = 3;
        Tester tester = new Tester(bufferSize);
        tester.addFile("the-path/file-a.txt", "abcdefg");
        tester.addFile("the-path/file-b.txt", "hij");
        tester.addFile("the-path/file-c.txt", "klmn");
        String expected = "digest for: abc, def, g, hij, klm, n";
        String expectedEvents =
                "Processing file 'the-path/file-a.txt' of size 7 bytes\n" +
                        "Processing file 'the-path/file-b.txt' of size 3 bytes\n" +
                        "Processing file 'the-path/file-c.txt' of size 4 bytes";

        // when
        String actual = tester.messageDigestForDirectory(pathName);

        // then
        assertEquals(expected, actual);
        String actualEvents = tester.getEvents();
        assertEquals(expectedEvents, actualEvents);
    }

    static class Tester {
        final int bufferSize;
        final MessageDigestStub messageDigest;
        final FilesStub files;
        final ProcessingFileEventStub processingFileEvent;
        final MessageDigestUtilityInverted messageDigestUtility;

        public Tester(int bufferSize) {
            this.bufferSize = bufferSize;
            this.messageDigest = new MessageDigestStub();
            this.files = new FilesStub();
            this.processingFileEvent = new ProcessingFileEventStub();
            this.messageDigestUtility = new MessageDigestUtilityInverted(
                    messageDigest,
                    files,
                    bufferSize,
                    processingFileEvent
            );
        }

        void addFile(String fileName, String content) {
            files.addFile(fileName, content);
        }

        String messageDigestForDirectory(String pathName) {
            Path path = Paths.get(pathName);
            byte[] messageDigestBytes = messageDigestUtility.messageDigestForDirectory(path);
            String messageDigestString = new String(messageDigestBytes);
            return messageDigestString;
        }

        String getEvents() {
            return String.join("\n", processingFileEvent.events);
        }
    }

    static class MessageDigestStub extends MessageDigestUnsupportedOperation {
        final List<String> updateCalls = new ArrayList<>();

        @Override
        public byte[] digest() {
            String digestString = "digest for: " + String.join(", ", updateCalls);
            return digestString.getBytes();
        }

        @Override
        public void update(byte[] input, int startOffset, int len) {
            int endOffset = startOffset + len;
            byte[] relevant = Arrays.copyOfRange(input, startOffset, endOffset);
            String relevantAsString = new String(relevant);
            updateCalls.add(relevantAsString);
        }
    }

    static class FilesStub extends FilesUnsupportedOperation {
        List<String> fileNames = new ArrayList<>();
        Map<String, String> fileContents = new HashMap<>();

        void addFile(String fileName, String content) {
            fileNames.add(fileName);
            fileContents.put(fileName, content);
        }

        @Override
        public Path walkFileTree(Path start, FileVisitor<? super Path> visitor) {
            for (String fileName : fileNames) {
                Path filePath = Paths.get(fileName);
                try {
                    visitor.visitFile(filePath, null);
                } catch (IOException e) {
                    throw new RuntimeException(e.getMessage(), e);
                }
            }
            return start;
        }

        @Override
        public boolean isRegularFile(Path path, LinkOption... options) {
            String pathString = path.toString();
            boolean result = fileContents.containsKey(pathString);
            return result;
        }

        @Override
        public long size(Path path) {
            return fileContents.get(path.toString()).getBytes().length;
        }

        @Override
        public InputStream newInputStream(Path path, OpenOption... options) {
            String pathString = path.toString();
            String contents = fileContents.get(pathString);
            byte[] bytes = contents.getBytes();
            return new ByteArrayInputStream(bytes);
        }
    }

    static class ProcessingFileEventStub implements Consumer<MessageDigestUtilityInverted.ProcessingFileEvent> {
        List<String> events = new ArrayList<>();

        @Override
        public void accept(MessageDigestUtilityInverted.ProcessingFileEvent processingFileEvent) {
            Path path = processingFileEvent.file;
            long size = processingFileEvent.size;
            String message = String.format("Processing file '%s' of size %d bytes", path, size);
            events.add(message);
        }
    }
}
```
