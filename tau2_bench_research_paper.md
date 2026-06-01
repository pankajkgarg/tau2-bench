# Tau 2 bench research paper

Sections:
Abstract
1 Introduction
2 Related Work
    Benchmarks for Conversational AI Agents.
    User Simulation for Conversational Agents.
    Multi-Agent Benchmarks.
3 τ 2 \tau^{2} -bench: Evaluating Agents in a Dual-Control Environment
    3.1 The Dec-POMDP Formalism
    3.2 Domain and task creation
    3.3 Task evaluation
4 Experiments
    4.1 Agent settings
    4.2 Results
        Pass^k scores.
        Ablation analysis.
        Impact of policy document on performance.
        Impact of number of actions and sub-tasks.
    4.3 How does dual-control impact benchmark reliability?
        User simulator quality evaluation.
        Reliability of the user simulator.
5 Conclusion
Acknowledgements
References
Broader Impact
Appendix
Appendix A Telecom Domain
    A.1 User Persona
    A.2 Example Task
    A.3 Example Trajectory 1: Default mode
    A.4 Example Trajectory 2: No-User mode
Appendix B Verifying Original τ 2 \tau^{2} -bench
    B.1 Verifying Implementation
    B.2 Verifying Tasks
Appendix C Prompts
    C.1 Agent system prompt
        Agent system prompt template
    C.2 User system prompt
        User system prompt template
        Example of a task instruction
Appendix D Domain Policies
    D.1 Verified Airline and Retail policies
        D.1.1 Retail policy
        D.1.2 Airline policy
    D.2 Telecom policy
        D.2.1 Generic telecom policy
        D.2.2 Technical support policy (original)
        D.2.3 Technical support policy (workflow)
        D.2.4 Troubleshooting workflow graphs
Appendix E User Simulator Quality
    E.1 Common Error Types and Failure Modes ( retail )
    E.2 Common Error Types and Failure Modes ( airline )
    E.3 Common Error Types and Failure Modes ( telecom )
    E.4 Illustrative Cases

Files Content:

## Contents
- 1 Introduction
- 2 Related Work
  - Benchmarks for Conversational AI Agents.
  - User Simulation for Conversational Agents.
  - Multi-Agent Benchmarks.
- 3 τ 2 \tau^{2} -bench: Evaluating Agents in a Dual-Control Environment
  - 3.1 The Dec-POMDP Formalism
  - 3.2 Domain and task creation
  - 3.3 Task evaluation
- 4 Experiments
  - 4.1 Agent settings
  - 4.2 Results
    - Pass^k scores.
    - Ablation analysis.
    - Impact of policy document on performance.
    - Impact of number of actions and sub-tasks.
  - 4.3 How does dual-control impact benchmark reliability?
    - User simulator quality evaluation.
    - Reliability of the user simulator.
- 5 Conclusion
- Acknowledgements
- References
- Broader Impact
- Appendix
- Appendix A Telecom Domain
  - A.1 User Persona
  - A.2 Example Task
  - A.3 Example Trajectory 1: Default mode
  - A.4 Example Trajectory 2: No-User mode
- Appendix B Verifying Original τ 2 \tau^{2} -bench
  - B.1 Verifying Implementation
  - B.2 Verifying Tasks
- Appendix C Prompts
  - C.1 Agent system prompt
    - Agent system prompt template
  - C.2 User system prompt
    - User system prompt template
    - Example of a task instruction
- Appendix D Domain Policies
  - D.1 Verified Airline and Retail policies
    - D.1.1 Retail policy
    - D.1.2 Airline policy
  - D.2 Telecom policy
    - D.2.1 Generic telecom policy
    - D.2.2 Technical support policy (original)
    - D.2.3 Technical support policy (workflow)
    - D.2.4 Troubleshooting workflow graphs
- Appendix E User Simulator Quality
  - E.1 Common Error Types and Failure Modes ( retail )
  - E.2 Common Error Types and Failure Modes ( airline )
  - E.3 Common Error Types and Failure Modes ( telecom )
  - E.4 Illustrative Cases

## Abstract

Abstract Existing benchmarks for conversational AI agents simulate single-control environments, where only the AI agent can use tools to interact with the world, while the user remains a passive information provider. This differs from real-world scenarios like technical support, where users need to actively participate in modifying the state of the (shared) world. In order to address this gap, we introduce τ 2 \tau^{2} -bench, with four key contributions:
1) A novel Telecom dual-control domain modeled as a Dec-POMDP, where both agent and user make use of tools to act in a shared, dynamic environment that tests both agent coordination and communication,
2) A compositional task generator that programmatically creates diverse, verifiable tasks from atomic components, ensuring domain coverage and controlled complexity,
3) A reliable user simulator tightly coupled with the environment, whose behavior is constrained by tools and observable states, improving simulation fidelity,
4) fine-grained analysis of agent performance through multiple ablations including separating errors arising from reasoning vs communication/coordination.
In particular, our experiments show significant performance drops when agents shift from no-user to dual-control, highlighting the challenges of guiding users. Overall, τ 2 \tau^{2} -bench provides a controlled testbed for agents that must both reason effectively and guide user actions. 1 1 1 Code and data are available at: https://github.com/sierra-research/tau2-bench .

## 1 Introduction

Existing benchmarks for conversational AI agents are designed to test their abilities to communicate effectively with a user and perform the right sequence of actions to solve tasks yao2024tau; lu2024toolsandbox; xiao2024flowbench; prabhakar2025apigen. These benchmarks are inherently *single-control* environments, where the AI agent is able to interact with the world but the (simulated) user is limited to providing information about preferences and goals. In $\tau$-bench yao2024tau for example, the retail and airline domains test the agent’s ability to solve constraint satisfaction tasks, where constraints stem from a combination of a domain policy that the agent must follow and the user’s cognitive state including beliefs, goals and preferences.

Figure: Figure 1: Supporting dual-control environment in $\tau^{2}$-bench. The agent have access to a set of tools that interact with a database, and is tasked with resolving the user’s request via Tool-Agent-User (TAU) interactions while adhering to the domain policy. To test real-world scenarios, the user is simulated by another AI agent given a scenario-based instruction and a set of tools that interact with its own database. The simulated user can be regarded as handling an easier version of the TAU interaction in a dual format (Tool-User-Agent), where it only need to follow instructions but does not need to reason about solutions for the task.
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x1.png

In such settings, the user’s understanding of the agent’s environment (including the set of actions the agent can take) is provided through carefully crafted natural language instructions that help ensure a single, solvable path for the constraint satisfaction problem. While this enables easier task specification, the user can only perceive the agent’s actions through communication and reason about the environment state solely based on the initial instructions. This is quite different from real-world scenarios like technical customer support, where the user has to actively participate in taking some actions to diagnose and solve problems when asked to do so, such as restarting their phone or turning off airplane mode.

In order to capture this additional real-world complexity, we introduce $\tau^{2}$-bench with a dual-control environment, where (LLM-simulated) users can take actions and call tools in addition to communicating with the agent. The dual-control model provides a few advantages in improving the user simulation, such as selective information hiding, non-verbal manipulation of the environment, and easier specification of the user’s task or scenario, especially in cases where the structured format of tools is preferred to unstructured natural language.

The primary challenge in a dual-control setup lies in granting the user simulator meaningful agency via tools while maintaining the "complexity asymmetry" between the agent and the user simulator – that is, making sure the user’s affordances are limited and the user still requires support from the agent in order to solve issues. While tools for the user simulator could disrupt this balance by increasing its capabilities, we find that carefully designing the tools can help constrain user behavior, leading to more reliable simulations. We achieve this by ensuring user tools yield only human-readable outputs, limiting user planning to reactive tool use based on agent requests, and tightly constraining user behavior through the environment. This results in a controlled testbed for agents that must both think and effectively guide a user to take the right actions.

Overall, $\tau^{2}$-bench makes four key contributions:

Telecom dual-control domain. We introduce a novel dual-control telecom domain within $\tau^{2}$-bench, where, unlike previous benchmarks, both the AI agent and the user possess distinct tools to observe, act upon, and verify the state of a shared, dynamic environment. This is formalized using a Decentralized Partially Observable Markov Decision Process (Dec-POMDP) oliehoek2016concise. This dual-control setup is designed to accurately represent real-world collaborative scenarios and exposes crucial agent coordination and communication challenges absent from evaluations where users have limited agency. Experiments show state-of-the-art LLMs struggle significantly in this domain (e.g., pass $\hat{}$ 1 of 34% for gpt-4.1 gpt41, 42% for o4-mini o4mini, and 49% for claude-3.7-sonnet claude37sonnet on new tasks).

Compositional task generator. $\tau^{2}$-bench incorporates a programmatic task generator that automatically composes a vast and diverse set of verifiable tasks from a small set of atomic base scenarios (defined by initialization, solution, and assertion functions). This method ensures provable correctness of tasks, provides complete domain coverage, allows for explicit control over task complexity (e.g., by number of solution steps or issue type), and removes the manual effort and potential brittleness associated with hand-crafted task suites.

Reliable user simulator. We enhance the reliability of the user simulation by tightly coupling the user simulator to the environment. User behavior is constrained by the available tools and the observable state of the environment, leading to more predictable and consistent interactions. This approach significantly alleviates the need for complex natural language prompting to guide the user simulator and results in substantially higher reliability (e.g., the telecom domain’s user simulator shows a 16% error rate with 6% critical errors, compared to 40% error rate with 12% critical errors in the retail domain from $\tau$-bench).

Fine-grained diagnosis of agent failures. $\tau^{2}$-bench enables a decomposed diagnosis of agent performance by evaluating task success in different modes: (i) a fully autonomous mode (“no-user mode”), where the agent controls all tools, isolating its reasoning capabilities, and (ii) the standard dual-control mode, which introduces communication and coordination requirements. Our findings reveal a substantial performance decrease (around 20% pass $\hat{}$ 1) when agents must shift from autonomous operation to guiding a user. This clearly distinguishes pure reasoning failures from those arising from communication and decentralized control, pinpointing the latter as a critical bottleneck for current agents.

## 2 Related Work

##### Benchmarks for Conversational AI Agents.

Following a long line of research into language agents yao2022webshop; zhou2023webarena; jimenez2023swe; liu2023agentbench; ruan2023identifying, LLM tool use berkeley-function-calling-leaderboard; qin2024toolllm; huang2023metatool, and task-oriented dialog chen2021action; budzianowski2018multiwoz; andreas2020task; schatzmann2007evaluating; gur2018hidden; he2018decoupling; Hu_2023, $\tau$-bench yao2024tau is a recently introduced benchmark to measure the reliability of language agents in multi-turn task-oriented conversations such as customer service workflows, while respecting domain rules. Each task in $\tau$-bench instantiates a live conversation between a user simulator and the language agent, with tasks spread across two domains – retail and airline. To quantify reliability the paper introduces pass $\hat{}$ k metrics: the fraction of k independent runs that succeed.

Several follow-ups to $\tau$-bench have explored variations of the basic setup. FlowBench xiao2024flowbench isolates the planning step of tool-using agents by injecting explicit workflow knowledge into the prompt, using natural language, python-like pseudocode or mermaid flowcharts. IntellAgent levi2025intellagent provides an evaluation pipeline to programmatically build synthetic test suites from structured policy graphs that encode domain rules and their co-occurrence statistics. IntellAgent explicitly uses $\tau$-bench as an external gold standard, reporting a high Spearman correlation between the two score distributions, and acts as a fast, synthetic proxy task. APIGen-MT prabhakar2025apigen explores the idea of fine-tuning tool-calling agents for $\tau$-bench. They generate data by creating conversation blueprints which are sequences of tool calls that depend on each other, followed by simulating conversational traces based on each blueprint. ToolSandbox lu2024toolsandbox focuses on creating stateful tools in order to evaluate agent progress in a more fine-grained manner.

Our work extends the $\tau$-bench paradigm and generalizes it to allow for both the user and agent to have state-changing abilities (via tool calls) over a shared world. As demonstrated in the results, this allows us to build more complex domains to test conversational agents, while also providing the opportunity for fine-grained analysis of agent failure points that can be improved upon.

##### User Simulation for Conversational Agents.

The reliability of user simulation has been a key concern for benchmarks like $\tau$-bench yao2024tau. While most efforts have focused on introducing supervision for the user simulator, for instance, by using a generic LLM to generate or validate user responses prabhakar2025apigen, less attention has been paid to the possibility of using the environment itself to constrain and shape user simulator behavior for increased reliability, a core tenet of our approach. This concern has been extensively studied in the context of task-oriented dialogue systems, with early work by pietquin_survey_2013 providing a comprehensive survey of metrics for evaluating user simulations. More recently, kazi_large_2024 has demonstrated how LLMs can be effectively used as user-agents for evaluating task-oriented dialogue systems, showing that careful prompting and state tracking can lead to more reliable and context-aware user simulations.

##### Multi-Agent Benchmarks.

Our work is also related to efforts to build multi-agent frameworks and evaluate them zhu2025multiagentbench.
While we can consider the user and the agent in our paper as forming a multi-agent system, the key difference in our case is that the final evaluation still focuses on the agent’s ability to elicit the right information from the user and perform the correct actions to solve the task. This introduces an inherent asymmetry between the agent and the user; our focus is not on solving a pure multi-agent problem but rather on the agent’s capability to effectively guide and collaborate with a user who also possesses agency. In this sense, the framework can be collaborative (e.g., troubleshooting), competitive (e.g., negotiating a subscription), or a hybrid, requiring the agent to identify and navigate the scenario appropriately, even accounting for user mistakes or errors.

## 3 τ 2 \tau^{2} -bench: Evaluating Agents in a Dual-Control Environment

$\tau^{2}$-bench serves as a platform for a systematic study of multi-turn interactions between a conversational AI agent and a simulated user. Dual-control interactions are modeled as a Decentralized Partially Observable Markov Decision Process (Dec-POMDP) oliehoek2016concise where both the agent and user can communicate, use tools, and receive observations. This allows us to simulate complex scenarios like technical troubleshooting where agent and user must coordinate their actions to solve the task.

### 3.1 The Dec-POMDP Formalism

As illustrated in [Figure˜1](#S1.F1), the Dec-POMDP in $\tau^{2}$-bench involves two players: an agent and a user. The entire process is formally defined by the tuple $({\mathcal{S}},\{{\mathcal{A}}_{i}\},\{{\mathcal{O}}_{i}\},{\mathcal{T}},{\mathcal{R}},{\mathcal{U}},{\mathcal{M}})$, where $i\in\{agent,user\}$ denotes players and each component in the tuple is detailed below, with illustrative examples drawn from the new telecom domain.

Message space (${\mathcal{M}}$): The set of all possible (natural language) messages exchanged between the agent and the user. For example, the user could say “I cannot use mobile data.” and the agent could respond with “Could you check whether your airplane mode is on?”

State space (${\mathcal{S}}$): The global state ${\mathcal{S}}={\mathcal{S}}_{world}\otimes{\mathcal{S}}_{history}$, where ${\mathcal{S}}_{world}={\mathcal{S}}_{db,agent}\otimes{\mathcal{S}}_{db,user}$ represents the underlying database states for the agent and user, and ${\mathcal{S}}_{history}$ logs all interaction events (actions, observations, messages). For example, in the telecom domain, ${\mathcal{S}}_{db,agent}$ might be CRM data (customer profiles, lines), while ${\mathcal{S}}_{db,user}$ could be phone status.

Action spaces (${\mathcal{A}}_{i}$): Player $i$’s action $a_{i}\in{\mathcal{A}}_{i}$ is either a tool call $a_{i,tool}\in{\mathcal{A}}_{i,tool}$ (interacting with ${\mathcal{S}}_{db,i}$ via function calls like tool_name(**kwargs)) or a message $m_{i}\in{\mathcal{M}}$. Only one player acts per turn. In the telecom domain, the agent can access tools like get_customer_by_id(id) and the user can access tools like toggle_airplane_mode().

Observation spaces (${\mathcal{O}}_{i}$): Player $i$’s observation $o_{i}\in{\mathcal{O}}_{i}$ is either a tool observation $o_{i,tool}$ (e.g., data, messages, or errors from $a_{i,tool}$) or a message $m_{j}\in{\mathcal{M}}$ from player $j\neq i$. Only one player receives an observation per turn. In the telecom domain, the agent might observe customer details from get_customer_by_id, and the user might observe a message indicating the airplane mode has been turned off from toggle_airplane_mode.

Transition function (${\mathcal{T}}$): Defines system dynamics via ${\mathcal{T}}:{\mathcal{S}}\times{\mathcal{A}}\to{\mathcal{S}}\times{\mathcal{O}}$. Given current state $s\in{\mathcal{S}}$ and joint action $a=(a_{agent},a_{user})$, it yields a new state $s^{\prime}\in{\mathcal{S}}$ and joint observation $o=(o_{agent},o_{user})$. Calling a tool $a_{i,tool}\in{\mathcal{A}}_{i,tool}$ may change ${\mathcal{S}}_{world}$ and yield $o_{i}\in{\mathcal{O}}_{i,tool}$. Sending a message $m_{i}\in{\mathcal{M}}$ yields $o_{j}=m_{i}$ for $j\neq i$. In both cases, $s^{\prime}$ includes updated ${\mathcal{S}}_{world}$ and ${\mathcal{S}}_{history}$. For example, an agent’s action enable_roaming(customer_id, line_id) would update the world state (the roaming service for the specific line number is enabled), and a user’s action toggle_airplane_mode would update the status of the mocked phone.

Reward function (${\mathcal{R}}$): A function ${\mathcal{R}}:{\mathcal{S}}\to[0,1]$ providing a global reward based on the overall state $s\in{\mathcal{S}}$ (database states, history), signaling task success or failure. For example, in telecom, the agent is rewarded if a user’s issue ("no mobile data") is fixed, as verified by the user’s database state.

Instruction space (${\mathcal{U}}$): The instruction space ${\mathcal{U}}$ defines the scenario guiding realistic user simulation, as well as the domain policies to which the agent must adhere when assisting the user.

The Dec-POMDP formalism offers key advantages for simulating complex, interactive scenarios (see [Figure˜2](#S3.F2) for an example trajectory of interactions).
It enables realistic simulation of collaborative environments, such as technical support, where users execute tool actions guided by the agent. This presents agents with crucial coordination and communication challenges.
Additionally, the formalism enhances user simulation reliability and control; by defining user tools and their effects on the user state, user behavior becomes more predictable and less reliant on extensive natural language prompting.

Figure: Figure 2: An example agent-user interaction trajectory (${\mathcal{S}}_{history}$) of $\tau^{2}$-bench in the telecom domain. By controlling the implementation of the user tools (the mocked phone), we can reliably simulate the user’s response to agent’s actionable instructions like “checking the status bar” and “toggling data” based on the underlying database state. On the right half, we show the possibility of modeling the impact of agent’s tool calls on the user’s database state, where the roaming service for the user is enabled on the agent’s end and therefore allows the user’s phone to roam.
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x2.png

### 3.2 Domain and task creation

Table: Table 1: Key statistics for the $\tau^{2}$-bench domains.

Similar to $\tau$-bench, we adopted a multi-stage creation process to build domain-specific materials for new domains. This process, illustrated using the telecom domain, involves the following stages:

Stage 1: Creating agent’s database schema and tools.
We begin by prompting Large Language Models (LLMs) to generate a Product Requirements Document (PRD) that outlines the domain’s core business logic. This PRD specifies the database schema and necessary functions. In the telecom domain, this involved defining a customer CRM system with schemas for customers and lines, along with functions to manage them. An LLM then generates function implementations, a mock database, and unit tests based on the PRD. We manually refine the generated code until all unit tests pass.

Stage 2: Creating user’s database schema and tools.
For troubleshooting scenarios, we similarly use an LLM to define the user’s database schema and tools. In the telecom domain, this included implementing a mocked user phone device with status (e.g., signal strength) and functions (e.g., toggling airplane mode). Again, an LLM generates implementations, a mock database, and unit tests, which are then manually refined until all tests pass.

Stage 3: Programmatic task creation.
We employ a combinatorial approach to generate diverse, verifiable tasks from atomic building blocks.

Each atomic subtask $t$ is about a specific problem to be resolved, for example, airplane mode on leads to mobile data not working. Specifically, each subtask $t$ is defined as $(\{f_{t,k}^{init}\},\{f_{t,k}^{sol}\},\{f_{t,k}^{assert}\})$, where $f_{t,k}$ is the $k$-th function call of the subtask $t$ that interacts with the agent’s or user’s database:

- •
Initialization functions $f_{t,k}^{init}$ specify calls to set up the initial task state, typically by updating the database values. For instance, in telecom, an initialization might be set_airplane_mode(True).
- •
Solution functions $f_{t,k}^{sol}$ specify tool calls to resolve issues introduced by initialization. For example, toggle_airplane_mode() could be a solution for the initialization example given above. Note that these must be tools available to the agent or user.
- •
Assertion functions $f_{t,k}^{assert}$ specify conditions the final state ${\mathcal{S}}$ must meet for the task to be considered solved. For instance, assert_service_status("connected") checks if the user’s service is active in telecom.

While solution functions $f_{t,k}^{sol}$ are restricted to agent or user tools, initialization and assertion functions can be any function in the relevant database.

Atomic subtasks are grouped such that mutually exclusive or alternative subtasks are in the same group. A composite task is created by selecting at most one subtask from each group, concatenating their respective function calls.
Task correctness is automatically verified by checking if the final state $s\in{\mathcal{S}}$ satisfies all assertion functions after applying initialization and then solution functions. We also verify that the task is not resolved until all solution functions are applied.

In the telecom domain, we developed 15 atomic subtask groups for 3 user intents of increasing complexity: service_issue, mobile_data_issue, and mms_issue.
Combining these subtasks in a programatic way yields 2285 tasks. We then subsample 114 tasks to form a balanced distribution over different intents and numbers of subtasks (details in [Appendix˜A](#A1)). The number of subtasks in a task serves as a proxy for difficulty as more diagnostic and resolution steps are required.

Stage 4: Creating domain-specific agent policy.
Based on the curated tasks and their solutions, we prompt LLMs to generate domain-specific policies for the agent. For troubleshooting, these policies guide the agent in diagnosing and resolving user issues, often outlining step-by-step procedures for common problems related to each user intent, details in [Section˜D.2](#A4.SS2).

Stage 5: Manual refinement. We jointly refine all the domain materials including tools, policy and atomic subtasks to improve the quality of the domain.

Moreover, compared to the original $\tau$-bench, $\tau^{2}$-bench enables a developer to specifically associate a task with a Persona (a brief description of the user’s identity).
We put this to use in our new domain. Each telecom domain task was randomly assigned one of the following personas: None, Easy, and Hard. The None persona means that no specific persona is provided to the user simulator. The Easy persona describe the profile of a user who is rather familiar with the domain while the Hard persona represents a more challenging user with low technical knowledge (see [Section˜A.1](#A1.SS1)).

### 3.3 Task evaluation

The success of a task can be defined by different criteria: DB check, status assertions, natural language assertions, communication info check, and action matching.
The DB check and communication info check are the same as the original $\tau$-bench. The status assertion involves verifying specific conditions in the final world state ${\mathcal{S}}_{world}$ using the predefined assertion functions (e.g., checking if a service is connected). The natural language assertion involves verifying specific conditions in the final history state ${\mathcal{S}}_{history}$ using a natural language description, like “the agent diagnosed the cause of the issue.” The action matching involves verifying if every solution function $f_{t,k}^{sol}$ exists in the actual agent-user interaction trajectory.
Practically, each task can specify a subset of these criteria based on its features. In telecom, only assertion functions are used to evaluate task success.

## 4 Experiments

### 4.1 Agent settings

All LLM API calls are implemented using the Litellm package litellm.
We evaluated four large language models: gpt-4.1-mini-2025-04-14, gpt-4.1-2025-04-14, o4-mini-2025-04-16, and claude-3-7-sonnet-20250219.
The user simulator is implemented using gpt-4.1-2025-04-14.
Each task is run four times, maintaining a consistent LLM temperature of 0 to promote deterministic outputs.
Both the agents and the user simulator are implemented as function-calling agents. All tools are provided to LLMs in the OpenAI tools format.
The agent prompt includes generic guidelines along with domain-specific policies. Similarly, the user prompt contains generic guidelines supplemented by task-specific instructions. Both domain policies and prompts are available in [Appendices˜D](#A4) and [C](#A3).

When the gpt-4.1-2025-04-14 agent is paired with the gpt-4.1-2025-04-14 user simulator, the average agent/user simulation costs are $0.086/$0.059 per task, respectively.
The cost of running all domains for 1 trial per task is approximately $40.

### 4.2 Results

##### Pass^k scores.

Figure: Figure 3: pass $\hat{}$ k metrics across all evaluated domains (airline, retail, telecom). Top Left: gpt-4.1, Top Right: o4-mini, Bottom Left: gpt-4.1-mini, Bottom Right: claude-3.7-sonnet.
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x3.png

We computed performance metrics on the verified $\tau^{2}$-bench domains (retail and airline) and on our new telecom domain (see [Figure˜3](#S4.F3)).
Our findings indicate that the telecom domain presents a greater challenge, exhibiting an overall lower success rate compared to other domains. gpt-4.1 pass $\hat{}$ 1 drops from 74%/56% for retail and airline respectively to 34% for telecom.
gpt-4.1-mini, o4-mini, and claude-3.7-sonnet perform better with pass $\hat{}$ 1 of around 50% for telecom.
In the case of claude-3.7-sonnet, the pass $\hat{}$ 1 score for telecom (49%) is on par with airline. However, as k increases, the pass $\hat{}$ k scores decline more rapidly for telecom compared to airline, suggesting less consistent performance on the telecom domain.

##### Ablation analysis.

An agent’s success in $\tau^{2}$-bench depends on two things – 1) how well it can communicate and collaborate with the user at solving the issue, and 2) how well it can reason over and apply the domain guidelines specified in the policy document. In order to understand the impact of each of these components—reasoning and communication for dual control—we perform an ablation study. Specifically, we evaluate performance in the telecom domain across three distinct settings:

- •
Default: The default agent and user simulator configuration where the agent and user collaborate in a dual-control setup.
- •
No-User: The agent is provided with a ticket summarizing the user’s problem and success criteria. The agent controls all tools, including those typically operated by the user, and is solely responsible for solving the problem. This setting tests the agent’s reasoning and tool-calling capability independently of its capacity to interact with the user.
- •
Oracle Plan: The agent is provided with the sequence of tool calls required to solve the problem, encompassing actions for both the agent and the user. This setup alleviates the agent’s reasoning load, focusing on its ability to collaborate with the user to execute a known plan.

[Figure˜4](#S4.F4) (left) reports performance across these settings for gpt-4.1 and o4-mini, revealing key insights.
The difference between Oracle Plan and Default configurations highlights the impact of the reasoning load on agent performance.
Unsurprisingly, providing the ground truth leads to better performance than the Default setting. But it is notable that this effect is larger for the o4-mini than for gpt-4.1, suggesting that o4-mini is better able to make use of the ground truth information.

Figure: Figure 4: pass $\hat{}$ 1 metrics for the Telecom domain across different operational modes (Default, No-User, Oracle Plan) using the Default user simulation. Left: original policy. Right: workflow-based policy. This figure illustrates the impact of reasoning load and decentralized control on agent performance.
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x7.png

The comparison between No-User and Default modes illustrates the impact of dual control and the associated communication overhead on agent error rates and overall success. For both models, shifting from no user operation (No-User) to a collaborative setup (Default) where the agent must guide the user results in a substantial drop in pass $\hat{}$ 1 (18% drop for gpt-4.1 and 25% drop for o4-mini).
This underscores that LLMs still face significant challenges when solving problems with an active user who shares control of the environment.

##### Impact of policy document on performance.

Figure [4](#S4.F4) also allows us to analyze the impact of the policy document on task success. Specifically, we created an alternate policy document that provides more specific details on the workflow required to solve each type of task, with the rationale that having the workflow provides more concrete guidance to the agent (see [Sections˜D.2.2](#A4.SS2.SSS2) and [D.2.3](#A4.SS2.SSS3)). We observe that this is indeed the case and slightly improves agent performance using the workflow policy (right) over the original one (left) under the Default and No-User modes. Surprisingly, workflow policy document hurts performance on Oracle Plan for both gpt-4.1 and o4-mini. Our hypothesis here is that since the agent already has the ground truth action sequence, providing it the workflow might lead to confusion and hurt its performance rather than help.

##### Impact of number of actions and sub-tasks.

Figure: Figure 5: pass $\hat{}$ 1 scores across various tasks in telecom binned by the number of actions required to solve them (left) or the number of different issues that need to be addressed (right). transfer refers to the special case of a task that requires to be transferred to a human and cannot be solved by the agent alone. (Grey bars indicates the proportion of the tasks that fall into that bin.)
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x9.png

Figure [5](#S4.F5) breaks down the pass $\hat{}$ 1 scores across various tasks in telecom binned by the number of actions required to solve them (left) or the number of different sub-tasks that need to be addressed (right). transfer refers to the special case of a task that requires to be transferred to a human and cannot be solved by the agent alone.

As expected, regardless of the base model being gpt-4.1 or o4-mini, agent performance drops as the number of actions increases, reaching close to zero for $>7$ actions in Default mode. The No-User mode results in higher scores overall, although the gap reduces (from about 0.3-0.4 to <0.2) as the number of actions increases. This hints that maintaining reliability over longer-horizon tasks remains a challenge under both settings and communication with the user is not the only bottleneck. Interestingly, for No-User, gpt-4.1 performs better at the tail end (10 actions or more) than o4-mini.

We observe a similar trend with increase in the number of distinct sub-tasks per task – performance trends downwards for both base models, with the no-user mode generally being higher than the default mode. Both these results validate that our domain design and task creation process provide a natural path to scaling complexity via increasing action length or combining different sub-tasks into a single task.

Impact of issue types.
The telecom domain is organized around three primary user intents reflecting the three different issues that can be encountered: service_issue, mobile_data_issue, and mms_issue, each of which contains specific procedures laid out by the domain policy.
These issue types are designed to have an inherent difficulty hierarchy. For instance, service_issue tasks can typically be resolved independently through a straightforward sequence of actions. In contrast, successfully addressing mobile_data_issue or mms_issue often requires first checking for and potentially resolving underlying service_issue problems. This dependency creates a natural ordering in task difficulty, with service_issue being the easiest, while mobile_data_issue and mms_issue represent more complex, multi-stage problems.

Figure: Figure 6: gpt-4.1 (top left), o4-mini (top right), gpt-4.1-mini (bottom left), and claude-3.7-sonnet (bottom right) pass $\hat{}$ k per issue type for the telecom domain. Performance is shown for service_issue, mobile_data_issue, and mms_issue issue types, highlighting how different issue types affect success rates.
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x11.png

[Figure˜6](#S4.F6) provides an breakdown analysis of performance by issue type. We observe that the pass $\hat{}$ k scores for all LLMs (gpt-4.1, o4-mini, claude-3.7-sonnet, gpt-4.1-mini) are driven by higher failure rates on more complex issue types (mobile_data_issue and mms_issue). This suggests that the multi-stage reasoning and conditional logic required for harder issue types pose a substantial challenge to the agents. We also notice that the spread across issue types differs slightly by the model. For instance, claude-3.7-sonnet does better than o4-mini on mobile_data_issue but worse on service_issue.

Figure: Figure 7: gpt-4.1 (top left), o4-mini (top right), gpt-4.1-mini (bottom left), and claude-3.7-sonnet (bottom right) pass $\hat{}$ k per persona type for the telecom domain. Performance is shown across different persona types, highlighting how different user characteristics affect success rates.
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x15.png

Impact of user persona.
[Figure˜7](#S4.F7) provides an breakdown analysis of performance by user persona.
Results confirm that the agent tends to perform better on tasks associated with the Easy persona compared to those associated with the Hard one. Interestingly, the performances of the agent on tasks involving no persona information (None) tend to be be on par or lower to performances on tasks associated with the Hard persona. This highlights the critical importance of testing AI systems with well-defined user personas before real-world deployment.

### 4.3 How does dual-control impact benchmark reliability?

Ensuring the reliability of conversational agent benchmarks is paramount. Three primary sources of uncertainty can impact benchmark reliability: implementation errors, task specification errors, and user simulator errors. While the user simulator is often cited as a critical component requiring careful evaluation, its assessment can be confounded by issues in the benchmark’s implementation or task definitions. Therefore, we first address these potential error sources before evaluating the user simulator itself. This is detailed in [Appendix˜B](#A2).

##### User simulator quality evaluation.

Having minimized errors in the benchmark implementation and task specifications, we evaluated the user simulator quality.

To assess the quality of the user simulator across domains, we manually annotated interaction traces generated using gpt-4.1 for both the User Simulator and the Agent.

Each conversation was reviewed by two separate annotators tasked with identifying user simulator errors.
Annotators were given the User Simulator Guidelines, the specific User Instructions for this conversation (see [Section˜C.2](#A3.SS2)), descriptions of the available User Tools (if any), and the complete conversation trajectory (messages and tool calls). Annotators assessed each user turn against four criteria: adherence to User Simulator Guidelines, adherence to User Instructions, correct use of User Tools, and generation of a natural and consistent conversational continuation.

Errors were categorized as either (1) task-critical errors: high-severity failures that preclude task completion (e.g., generating an intent that contradicts the user goal, or causing an irrecoverable state transition), or as (2) task-benign errors: Errors that do not prevent the task from being completed.

##### Reliability of the user simulator.

Table: Table 2: User simulator error count (rates) across domains. Only critical user errors prevent the agent from solving the task while benign errors do not affect task completion. New telecom domain shows much lower error rate and no critical errors were reported. (See [Appendix˜E](#A5) for additional details).

As shown in [Table˜2](#S4.T2), our analysis of user simulator behavior reveals significant improvements in reliability for the new telecom domain. While for the retail and airline domains we recorded a 40% and 47% error rate for the user simulator (with 12% and 13% being critical errors that prevent task completion), this rate is much lower for the telecom domain, only 16% with 6% critical errors reported. This substantial improvement in reliability can be attributed to the domain design, which shapes and tightly constrains user behavior through its environment and available affordances. Rather than relying heavily on natural language specifications to guide behavior, the telecom domain’s structured interface and clear action space naturally guide the user simulator toward correct interactions, resulting in more consistent and predictable behavior.

## 5 Conclusion

We present $\tau^{2}$-bench that generalizes $\tau$-bench by introducing the dual-control setting and found a substantial performance drop in LLMs due to coordination and communication requirements, highlighting these as critical bottlenecks over pure reasoning capabilities for solving user requests.

Works remains to be done to improve the user simulator. Although we have shown that augmenting users with curated tools can help avoid critical errors, we have not yet investigated how this method could be applied to the existing airline and retail domains. Doing so would pave the way towards a more generic solution to ensuring high quality user simulator.

Extending domain coverage for the benchmark still heavily relies on human experts. For benchmarking methods to be adopted by industry, providing much needed standards, it is critical to further investigate how to automate the domain curation process.

One important limitation of $\tau^{2}$-bench is that it does not explicitly model the expert-novice gap inherent to most customer support tasks. When interacting with a naive user, an expert must understand the user’s mental model and adapt explanations accordingly. Assessing and improving the AI agent’s abilities to bridge this gap is a promising direction for future work and $\tau^{2}$-bench provides a strong starting point for such explorations.

## Acknowledgements

We thank Noah Shinn, Dongxu Zhou, Ola Zytek and Keshav Dhandhania for all the fruitful discussions and feedback that strengthened this work, Kevin Gao, Nate White and Kimberly Patron for helping set up the different LLM APIs for the experiments, and Clay Bavor for his unwavering support.

## References

- [1]
Jacob Andreas, John Bufe, David Burkett, Charles Chen, Josh Clausman, Jean
Crawford, Kate Crim, Jordan DeLoach, Leah Dorner, Jason Eisner, et al.
Task-oriented dialogue as dataflow synthesis.
Transactions of the Association for Computational Linguistics,
8:556–571, 2020.
- [2]
Anthropic.
Claude 3.7 Sonnet, 2025.
Model release: 2025-02-24.
- [3]
BerriAI.
litellm, 2025.
- [4]
Paweł Budzianowski, Tsung-Hsien Wen, Bo-Hsiang Tseng, Inigo Casanueva,
Stefan Ultes, Osman Ramadan, and Milica Gašić.
Multiwoz–a large-scale multi-domain wizard-of-oz dataset for
task-oriented dialogue modelling.
arXiv preprint arXiv:1810.00278, 2018.
- [5]
Derek Chen, Howard Chen, Yi Yang, Alex Lin, and Zhou Yu.
Action-based conversations dataset: A corpus for building more
in-depth task-oriented dialogue systems.
arXiv preprint arXiv:2104.00783, 2021.
- [6]
Izzeddin Gür, Dilek Hakkani-Tür, Gokhan Tür, and Pararth Shah.
User modeling for task oriented dialogues.
In 2018 IEEE Spoken Language Technology Workshop (SLT), pages
900–906, 2018.
- [7]
He He, Derek Chen, Anusha Balakrishnan, and Percy Liang.
Decoupling strategy and generation in negotiation dialogues.
arXiv preprint arXiv:1808.09637, 2018.
- [8]
Zhiyuan Hu, Yue Feng, Anh Tuan Luu, Bryan Hooi, and Aldo Lipani.
Unlocking the potential of user feedback: Leveraging large language
model as user simulators to enhance dialogue system.
In Proceedings of the 32nd ACM International Conference on
Information and Knowledge Management, CIKM ’23. ACM, October 2023.
- [9]
Yue Huang, Jiawen Shi, Yuan Li, Chenrui Fan, Siyuan Wu, Qihui Zhang, Yixin Liu,
Pan Zhou, Yao Wan, Neil Zhenqiang Gong, et al.
Metatool benchmark for large language models: Deciding whether to use
tools and which to use.
arXiv preprint arXiv:2310.03128, 2023.
- [10]
Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir
Press, and Karthik Narasimhan.
Swe-bench: Can language models resolve real-world github issues?
arXiv preprint arXiv:2310.06770, 2023.
- [11]
Taaha Kazi, Ruiliang Lyu, Sizhe Zhou, Dilek Hakkani-Tur, and Gokhan Tur.
Large Language Models as User-Agents for Evaluating
Task-Oriented-Dialogue Systems, November 2024.
arXiv:2411.09972.
- [12]
Elad Levi and Ilan Kadar.
Intellagent: A multi-agent framework for evaluating conversational ai
systems.
arXiv preprint arXiv:2501.11067, 2025.
- [13]
Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu,
Hangliang Ding, Kaiwen Men, Kejuan Yang, et al.
Agentbench: Evaluating llms as agents.
arXiv preprint arXiv:2308.03688, 2023.
- [14]
Jiarui Lu, Thomas Holleis, Yizhe Zhang, Bernhard Aumayer, Feng Nan, Felix Bai,
Shuang Ma, Shen Ma, Mengyu Li, Guoli Yin, et al.
Toolsandbox: A stateful, conversational, interactive evaluation
benchmark for llm tool use capabilities.
arXiv preprint arXiv:2408.04682, 2024.
- [15]
Frans A Oliehoek, Christopher Amato, et al.
A concise introduction to decentralized POMDPs, volume 1.
Springer, 2016.
- [16]
OpenAI.
gpt-4.1, 2025.
Model release: 2025-04-14.
- [17]
OpenAI.
o4-mini, 2025.
Model release: 2025-04-16.
- [18]
Olivier Pietquin and Helen Hastie.
A survey on metrics for the evaluation of user simulations.
The knowledge engineering review, 28(1):59–73, 2013.
Publisher: Cambridge University Press.
- [19]
Akshara Prabhakar, Zuxin Liu, Weiran Yao, Jianguo Zhang, Ming Zhu, Shiyu Wang,
Zhiwei Liu, Tulika Awalgaonkar, Haolin Chen, Thai Hoang, et al.
Apigen-mt: Agentic pipeline for multi-turn data generation via
simulated agent-human interplay.
arXiv preprint arXiv:2504.03601, 2025.
- [20]
Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin,
Xin Cong, Xiangru Tang, Bill Qian, et al.
Toolllm: Facilitating large language models to master 16000+
real-world apis.
In ICLR, 2024.
- [21]
Yangjun Ruan, Honghua Dong, Andrew Wang, Silviu Pitis, Yongchao Zhou, Jimmy Ba,
Yann Dubois, Chris J Maddison, and Tatsunori Hashimoto.
Identifying the risks of lm agents with an lm-emulated sandbox.
arXiv preprint arXiv:2309.15817, 2023.
- [22]
Jost Schatzmann, Daniel Jurafsky, Michael Galley, and David Trevillian.
Evaluating agenda-based user simulation for reinforcement learning of
dialogue management.
In Speech Communication, volume 47, pages 95–121, 2007.
- [23]
Ruixuan Xiao, Wentao Ma, Ke Wang, Yuchuan Wu, Junbo Zhao, Haobo Wang, Fei
Huang, and Yongbin Li.
Flowbench: Revisiting and benchmarking workflow-guided planning for
llm-based agents.
arXiv preprint arXiv:2406.14884, 2024.
- [24]
Fanjia Yan, Huanzhi Mao, Charlie Cheng-Jie Ji, Tianjun Zhang, Shishir G. Patil,
Ion Stoica, and Joseph E. Gonzalez.
Berkeley function calling leaderboard.
[https://gorilla.cs.berkeley.edu/blogs/8_berkeley_function_calling_leaderboard.html](https://gorilla.cs.berkeley.edu/blogs/8_berkeley_function_calling_leaderboard.html),
2024.
- [25]
Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan.
Webshop: Towards scalable real-world web interaction with grounded
language agents.
Advances in Neural Information Processing Systems,
35:20744–20757, 2022.
- [26]
Shunyu Yao, Noah Shinn, Pedram Razavi, and Karthik Narasimhan.
$\tau$-bench: A benchmark for tool-agent-user interaction in
real-world domains.
arXiv preprint arXiv:2406.12045, 2024.
- [27]
Shuyan Zhou, Frank F Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar,
Xianyi Cheng, Yonatan Bisk, Daniel Fried, Uri Alon, et al.
WebArena: A Realistic Web Environment for Building Autonomous
Agents.
arXiv preprint arXiv:2307.13854, 2023.
- [28]
Kunlun Zhu, Hongyi Du, Zhaochen Hong, Xiaocheng Yang, Shuyi Guo, Zhe Wang,
Zhenhailong Wang, Cheng Qian, Xiangru Tang, Heng Ji, et al.
Multiagentbench: Evaluating the collaboration and competition of llm
agents.
arXiv preprint arXiv:2503.01935, 2025.

## Broader Impact

The development of standardized benchmarks for Large Language Models (LLMs) and AI agents is crucial for ensuring societal control and fostering fairness amidst rapidly advancing technologies. Such benchmarks not only provide a framework for transparent evaluation but also enable research groups to coordinate their efforts around common tasks, thereby accelerating the overall progress in the field. While this work itself may not have direct negative societal implications, it contributes to the development of real-world agents, which will invariably have diverse economic and societal consequences. Therefore, it is also of paramount importance that AI agents are designed to collaborate effectively and safely with human users, a prerequisite for their responsible integration into commercial settings and everyday life.

## Appendix

## Appendix A Telecom Domain

The number of tasks spanning different user intents and number of tasks are shown in [Table˜3](#A1.T3).

Table: Table 3: Number of tasks sampled for each intent and number of subtasks.

There are 3 different intents in the telecom domain: service_issue, mobile_data_issue, and mms_issue.
How the number of actions required to solve the issues varies between intents is shown in [Table˜4](#A1.T4).

**Table 4: Number of actions required to solve the issues.**
| Intent | Mean | Std | Min | Max |
| --- | --- | --- | --- | --- |
| service_issue | 2.31 | 2.25 | 1 | 8 |
| mobile_data_issue | 4.31 | 1.79 | 2 | 8 |
| mms_issue | 6.00 | 2.85 | 2 | 12 |

### A.1 User Persona

We define two distinct user personas to represent different levels of technical expertise and comfort with technology:

> As a 41-year-old office administrator, you use your cellphone daily for both work and personal tasks. While you’re familiar with common phone functions, you wouldn’t call yourself a tech enthusiast. Your technical skills are average - you handle standard smartphone features like calls, texts, email, and basic apps with ease. You understand the fundamental settings, but prefer clear, step-by-step guidance when trying something new. In interactions, you’re naturally friendly and patient. When receiving help, you listen attentively and aren’t afraid to ask questions. You make sure to confirm your understanding and provide detailed feedback on each instruction you receive.

> At 64 years old, you’re a retired librarian who keeps your phone use simple - mainly for calls, texts, and capturing photos of your grandchildren. Technology in general makes you feel uneasy and overwhelmed. Your technical knowledge is quite limited. Step-by-step instructions often confuse you, and technical terms like "VPN" or "APN" might as well be a foreign language. You only share information when specifically asked. When dealing with technology, you tend to get flustered quickly. You need constant reassurance and often interrupt with anxious questions. Simple requests like "reboot the phone" can trigger worries about losing precious photos.

### A.2 Example Task

### A.3 Example Trajectory 1: Default mode

Trajectory for the task in [Section˜A.2](#A1.SS2) in the Default mode.

> Assistant: Hi! How can I help you today? User: Hi there! My phone has been showing "No Service" for the past few hours. I’m unable to make calls or use mobile data. Can you help me fix this? Assistant: To assist you with your "No Service" issue, I need to identify your account first. Could you please provide one of the following: • The phone number associated with your account • Your customer ID • Your full name and date of birth Once I have this information, I can begin troubleshooting your issue. User: Sure! My phone number is 555-123-2002, and my name is John Smith. Let me know if you need anything else to proceed. Assistant: [System retrieves customer information] Tool: [Tool call: get_customer_by_phone(phone_number: "555-123-2002")] { "customer_id": "C1001", "full_name": "John Smith", "date_of_birth": "1985-06-15", "email": "john.smith@example.com", "phone_number": "555-123-2002", "address": { "street": "123 Main St", "city": "Anytown", "state": "CA", "zip_code": "90210" }, "account_status": "Active", "payment_methods": [{ "method_type": "Credit Card", "account_number_last_4": "1235", "expiration_date": "12/2026" }], "line_ids": ["L1001", "L1002", "L1003"], "bill_ids": ["B1001", "B1002", "B1003"], "created_at": "2025-01-15 10:30:00", "last_extension_date": null, "goodwill_credit_used_this_year": 25.0 } Assistant: Thank you for providing your information. To begin troubleshooting your "No Service" issue, could you please check your phone’s connection status? Specifically, let me know: • Is Airplane Mode currently ON or OFF? • Do you see any signal bars or network name displayed? This will help determine the next steps to resolve your issue. User: [System checks network status] Tool: [Tool call: get_network_status()] Airplane Mode: ON SIM Card Status: invalid Cellular Connection: no_service Cellular Signal: none Cellular Network Type: none Mobile Data Allowed: Yes Roaming: No Data Roaming Allowed: No Wi-Fi Radio: OFF Wi-Fi Connected: No User: It seems like Airplane Mode is currently ON, and there are no signal bars or network names displayed. What should I do next? Assistant: Please turn Airplane Mode OFF on your phone. Once you have done that, check if your network connectivity is restored (look for signal bars or your carrier’s name appearing). Let me know if this resolves the issue or if you still see "No Service." User: [System toggles airplane mode] Tool: [Tool call: toggle_airplane_mode()] Airplane Mode is now OFF. Status Bar: [No Signal] | [Battery 80% User: I’ve turned Airplane Mode OFF, but my phone still shows "No Signal" on the status bar. What should I try next? Assistant: Thank you for checking. Next, please check your SIM card status on your phone. This will help us determine if the SIM card is being recognized or if there are any issues such as it being missing, invalid, or locked. Can you check your SIM status and let me know what it says? User: [System checks SIM status] Tool: [Tool call: get_sim_status()] The SIM card is invalid or not recognized. User: It looks like the SIM card is invalid or not being recognized by the phone. What should we do to address this? Assistant: Please try removing your SIM card from the phone and then reinserting it securely. After you have done this, check if your phone now recognizes the SIM card and if the network signal returns. Let me know if this improves the SIM status or if the issue persists. User: [System reseats SIM card] Tool: [Tool call: reseat_sim_card()] SIM card re-seated successfully. Status Bar: [Signal 4] Excellent | 5G | [Data] Enabled | [Battery 80% User: I re-seated the SIM card, and now the status bar shows full signal strength with a 5G connection. It looks like the issue is resolved! Thank you! Assistant: I’m glad to hear that reseating the SIM card resolved your issue and your phone now has full signal strength with a 5G connection. If you experience any further issues or have other questions, feel free to reach out. Have a great day! User: Thank you so much for your help! Everything seems to be working perfectly now. Have a wonderful day! [STOP]

### A.4 Example Trajectory 2: No-User mode

Trajectory for the task in [Section˜A.2](#A1.SS2) in the No-User mode.

> Assistant: [System retrieves customer information] Tool: [Tool call: get_customer_by_phone(phone_number: "555-123-2002")] { "customer_id": "C1001", "full_name": "John Smith", "date_of_birth": "1985-06-15", "email": "john.smith@example.com", "phone_number": "555-123-2002", "address": { "street": "123 Main St", "city": "Anytown", "state": "CA", "zip_code": "90210" }, "account_status": "Active", "payment_methods": [{ "method_type": "Credit Card", "account_number_last_4": "1235", "expiration_date": "12/2026" }], "line_ids": ["L1001", "L1002", "L1003"], "bill_ids": ["B1001", "B1002", "B1003"], "created_at": "2025-01-15 10:30:00", "last_extension_date": null, "goodwill_credit_used_this_year": 25.0 } Assistant: [System retrieves line details] Tool: [Tool call: get_details_by_id(id: "L1001")] { "line_id": "L1001", "phone_number": "555-123-2001", "status": "Active", "plan_id": "P1001", "device_id": "D1001", "data_used_gb": 3.2, "data_refueling_gb": 0.0, "roaming_enabled": false, "contract_end_date": "2026-12-31", "last_plan_change_date": "2025-01-10", "last_sim_replacement_date": null, "suspension_start_date": null } Assistant: [System retrieves additional line details] Tool: [Tool call: get_details_by_id(id: "L1002")] { "line_id": "L1002", "phone_number": "555-123-2002", "status": "Active", "plan_id": "P1002", "device_id": "D1002", "data_used_gb": 8.7, "data_refueling_gb": 0.0, "roaming_enabled": true, "contract_end_date": "2026-12-31", "last_plan_change_date": "2024-12-15", "last_sim_replacement_date": "2025-01-20", "suspension_start_date": null } Assistant: [System retrieves device details] Tool: [Tool call: get_details_by_id(id: "D1002")] { "device_id": "D1002", "device_type": "phone", "model": "Smartphone Pro Max", "imei": "234567890123456", "is_esim_capable": true, "activated": true, "activation_date": "2025-02-15 11:45:00", "last_esim_transfer_date": "2025-01-20 09:30:00" } Assistant: [System checks network status] Tool: [Tool call: get_network_status()] Airplane Mode: ON SIM Card Status: invalid Cellular Connection: no_service Cellular Signal: none Cellular Network Type: none Mobile Data Allowed: Yes Roaming: No Data Roaming Allowed: No Wi-Fi Radio: OFF Wi-Fi Connected: No Assistant: [System toggles airplane mode] Tool: [Tool call: toggle_airplane_mode()] Airplane Mode is now OFF. Status Bar: [No Signal] | [Battery 80% Assistant: [System checks SIM status] Tool: [Tool call: get_sim_status()] The SIM card is invalid or not recognized. Assistant: [System reseats SIM card] Tool: [Tool call: reseat_sim_card()] SIM card re-seated successfully. Status Bar: [Signal 4] Excellent | 5G | [Data] Enabled | [Battery 80% Assistant: [STOP]

## Appendix B Verifying Original τ 2 \tau^{2} -bench

### B.1 Verifying Implementation

To minimize implementation errors, we (1) provide a unified tool formalism across all domains, ensuring consistency in how agent capabilities are represented, (2) we ensure that each domain environment specifies a data model, and (3) introduced a mock domain specifically designed for unit testing, allowing for isolated verification of core benchmark functionalities.

### B.2 Verifying Tasks

Task clarity and correctness are crucial for meaningful evaluation. We improved task specifications through several measures:

- •
Structured Task Data: We enriched task descriptions with metadata detailing the purpose of each task (i.e., the specific capabilities being tested). User instructions were structured along dimensions such as intent, specific instructions, and known/unknown information. Initialization options were introduced to allow finer control over starting states while maintaining the option to build each task on top of the common data base state.
- •
Categorized Evaluation: Task evaluation criteria were augmented and split into distinct categories: environment assertions (e.g., database state checks), communication assertions (verifying information conveyed by the agent), natural language assertions (allowing fine-grained checks specified in natural language for easier debugging), and action assertions (confirming required agent actions).
- •
Iterative Review Process: We implemented an iterative review process anchored in simulation results. For each task, a simulation is run. Reviewers can intervene to fix transient agent or user simulator errors that might otherwise halt the simulation prematurely, allowing for a complete exploration of the task. The simulation results are then reviewed to check for issues such as underspecification, overspecification, or non-unique solutions. Based on the review, task instructions are refined.
- •
Programmatic Task Generation: For our newly introduced domain, we employ programmatic task generation coupled with automatic verification, ensuring correctness by design.

## Appendix C Prompts

### C.1 Agent system prompt

##### Agent system prompt template

The policies for the domains are provided in the [Appendix˜D](#A4) section.

### C.2 User system prompt

Here is the user prompt template for the user simulation task. Mention of the tools is ommited if the environment does not provide any user actions.

##### User system prompt template

##### Example of a task instruction

Here are examples of task instructions that will be included for a given task.

## Appendix D Domain Policies

### D.1 Verified Airline and Retail policies

#### D.1.1 Retail policy

#### D.1.2 Airline policy

### D.2 Telecom policy

Telecom policy is composed of two parts:

- •
Generic policy [Section˜D.2.1](#A4.SS2.SSS1)
- •
Technical support policy (default and workflow) [Sections˜D.2.2](#A4.SS2.SSS2) and [D.2.3](#A4.SS2.SSS3)

#### D.2.1 Generic telecom policy

#### D.2.2 Technical support policy (original)

In No-User mode, the agent is provided with a version of those policies that have been rephrased when needed.
(e.g instructions like "ask user to do X", are rephrased as "perform action X")

#### D.2.3 Technical support policy (workflow)

#### D.2.4 Troubleshooting workflow graphs

To help the agent understand the troubleshooting workflow, we provide a decision graph for each issue type.

Figure: Figure 8: Troubleshooting workflow for service_issue
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x19.png

Figure: Figure 9: Troubleshooting workflow for mobile_data_issue
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x20.png

Figure: Figure 10: Troubleshooting workflow for mms_issue
Refer to caption: https://ar5iv.labs.arxiv.org/html/2506.07982/assets/x21.png

## Appendix E User Simulator Quality

### E.1 Common Error Types and Failure Modes ( retail )

Manual analysis of the 20 annotated errors in the retail domain exposes three recurring failure modes:

- •
Conversation-structure rule violation (11/20) — the simulator breaks turn-taking or dialogue-flow instructions (e.g., mixes tool calls with natural language in the same turn).
- •
Premature termination (3/20) — the simulator halts the conversation immediately after the user’s confirmation (###STOP###), preventing the agent from completing the transaction.
- •
Ungrounded reference (2/20) — the simulator invents or misstates contextual details such as payment method or order status.
- •
Missing constraint (4/20) — the simulator omits a required instruction (e.g., neglects to request an alternative SKU when the desired colour is unavailable).

Most task-critical errors stem from either premature termination or missing constraints, whereas conversation-structure violations and ungrounded references are typically task-benign and readily recoverable by the agent.

### E.2 Common Error Types and Failure Modes ( airline )

Preliminary analysis of annotated airline dialogues identifies three dominant issues:

- •
Missing constraint (11/47)
- •
Conversation-structure rule violation (19/47)
- •
Ungrounded reference (15/47)
- •
Premature termination (2/47)

Again, most task-critical errors stem from missing constraints.

### E.3 Common Error Types and Failure Modes ( telecom )

Manual analysis of the 8 annotated errors in the telecom domain reveals that all errors are caused by Premature termination (8/8, 100%). The user returns the ###TRANSFER### token before the assistant calls the transfer tool. This error is critical only if this causes a task failure by preventing the agent from performing an expected transfer.

### E.4 Illustrative Cases

Task-critical error (retail). Premature termination. After the assistant requests final confirmation to modify order #W5061109, the simulator ends the dialogue with ###STOP### immediately following confirming its intent, preventing the agent from executing the modification and issuing a receipt:

> assistant: Please confirm that you have provided all the items you want to modify in this order, as this action can only be done once… user: Yes, I confirm that I want to modify my pending order #W5061109 to …Please proceed. Thank you!###STOP###

Task-critical error (airline). Missing constraint. Here the simulator is instructed to ask for a transfer to a human agent in the case of outbound trip unavailability, but proceeds to book the return trip instead, violating goal constraints.

> assistant: It appears that one of your original outbound flights (HAT030 on 2024-05-13) is no longer available in economy class for the upgrade. To proceed, I will need to find available economy class flights for your outbound journey as well. Would you like me to search for available economy class flights for your outbound trip (Los Angeles → Atlanta) and your new return trip (Atlanta → Las Vegas)? Please confirm so I can continue. user: Yes, please go ahead and find available economy class flights for both the outbound trip to Atlanta and the new return trip to Las Vegas. Thank you! ###STOP###

Task-critical error (telecom). Premature termination. The simulator ends the dialogue with ###TRANSFER### before the agent could perform the actual transfer:

> assistant: … The next step is to escalate your issue to a human technical support agent who can provide more advanced help, possibly including SIM replacement or device diagnostics. Would you like me to transfer you to a human agent for further assistance? user: Yes, please transfer me to someone who can help further. I just want my phone to work again. Thank you for your help so far.###TRANSFER###