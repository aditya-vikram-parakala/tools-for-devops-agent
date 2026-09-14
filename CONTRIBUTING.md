# Contributing Guidelines

Thank you for your interest in contributing to our project. Whether it's a bug report, new feature, correction, or additional
documentation, we greatly value feedback and contributions from our community.

Please read through this document before submitting any issues or pull requests to ensure we have all the necessary
information to effectively respond to your bug report or contribution.

## General Guidelines

Before contributing a new tool or to an existing tool (skill, custom agent, MCP server or any other contribution), make sure you check the following:

1. Check existing tools, issues, and pull requests, to make sure that the tool or functionality you want to contribute doesn't already exist. Note that a new tool might sometimes better fit as an update to an existing tool
2. Create a setup in your AWS account, that can be used to test your tool (meaning, create the relevant AWS resources and simulate relevant scenarios)
3. Check if DevOps Agent already has the capability for which you'd like to build the tool (it might be capable of doing what you planned without it). Test it yourself without the tool (more details in the tool-specific guidelines below)
4. If you have concluded that a new tool or update to an existing tool is required, fork the repository, clone the fork, create a [tool request GitHub issue](https://github.com/aws/tools-for-devops-agent/issues/new?template=community-request.yml), and start writing your tool
5. Follow tool-specific guidelines below. Note - in some cases, you may build multiple tools that fit together to serve a specific purpose. For example, a skill for an operations review with a custom agent to run it on schedule, or a skill for investigation of issues in a specific AWS service, with an MCP server that gives it access to additional data sources. In those cases, follow the tool-specific guidelines for each of the tools you intend to contribute
6. Once you're happy with the result, create a PR (see [Contributing via Pull Requests](CONTRIBUTING.md#contributing-via-pull-requests)). A maintainer will review your content


### Build the Tool

Here are some common guidelines for building tools for this repo. Before you begin, review the following docs, depending on the tools you plan to build:

1. If you build a skill, review the [DevOps Agent skills documentation](https://docs.aws.amazon.com/devopsagent/latest/userguide/about-aws-devops-agent-devops-agent-skills.html) and the [Agent Skills specification](https://agentskills.io/home)
2. If you build a custom agent, review the [DevOps Agent custom agents documentation](https://docs.aws.amazon.com/devopsagent/latest/userguide/working-with-devops-agent-custom-agents-index.html) and the [AGENTS.md format docs](https://agents.md/), to understand how to properly write a custom agent
3. If you build an MCP server, review the [MCP documentation](https://modelcontextprotocol.io), and make yourself familiar with the [process of connecting MCP servers to DevOps Agent](https://docs.aws.amazon.com/devopsagent/latest/userguide/configuring-integrations-and-knowledge-connecting-mcp-servers.html)

Follow these guidelines for each tool:

1. Create a directory for your skill, custom agent or MCP server, inside the `skills/`, `custom-agents/`, or `mcp/` directory (respectively)
2. If you're building a skill, decide which DevOps Agent subagents are relevant to your skill (e.g., Chat tasks, Incident RCA). If you're building a custom agent, decide which tools and skills are relevant to it
3. Start building your skill, custom agent or MCP server, according to the documentation referenced above. If you're working with an AI tool like Kiro or Claude Code, it would be a good idea to include those links in your prompt
4. Document your tool in a `README.md` file inside your tool's directory
5. Create a `CHANGELOG.md` file for your tool, inside your tool's directory
6. Include a non-production disclaimer in your tool's `README.md` file. Add a note stating that it's a sample code, not intended for production use without additional review and testing, and that users should validate in a non-production environment first

## Tool-Specific Guidelines

### Skills

#### Skill Metadata

Make sure the frontmatter in the `SKILL.md` file includes a `metadata` block with `version`, `author` (your GitHub user) and `aws-devops-agent-skills.*` fields (see examples in existing skills)

#### Test Your Skill

1. Test relevant scenarios with DevOps Agent, with and without skill, to understand what good looks like. Iterate several times and make changes as necessary. Make sure you test relevant functionalities. For example, if your skill is intended for DevOps Agent investigations related to RDS PostgreSQL, then set up relevant AWS resources, simulate issues, and then start DevOps Agent investigations and evaluate the root cause with and without skill. Another example is, if your skill is intended to generate a report using DevOps Agent chat (such as EKS operations review), then set up relevant AWS resources, simulate scenarios that you expect your report to highlight, and then start DevOps Agent chat and evaluate its response with and without skill. Some tips for what you should check when evaluating your skill: for investigation, evaluate the different investigation root cause parts in the "Root cause" investigation tab (root cause, key findings, gaps, evidence) for correctness and quality across multiple iterations with and without skill. For chat, evaluate the chat's final response for actionability, correctness and completeness, with and without skill. Compare with and without skill runs and iterate to see if there are improvements. In both chat and investigation, evaluate multiple iterations for output consistency (investigation root cause or chat final response).

2. Test your skill with our skill evaluation tool. This tool isn't yet published in this repo. If you're an AWS employee, please follow our internal guidelines or contact us. If you're an external contributor, please tag `@aws/tools-for-devops-agent-admins` GitHub team in your GitHub issue or PR, and we'll help you evaluate your skill. The skill evaluation tool tests skills which are intended for chat and investigation, across different types of tests: structure tests (SKILL.md format and fields format), best practices tests (SKILL.md and dir structure best practices), and functional tests (with real DevOps Agent space, testing relevant parts of the investigation and chat outputs against different criteria, similar to the examples above). The tests that are done by this tool are inspired from the [Agent Skill spec](https://agentskills.io/home).

#### Keeping a Skill Fresh

Skills reference AWS APIs, thresholds, and documentation that drift over time. A scheduled workflow ([`.github/workflows/skill-staleness-reminder.yml`](.github/workflows/skill-staleness-reminder.yml)) runs on the 1st of every other month and, for any skill whose `skills/<name>/` directory has had no commit in the last 60 days, opens a `skill-freshness` issue. Each new issue is assigned to a random repository maintainer to triage, and the issue body lists the skill's `SKILL.md` frontmatter `author` so the maintainer can reassign to the skill owner to verify.

If an issue lands with you, verify the skill against the checklist in it. When it's still accurate, bump the patch version in `SKILL.md` and add a `CHANGELOG.md` line noting the review — any commit touching the skill directory resets the clock and stops the reminder next cycle.

### Custom Agents

#### Test Your Custom Agent

Test relevant scenarios with and without the custom agent, multiple times. Focus on quality and consistency of the output, compared to asking DevOps Agent the same question using chat. When checking consistency, the output doesn't have to be the same verbatim - focus on the substance

## Reporting Bugs/Feature Requests

We welcome you to use the GitHub issue tracker to report bugs or suggest features.

When filing an issue, please check existing open, or recently closed, issues to make sure somebody else hasn't already
reported the issue. Please try to include as much information as you can. Details like these are incredibly useful:

* A reproducible test case or series of steps
* The version of our code being used
* Any modifications you've made relevant to the bug
* Anything unusual about your environment or deployment


## Contributing via Pull Requests
Contributions via pull requests are much appreciated. Before sending us a pull request, please ensure that:

1. You are working against the latest source on the *main* branch.
2. You check existing open, and recently merged, pull requests to make sure someone else hasn't addressed the problem already.
3. You open an issue to discuss any significant work - we would hate for your time to be wasted.

To send us a pull request, please:

1. Fork the repository.
2. Modify the source; please focus on the specific change you are contributing. If you also reformat all the code, it will be hard for us to focus on your change.
3. Ensure local tests pass.
4. Commit to your fork using clear commit messages.
5. Send us a pull request, answering any default questions in the pull request interface.
6. Pay attention to any automated CI failures reported in the pull request, and stay involved in the conversation.

GitHub provides additional documentation on [forking a repository](https://help.github.com/articles/fork-a-repo/) and
[creating a pull request](https://help.github.com/articles/creating-a-pull-request/).


## Finding contributions to work on
Looking at the existing issues is a great way to find something to contribute on. As our projects, by default, use the default GitHub issue labels (enhancement/bug/duplicate/help wanted/invalid/question/wontfix), looking at any 'help wanted' issues is a great place to start.


## Code of Conduct
This project has adopted the [Amazon Open Source Code of Conduct](https://aws.github.io/code-of-conduct).
For more information see the [Code of Conduct FAQ](https://aws.github.io/code-of-conduct-faq) or contact
opensource-codeofconduct@amazon.com with any additional questions or comments.


## Security issue notifications
If you discover a potential security issue in this project we ask that you notify AWS/Amazon Security via our [vulnerability reporting page](https://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public github issue.


## Licensing

See the [LICENSE](LICENSE) file for our project's licensing. We will ask you to confirm the licensing of your contribution.

By submitting this pull request, I confirm that my contribution is made under the terms of the Apache License 2.0.
