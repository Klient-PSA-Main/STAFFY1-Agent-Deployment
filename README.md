# STAFFY1 — Resource Management Employee Agent

<a href="https://githubsfdeploy.herokuapp.com?owner=Klient-PSA-Main&repo=STAFFY1-Agent-Deployment&ref=main">
  <img alt="Deploy to Salesforce"
       src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/deploy.png">
</a>

STAFFY1 is an Agentforce Employee Agent built with legacy Bot metadata (no Agent Script / `AiAuthoringBundle`).
It reads live capacity, surfaces open roles on In Progress projects, proposes best-fit resources with reasoning,
raises conflict alerts, and creates task assignments only after explicit human approval.

## What gets deployed

| Component | Type | Path |
|---|---|---|
| `STAFFY1` | Bot (`AgentforceEmployeeAgent`) | `force-app/main/default/bots/STAFFY1/STAFFY1.bot-meta.xml` |
| `STAFFY1.v1` | BotVersion | `force-app/main/default/bots/STAFFY1/v1.botVersion-meta.xml` |
| `STAFFY1` | GenAiPlannerBundle | `force-app/main/default/genAiPlannerBundles/STAFFY1/STAFFY1.genAiPlannerBundle` |

The planner bundle wires in the 14 STAFFY topics that ship in the STAFFY1 managed package (namespace `KPSA004`):

`Staffy_Agent_Instructions`, `Staffy_Approve`, `Staffy_Assignments`, `Staffy_Capacity`, `Staffy_Conflicts`,
`Staffy_EntityDetail`, `Staffy_ForecastTopic`, `Staffy_Help`, `Staffy_Match`, `Staffy_OpenRoles`,
`Staffy_ResourceRequests`, `Staffy_Review`, `Staffy_Scheduling`, `Staffy_Updates`

This repo holds only the agent shell. The topics, actions, and Apex come from the package.

## Prerequisites

- An org with Einstein generative AI and Agentforce turned on (Setup > Agentforce Agents).
- The STAFFY1 managed package (namespace `KPSA004`) installed. Without it the deploy can still succeed,
  but the agent has no topics to route to.
- If the org already has a Bot named `STAFFY1`, deploying updates it rather than creating a new one.

## Deploy

**Button:** click **Deploy to Salesforce** above, log in to the target org, and confirm the deploy.

**CLI:**

```bash
sf project deploy start --target-org <alias> \
  --source-dir force-app/main/default/bots/STAFFY1 \
  --source-dir force-app/main/default/genAiPlannerBundles/STAFFY1 --wait 30
```

## After deploying

Check the agent landed and whether it is active:

```bash
sf data query --target-org <alias> --query \
  "SELECT BotDefinition.DeveloperName, DeveloperName, Status FROM BotVersion WHERE BotDefinition.DeveloperName = 'STAFFY1'"
```

1. Open Setup > Agentforce Agents > STAFFY1 and confirm all 14 topics are listed.
2. If `Status` is `Inactive`, activate it there.
3. Assign the package's **STAFFY1 Agent Access** permission set to the users who will chat with the agent.

Keep the API name `STAFFY1`: the Agent Instructions topic loads its admin instructions by that developer name.
