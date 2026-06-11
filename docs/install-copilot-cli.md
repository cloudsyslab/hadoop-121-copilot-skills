# GitHub Copilot Installation and Setup

This guide explains how to install and set up GitHub Copilot CLI on a Cloud VM.

## Prerequisites

Before starting, make sure you have completed the following:

* Create a GitHub account using your UTSA email address, if you do not already have one.
* Sign up for the GitHub Student Developer Pack to access GitHub Copilot Pro for free:
  https://education.github.com/pack

## 1. Log in to your Cloud VM

Log in to your Cloud VM using SSH.

Then install the required packages:

```bash
sudo apt update
sudo apt install -y curl ca-certificates gnupg
```

## 2. Add the NodeSource repository for Node.js 24

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
```

## 3. Install Node.js 24

```bash
sudo apt install -y nodejs
```

## 4. Verify the Node.js and npm installation

```bash
node -v
npm -v
```

You should see version numbers printed for both commands.

## 5. Install GitHub Copilot CLI

```bash
sudo npm install -g @github/copilot
```

## 6. Start GitHub Copilot CLI

```bash
copilot
```

## 7. Log in to your GitHub Copilot account

Inside the Copilot CLI, run:

```text
/login
```

Copilot will provide a one-time device code.

Open the following page in your web browser:

https://github.com/login/device

Enter the one-time code and authorize the device.

## 8. Select the AI model

Inside the Copilot CLI, run:

```text
/model
```

Select the model you want to use. The **Auto** option is recommended.

## 9. Exit GitHub Copilot CLI

Inside the Copilot CLI, run:

```text
/quit
```

GitHub Copilot CLI is now installed and ready to use.
