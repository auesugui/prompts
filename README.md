# Pragmatic Coordination with Limited Domain Knowledge

Given your realistic constraints, here's a pragmatic approach to coordinate with your subagents while working with limited domain knowledge:

## Create a "Just-in-Time" Knowledge Discovery Framework

**Establish Information Gathering Protocols**
Instead of trying to have all domain knowledge upfront, create structured ways to quickly gather context when needed:

- **User Story Expansion Sessions**: Before sprint planning, schedule 30-minute sessions with your product owner to expand on vague tickets. Use the technical collaboration framework from the role definition to ask structured questions.

- **Domain Expert Access Points**: Identify which users are most accessible for quick clarifications and establish "office hours" or Slack channels for rapid domain questions.

## Implement Progressive Context Building

**Start with What You Know**
- Document the domain knowledge you do have, even if incomplete
- Create a shared knowledge base that grows with each sprint
- Have subagents contribute their learnings back to this shared resource

**Use Assumption-Driven Development**
- When domain context is unclear, document your assumptions explicitly
- Share these assumptions with subagents so they can build with the same understanding
- Create feedback loops to validate or correct assumptions quickly

## Leverage Your Subagents as Knowledge Multipliers

**Distribute Domain Discovery**
- Assign different subagents to become "mini-experts" in specific user personas or workflow phases
- Have your Frontend Engineer focus on understanding user experience pain points
- Have your Backend Engineer dig into data relationships and business rules
- Use your Testing Specialist to uncover edge cases that reveal business logic

**Create Cross-Pollination Opportunities**
- Hold brief "domain learning shares" where subagents present what they've discovered
- Use retrospectives to capture domain insights, not just process improvements

## Build Iterative Clarification Processes

**Question Templates for Product Owner Interactions**
Instead of open-ended questions, use structured formats:
- "When [user persona] does [action], what business rules apply?"
- "What happens if [specific scenario] occurs during [workflow phase]?"
- "What regulatory requirements affect [specific feature]?"

**User Feedback Integration**
- Create lightweight ways to capture domain knowledge during user conversations
- Have subagents prepare domain-specific questions when they interact with users
- Document not just what users want, but why they want it (business context)

## Establish "Good Enough" Standards

**Minimum Viable Domain Knowledge**
- Focus on understanding the immediate workflow being built rather than the entire domain
- Identify the 3-4 most critical business rules for each feature
- Understand the user's primary goal and main failure scenarios

**Risk-Based Prioritization**
- Identify which features have the highest business risk if implemented incorrectly
- Spend more discovery time on high-risk features, less on routine functionality
- Use your Testing Specialist to help identify where domain knowledge gaps could cause the most damage

## Create Feedback Loops for Continuous Learning

**Sprint-by-Sprint Knowledge Building**
- End each sprint by documenting what domain knowledge was discovered
- Start each sprint by reviewing what domain questions need answering
- Use your subagents' specialized perspectives to ask better domain questions

**Collaborative Documentation**
- Have subagents contribute to acceptance criteria based on their technical perspective
- Create living documentation that improves with each implementation
- Use code comments and technical documentation to capture business logic discoveries

The key is accepting that you'll never have perfect domain knowledge upfront, but you can create systems that help you and your subagents learn efficiently and share that knowledge effectively. Focus on building processes that make domain discovery faster and more systematic, rather than trying to become a complete domain expert before you start building.
