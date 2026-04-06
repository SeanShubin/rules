# Sean's process for using AI assistants as a human

## Flow
- Give the AI a task
- Interrogate the AI responses
  - Verify all truth claims you don't already know
  - Push back against all reasoning you don't already agree with
  - Clarify all suggestions you don't already understand
  - Identify facts and suggestions that were omitted
- Decide which factual claims to accept and which decisions follow
  - Make sure the AI sticks to the facts and decisions you already established with it
  - When the AI deviates (and it will), get it to generate the proper responses, possibilities include:
    - remind it of the relevant facts and decisions
    - explain to it why it is wrong
    - write it yourself
  - Review the git diff
    - Approve of each AI change individually, rejecting or using tab to add context as necessary
    - Or review the entire git diff at the end
    - Either way, repeat until the code looks like what you would have written given the facts and decisions you have verified
    - You are just as responsible for commits you push whether you
      - typed the code yourself
      - copied and pasted the code from the internet
      - prompted the code into existence with an AI Assistant

## Where to put context
- Best: in the code via names and abstraction boundaries
- Second best: in your skill plugin (global, reusable across projects)
- Third best: in project CLAUDE.md files (project-specific, persistent)
- Fourth: in memory files (facts about you/project/feedback, persistent across sessions)
- Last: in your current claude session (ephemeral, lost after conversation)
- Reminders
  - Subagents don't inherit your context, so delegate only tasks that don't need it
  - Goal is protecting context from noise, not reducing context itself
    - Agents are for shielding you from irrelevant search results
    - Not for avoiding work that requires understanding
  - The assistant should be hands-on with the primary task, delegating only ancillary work
    - Do work that requires judgment and accumulated understanding in the assistant
    - The assistant builds understanding by doing the work directly

## Repetition Examples
- The AI can be run repetitively, if, and only if, you can establish a feedback loop based on an objectively verifiable result.
  - Disassembling Java bytecode into a tree
    - "Look at how I have disassembled the code attribute into a tree structure according to section 4.7.3 of the Java Virtual Machine Specification Java SE 26 edition"
    - "Implement this same capability with the other 30 subsections of section 4.7"
    - "Verify by disassembling the entire jvm library and making sure no byte is unaccounted for"
  - Removing dependency cycles
    - "Check the root readme of my SeanShubin/code-structure project on github"
    - "This application runs during the mvn verify phase of this project"
    - "Reorganize the code such that this application no longer reports dependency cycles"
