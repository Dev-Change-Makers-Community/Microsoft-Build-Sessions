Log into the Github Account

![Github Login](images/github_login.png)
![Github Repo](images/github_login1.png)
Go to the [private workspace]("https://github.com/Skillable-Events/build26-LAB540") created for you
![Github Repo](images/github_workspace.png)


#### Go to Microsoft Foundry and check models

![Microsoft Foundry](images/foundry_build.png)


# Open the Codespace

Select the Codespace you want to open

![Codespace](images/codespace.png)

Learning Objectives

In this hands-on lab, we'll see how the Microsoft Foundry Observability platform works with GitHub Copilot and Foundry skills, to simplify the developer experience and accelerate their progress from plan to prototype.

By completing this lab you will learn to:
- Create and deploy a hosted agent using Azure Developer CLI
- Auto-generate test datasets and evaluators using the observe skill
Activate the evaluate-optimize loop to iteratively improve agent
Retrieve and analyze production insights to troubleshoot failures
- Explore new features like adaptive evaluations & optimization service


# Follow core folder for the lab instruction

![Core Folder](images/core.png)

### Lab 0
Now ask copilot to

```
"Run Lab 0"
```

![Run Lab 0](images/f1.png)
Follow the instructions to run the lab 0.
![Run Lab 0](images/f2.png)

```
az login --use-device-code
az account show --query '{name:name, id:id}' -o table
azd auth login --use-device-code
```
To populate the .env

```
chmod +x scripts/discover-env.sh
./scripts/discover-env.sh
```

Add missing values from Microsoft Foundry (Operate -> Admin)

![Microsoft Foundry](images/env_populate.png)

You can also verify the resource group and the foundry account in the Azure portal.

![Azure Portal](images/verify.png)

### Lab 1

Now write 

```
Complete Lab 1
```
![Run Lab 1](images/lab1_a.png)


Failure Note: The Zava Travel Concierge is not pre-deployed in the Foundry portal.
