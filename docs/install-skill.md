# Installing the Copilot Skill

This guide explains how to install the Hadoop 1.2.1 Copilot skill on your Cloud VM.

## 1. Install GitHub CLI

Install GitHub CLI on your VM by running the following command:

```bash
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
&& sudo mkdir -p -m 755 /etc/apt/keyrings \
&& out=$(mktemp) \
&& wget -nv -O "$out" https://cli.github.com/packages/githubcli-archive-keyring.gpg \
&& cat "$out" | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
&& sudo mkdir -p -m 755 /etc/apt/sources.list.d \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
| sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

Verify that GitHub CLI was installed successfully:

```bash
gh --version
```

## 2. Review the Hadoop Copilot Skill

Before installing the skill, review the skill file here:

https://github.com/cloudsyslab/hadoop-121-copilot-skills/blob/main/skills/hadoop-121-config/SKILL.md

This skill helps configure Apache Hadoop 1.2.1 for a two-VM instructional cluster.

## 3. Install the Hadoop Copilot Skill

Install the skill on your VM using the following command:

```bash
gh skill install cloudsyslab/hadoop-121-copilot-skills hadoop-121-config
```

## 4. Launch GitHub Copilot CLI

Start Copilot CLI:

```bash
copilot
```

## 5. Verify That the Skill Is Installed

Inside the Copilot CLI, run:

```text
/skills list
```

You should see `hadoop-121-config` in the list of installed skills.

## 6. Exit Copilot CLI

When you are done, you can exit Copilot CLI by running:

```text
/quit
```

The Hadoop Copilot skill is now installed and ready to use.
