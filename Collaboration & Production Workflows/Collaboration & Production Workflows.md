# How GIT Works in Production 

---

## Table of Contents

* [Introduction](#introduction)  
* [Why Remote](#why-remote)  
* [What are Remote Repositories](#what-are-remote-repositories)  
* [Types of Remote Repositories](#types-of-remote-repositories)  
* [Authenticating with Remote](#authenticating-with-remote)  
* [Why Authentication Is Required](#why-authentication-is-required)  
* [Authentication Methods for Git CLI](#authentication-methods-for-git-cli)  
* [How SSH and Tokens Are Used in Practice](#how-ssh-and-tokens-are-used-in-practice)  
  * [Practical Decision Guide](#practical-decision-guide)
* [**Demo:** Working with Remote Repository](#demo-working-with-remote-repository)  
  * [Step 1: Generate SSH Keys & Connect to GitHub](#step-1-generate-ssh-keys--connect-to-github)  
  * [Step 2: Create a Remote Repository](#step-2-create-a-remote-repository)  
  * [Step 3: Create Local Repository & Commits](#step-3-create-local-repository--commits)  
  * [Step 4: Add Remote](#step-4-add-remote)  
  * [Step 5: Push Local Commits to Remote](#step-5-push-local-commits-to-remote)  
  * [Setting Up Upstream (Making git push Work Seamlessly)](#setting-up-upstream-making-git-push-work-seamlessly)  
  * [Step 6: Cleanup (Local and Remote Branches)](#step-6-cleanup-local-and-remote-branches)  
* [Understanding `git fetch` and `git pull`](#understanding-git-fetch-and-git-pull)  
    * [Step 7: Simulate Changes from Another Developer](#step-7-simulate-changes-from-another-developer)  
    * [Step 8: Observe Mismatch and Sync with Remote](#step-8-observe-mismatch-and-sync-with-remote)  
* [Understanding `git diff`](#understanding-git-diff)  
  * [**Mini Demo:** `git diff`](#mini-demo-git-diff)  
* [Pull Requests and Code Review](#pull-requests-and-code-review)  
  * [Step 9: Protect the `main` Branch Using Rulesets (Production Setup)](#step-9-protect-the-main-branch-using-rulesets-production-setup)  
  * [Step 10: Create a Branch, Make Changes, and Push](#step-10-create-a-branch-make-changes-and-push)  
  * [Step 11: Create and Approve a Pull Request](#step-11-create-and-approve-a-pull-request)  
  * [Step 12: Sync Local Repository After Merge](#step-12-sync-local-repository-after-merge)  
  * [Step 13: Attempt Direct Push to Protected Branch](#step-13-attempt-direct-push-to-protected-branch)  
  * [Step 14: Cleanup (Local and Remote)](#step-14-cleanup-local-and-remote)  
* [Git Stash](#git-stash)  
  * [**Demo:** Git Stash](#demo-git-stash)  
* [Conclusion](#conclusion)  
* [References](#references)
  
---

## Introduction

In Collaboration & Production Workflows, we built a strong foundation of Git by understanding how it works **locally**. We explored commits, branching, merging, conflict resolution, and even how to undo mistakes. At that stage, Git helped us manage change within a **single machine**.

But real-world systems are not built in isolation.

In production environments, multiple developers work on the same codebase simultaneously. Changes are not just created, they are **shared, reviewed, validated, and integrated** into a common system. This introduces new challenges: synchronization, access control, collaboration, and maintaining a stable system while change is continuously happening.

This is where Git evolves from a local version control tool into a **collaborative system**.

In this part, we move from:
- managing history locally  
to  
- managing change across **people, machines, and systems**

We will cover:
- how remote repositories enable collaboration  
- how authentication secures access  
- how `push`, `fetch`, and `pull` synchronize systems  
- how Pull Requests introduce controlled change management  
- how branch protection enforces production-grade workflows  
- and how `git stash` helps safely handle context switching  

By the end of this part, you will understand not just how Git works, but **how Git is used in production systems to maintain stability at scale**.

---

## Why Remote?

In **GitHub Foundation**, everything we did was limited to our **local environment**. We created commits, worked with branches, merged changes, resolved conflicts, and even learned how to undo mistakes. All of this happened entirely within our **local repository**.

This leads to one very important realization: **all our work existed only on our machine**. There was no mechanism through which our code could be shared with others, others could contribute their changes, or a common version of the project could be maintained.

---

### The Real-World Problem

Now think about how software is actually built in the real world. You wrote code locally and it works. Your teammate also wrote code locally and it works. But both of you worked independently, and eventually those changes need to be combined into a single system. This is where problems begin in real teams.

Without a shared system:

* **changes get overwritten**
  One person’s updates can unintentionally replace another’s work because there is no structured way to merge changes safely.

* **work gets duplicated**
  Multiple engineers may end up solving the same problem independently, simply because they are unaware of each other’s work.

* **integration becomes painful**
  Combining changes manually becomes complex and error-prone, especially as the codebase and team size grow.

* **there is no single source of truth**
  Different team members may have different versions of the system, leading to confusion about what is actually correct or deployable.

---

### The Shift from Local to Collaborative Systems

Up until now, Git helped us manage change **within a single machine**. But modern software development is not an individual activity, it is a **collaborative system** where multiple engineers contribute simultaneously.

Git is therefore not just about commits and branches. It is about **managing and synchronizing change across people and systems**.

To enable this, Git introduces the concept of a **remote repository**, which acts as a shared system where all collaborators can push their work, pull updates, and stay in sync.

---

## What are Remote Repositories

Now that we understand **why remote repositories are required**, let us build on that foundation.

In Part 1, we learned that a Git repository is simply a directory managed by Git, containing both project files and their complete history. So far, all such repositories existed on our local machine, and these are referred to as **local repositories**.

When the same concept is extended to a repository hosted on a remote system such as a server, it is called a **remote repository**. From a Git perspective, nothing fundamentally changes in how history is stored or how commits behave. What changes is **where the repository lives and how it is accessed**.


> **Note:** In real-world conversations, people rarely say **“local repository”** or **“remote repository”**. They simply say **“repository”** or **“repo”**. The meaning is inferred from context because both are always used together in actual workflows.

---

### Role of a Remote Repository

A remote repository acts as a **shared point of synchronization** between multiple contributors. Instead of each developer working in isolation, everyone interacts with a common system where changes are exchanged and integrated.

This enables developers to:

* **push their changes**
  Send local commits to the remote repository so others can access and build upon them.

* **pull updates from others**
  Bring changes made by teammates into their own local repository to stay up to date.

* **collaborate on the same codebase**
  Work on different parts of the system while maintaining a consistent and unified history.

In addition to collaboration, remote repositories also provide several operational benefits:

* **centralized visibility**
  The current state of the project is visible in one place, making it easier to understand progress and history.

* **integration with CI/CD systems**
  Changes pushed to the remote can automatically trigger pipelines for build, test, and deployment.

* **access control and auditing**
  Organizations can control who can read or modify code and track who made specific changes.

* **backup of project history**
  Since the repository exists on a remote system, it acts as a reliable backup beyond individual machines.

---

### Choosing a Remote Platform

In this masterclass, we will use **GitHub** as our remote repository provider. This is not because it is the only option, but because it is the most widely adopted platform in the industry and aligns well with common production workflows.

Other commonly used providers include:

* GitLab
* Bitbucket
* AWS CodeCommit
* Azure Repos

In real-world environments, the choice of platform is usually influenced by:

* **organizational ecosystem and integrations**
  Teams often prefer platforms that integrate well with their existing cloud, CI/CD, and security tooling.

* **security and compliance requirements**
  Some organizations must adhere to strict policies that influence where code can be hosted.


> **Note:** The choice of remote repository provider is often influenced by an organization’s ecosystem and strategy. For example, a company heavily invested in **AWS** may prefer **CodeCommit** for tighter integration, while others may standardize on **GitHub Enterprise** or **GitLab** for their broader ecosystem.
> In **multi-cloud or cloud-agnostic environments**, teams may deliberately choose platforms like **GitHub** or **GitLab** to avoid tight coupling with a single cloud provider.

---

## Types of Remote Repositories

Remote repositories are broadly classified based on **access control**, that is, who is allowed to view and modify the code.

There are two primary types of remote repositories:

1. **Private repositories**
2. **Public repositories**

The distinction between them is not in how Git works internally, but in **who is allowed to access and interact with the repository**.

---

### 1. Private Repositories

A private repository is one where access is **restricted to authorized users**. Only users who have been explicitly granted permissions can view the repository, clone it, or push changes.

This type of repository is commonly used for:

* **Enterprise applications**
  Business-critical systems that contain proprietary logic and must not be exposed publicly.

* **Internal tools**
  Utilities and platforms used within an organization for internal operations.

* **Proprietary source code**
  Code that represents intellectual property and competitive advantage.

* **Infrastructure configurations**
  Terraform, Kubernetes manifests, and other configurations that may contain sensitive details about system architecture.

For example, backend services of a company, production Kubernetes manifests, or cloud infrastructure definitions are almost always stored in private repositories because exposing them publicly would introduce both **security and business risks**.

---

### 2. Public Repositories

A public repository is accessible to **anyone on the internet**. Anyone can view the code and clone the repository without any special permissions.

However, this does not mean that anyone can modify it. Write access is still controlled, and only authorized contributors can push changes.

Public repositories are typically used for:

* **open source projects**
  Software developed collaboratively by the community.

* **community-driven tools**
  Libraries and frameworks that evolve through contributions from multiple developers.

* **learning and educational content**
  Repositories created for teaching, tutorials, or demonstrations.

* **personal portfolios**
  Projects that developers showcase to demonstrate their skills.

Examples include widely used projects such as Kubernetes, Terraform, and many open source libraries.

> **Note:** All courses I create, such as **CI/CD**, **CKA**, and **ArgoCD**, including this **Git Masterclass**, have their own **public repositories**. You are free to **clone and use them for learning**, but you cannot modify them unless you have explicit write access.


> The key distinction between private and public repositories is not in how Git works internally, but in **who is allowed to access and interact with the repository**.

---

## Authenticating with Remote

Authentication with a remote repository happens in **two primary contexts**, depending on how you are interacting with the system.

### 1. Authentication via GUI (Browser Access)

This is when you log in to platforms like GitHub through a web browser. It allows you to view repositories, manage settings, and perform administrative actions. This is user-level authentication handled by the platform UI.

---

### 2. Authentication via Git CLI (Machine Access)

This is when your local machine interacts directly with the remote repository using Git commands like `push` and `pull`. Here, authentication ensures that your **machine or process is authorized** to perform actions on the repository.

Unlike the browser, there is no interactive login flow. Your **machine directly communicates with the remote system**, and identity must be verified using credentials such as **SSH keys or tokens**.

> The distinction between GUI and CLI authentication is not in what Git does, but in **how identity and access are verified depending on the mode of interaction**.

> **Note:** Even in GUI (browser-based) login, the platform ultimately uses **tokens or session credentials under the hood** (e.g., cookies, OAuth tokens) to authenticate subsequent requests.
>
> The distinction here is not the underlying mechanism, but the **mode of interaction**:
>
> * **GUI** → Browser logins use **interactive flows (SSO, MFA)** and then rely on **session tokens internally**
> * **CLI** → Uses **pre-configured credentials (SSH keys or tokens)** for direct, non-interactive access
>
> In both cases, the goal is the same: **verify identity and enforce access control**, but the way credentials are presented and managed differs.

---

## Why Authentication Is Required

When working through the browser, authentication is straightforward. You log in, and the platform knows who you are.

However, when you switch to the **Git CLI**, the interaction model changes. Your **local machine directly communicates with a remote server** whenever you run commands like `git push` or `git pull`.

At that point, the remote system must verify:

> “Is this user or machine allowed to perform this action on this repository?”

This verification is what we call **authentication**.

This applies to all repositories:

* **private repositories**
  Both read and write access are restricted to authorized users.

* **public repositories**
  Anyone can read (clone), but only authorized users can write (push).

So while read access may vary, **all write operations to a remote repository require authentication**.

> **Note:** Username and password-based authentication is deprecated due to security limitations. Passwords are easily reused or leaked, cannot be scoped to specific actions, and lack proper auditability. Modern systems instead use **SSH keys and tokens** for secure, controlled access.

---

## Authentication Methods for Git CLI

When interacting with a remote repository via the **Git CLI**, authentication is performed using one of two primary mechanisms. These are not separate from the CLI, but the **underlying protocols used by Git to securely communicate with remote systems**.

---

### 1. SSH (Typically for Humans)

SSH is based on a **public-private key pair**, where your machine is registered with the remote system using a public key, and authentication is performed using the corresponding private key.

* **Authentication model:** Asymmetric key-based challenge-response authentication, where the server verifies possession of the private key without it ever being transmitted over the network
* **User type:** Developers and DevOps engineers working interactively, typically mapped to individual user identities on platforms like GitHub
* **Developer experience:** Seamless, no repeated prompts due to SSH agent caching; supports high-frequency operations like push, pull, fetch without re-authentication overhead
* **Security:** Strong cryptographic guarantees with no shared secrets; resistant to phishing, credential replay, and man-in-the-middle attacks when host verification is enforced
* **Access model:** Keys are associated with a user account, not just a machine; access is implicitly tied to repository permissions granted to that user
* **Agent support:** SSH agents (`ssh-agent`) securely hold decrypted keys in memory, enabling reuse without exposing private keys or repeatedly entering passphrases
* **Enterprise controls:** Keys can be centrally audited, rotated, and revoked; organizations often enforce policies around key types, expiration, and usage

**Best Practices (Industry):**

* Use **passphrase-protected SSH keys** to protect against local key theft
* Prefer modern algorithms (e.g., `ed25519` over RSA) for stronger security and performance
* Avoid sharing keys across users or systems to maintain accountability
* Regularly rotate and remove unused keys to reduce attack surface
* Enforce **host key verification** to prevent MITM attacks
* Use hardware-backed keys (e.g., security tokens) in high-security environments

---

### 2. Tokens (Typically for Systems)

Tokens are generated credentials used in place of passwords, designed for **controlled, auditable, and programmatic access** over HTTPS.

* **Authentication model:** Bearer token-based authentication where each request includes a token, and possession of the token is sufficient for access (no additional proof required)
* **User type:** CI/CD pipelines, automation tools, scripts, integrations, and non-interactive systems; may represent either a user or a service identity
* **Access control:** Fine-grained and explicitly scoped permissions (repository-level, organization-level, or action-specific such as read, write, admin)
* **Security:** Can be short-lived, rotated, and revoked; reduces reliance on long-lived static credentials and limits blast radius on compromise
* **Auditability:** Every action performed using a token is logged and can be traced back to the issuing identity or system, enabling compliance and monitoring
* **Transport:** Works over HTTPS (port 443), making it compatible with restricted networks, proxies, and enterprise firewall policies

**Types of Tokens (Common in Industry):**

* Personal Access Tokens (PAT)
* Fine-grained tokens (repo-scoped with explicit permissions)
* Service account tokens (non-human identities for automation)
* Short-lived tokens via OIDC federation (modern best practice, issued dynamically at runtime)

**Best Practices (Industry):**

* Prefer **short-lived and dynamically generated tokens** over long-lived static credentials
* Follow **least privilege principle** by granting only the minimum required permissions
* Never hardcode tokens in code, scripts, or repositories
* Store in secure systems (secrets managers, vaults, CI/CD secret stores)
* Rotate frequently and monitor usage for anomalies
* Mask tokens in logs and outputs to prevent accidental exposure

---

### How SSH and Tokens Are Used in Practice

In real-world environments, the choice between SSH and tokens is not fixed. It depends on **organizational policies, scale, and security requirements**.

In real-world environments, the choice between SSH and tokens is influenced less by developer preference and more by **organizational security policies, identity integration, and scalability requirements**.

* Developers may use **HTTPS with tokens** instead of SSH in environments where authentication is integrated with **central identity providers (IdPs)**. In such setups (common in GitHub and GitLab), access is governed by **SSO, MFA, and conditional access policies**, making token-based authentication easier to enforce, audit, and centrally control than SSH keys.
* While CI/CD systems can use SSH via **deploy keys**, this approach has both advantages and limitations:

  **Advantages:**

  * Scoped to a single repository, reducing blast radius if compromised
  * Simple to configure for isolated automation use cases

  **Limitations:**

  * Limited to repository-level access (no fine-grained, action-level permissions)
  * **Not tied to a user** or centralized identity, making auditing and ownership less clear
  * Do not integrate naturally with centralized identity providers (IdPs)
  * Often lack strong protection mechanisms (e.g., passphrase usage is uncommon in automation)
  * Difficult to manage, rotate, and scale across multiple repositories and pipelines

  > **Note:** Platforms like GitHub recommend using **GitHub Apps** instead of deploy keys for improved security and control. GitHub Apps provide:
  >
  > * **Fine-grained, scoped permissions**
  > * **Better security and auditability** (short-lived tokens, traceable actions)
  > * **Stronger integration with modern identity models** (app identity, OIDC, least privilege)
* As a result, organizations increasingly prefer **token-based access models**, especially those that support **fine-grained permissions, centralized control, and auditability**.
* Modern systems are evolving toward **ephemeral credentials**, where tokens are:

  * **Short-lived and dynamically issued** (e.g., via OIDC federation)
  * Not stored as long-lived secrets in systems or pipelines
  * Automatically scoped and tied to workload or pipeline identity
* This shift reduces risks associated with **static credentials** (such as key leakage or token exposure) and aligns with **zero-trust principles**, where access is continuously verified and minimally scoped.

> **Note:** Earlier we said SSH keys are typically used by humans and tokens by CI/CD systems.
> This is a general pattern, not a strict rule.
>
> Platforms like GitHub support **deploy keys (SSH-based)**, which can be used by automation. However, they are **repository-scoped, not tied to a user or central identity**, and become difficult to manage and audit at scale.
>
> Because of these limitations, GitHub **generally recommends using GitHub Apps or token-based approaches** instead of deploy keys for most production and CI/CD use cases.
>
> In practice, modern systems prefer **token-based access (PATs, GitHub Apps, OIDC)** because they support **fine-grained permissions, better auditability, and integration with enterprise identity systems (SSO, IdP)**.
>
> So the choice is not about capability, but about **scale, control, and security requirements**. This is why enterprises predominantly standardize on tokens.

---

### Practical Decision Guide

#### 1. **Human Access (Developers/DevOps)**

* **Local CLI usage (individual developers)**
  * Use **SSH keys** for seamless, password-less authentication
  * Authentication is based on your **private key proving identity** (no login flow)
  * Actions are mapped directly to your **user account on the platform**
* **Enterprise environments (SSO, MFA, policies enforced)**
  * Use **HTTPS + tokens** integrated with **IdP (Okta, Azure AD, etc.)**
  * First authenticate via **SSO (interactive login + MFA)**
  * Git operations then use a **token derived from that session**
  * Token represents your **user identity**
  * Enables:
    * **User-level auditing**
    * **Centralized access control**
    * **Policy enforcement (MFA, device policies, conditional access)**

---

#### 2. **Non-Human Access (CI/CD, Automation Systems)**

* **Pipelines, scripts, and automation workflows**

  * Use **tokens (GitHub Apps / OIDC / short-lived tokens)**
  * Authentication is **non-interactive** (no human login possible)
  * Token represents a **service or workload identity**, not a person
  * Typically:
    * Issued **at runtime**
    * Used for a short duration
    * Automatically expires
  * Enables:
    * **Fine-grained permissions**
    * **Scalability across systems**
    * **Clear audit trails tied to the workload**

> Both humans and systems use tokens, but the difference is **whose identity the token represents**.

---

#### **Important Practices**

* **Avoid SSH deploy keys for automation at scale**
  * Repository-scoped access only
  * **Not tied to a user or centralized identity**
  * Difficult to **audit, rotate, and manage** across systems
  * Suitable only for **simple, isolated use cases**
* **Prefer short-lived credentials wherever possible**
  * Avoid long-lived **SSH keys and PATs** where feasible
  * Use **ephemeral tokens (OIDC, GitHub Apps)**
  * Benefits:
    * Reduced blast radius if compromised
    * Automatic rotation
    * Alignment with **zero-trust and least-privilege models**

---

## Demo: Working with Remote Repository

In this demo, we will connect our **local Git repository** with a **remote repository on GitHub**, authenticate our machine, and push our local commits to the remote system. This is the first time we move from **local-only workflows to collaborative workflows**.

---

## Step 1: Generate SSH Keys & Connect to GitHub

> **Reference:** For a complete step-by-step guide, refer to the official GitHub documentation:
> [https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=mac](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=mac)

Before our local machine can push code to GitHub, we must establish **trust between our machine and GitHub**. This trust ensures that when our machine attempts to perform operations like `git push` or `git pull`, GitHub can verify that the request is coming from an authorized source.

This trust is established using **SSH keys**.

---

### Generate SSH Key

Run:

~~~bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/Our GitHub account name
~~~

This command generates a **public-private key pair**.

* The **private key** remains on your machine and must never be shared.
* The **public key** is added to GitHub, allowing GitHub to recognize your machine.

Let us understand the flags:

* **`-t ed25519`**
  Specifies the type of key to generate. Ed25519 is a modern, secure, and recommended algorithm.

* **`-C`**
  Adds a comment to the key. This is simply metadata used for identification and is not limited to email addresses. In practice, developers use their email so platforms like GitHub can associate the key with their account.

* **`-f`**
  Specifies the file name and location of the key. This becomes especially important when managing multiple keys.

> **Note:** The value passed with `-C` does not play any role in authentication. It is only used for identification.

---

### Add Key to SSH Agent

In the previous step, we generated an SSH key, which is stored as a **private key file on our system**.
However, SSH does not always know **which key to use** for authentication with a remote like GitHub, especially when multiple private keys exist on the same machine.

By default, SSH can:

* use keys from the **SSH agent (memory)**
* fall back to **default identity files on disk** (e.g., `~/.ssh/id_ed25519`, `id_rsa`)

This creates two practical challenges:

* SSH may try **multiple keys sequentially**, leading to ambiguity
* Passphrase-protected keys would require **repeated manual entry**

---

To solve this, SSH uses a background process called the **SSH agent**, which loads private keys into memory and makes them available to SSH clients during authentication.

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/Our GitHub account name
```

* **`ssh-agent`** → Starts a background process that manages keys
* **`ssh-add`** → Loads your private key into the agent for use during authentication

---

#### Verify Loaded Keys

```bash
ssh-add -l
```

* Lists all keys currently loaded in the SSH agent
* Confirms whether your key is **available for authentication**

Example output:

```text
256 SHA256:abc123... ~/.ssh/Our GitHub account name (ED25519)
```

If no keys are loaded:

```text
The agent has no identities.
```

→ This means your key is **not currently in memory**, and you need to run `ssh-add` again

---

### What Actually Happens During Authentication

When you run:

```bash
git push
```

Git establishes an SSH connection to:

```bash
git@github.com
```

During this process:

1. SSH first checks if an **SSH agent** is running
2. If present, it tries **keys loaded in the agent**
3. If needed, it may also try **default identity files from disk**
4. The server checks if any presented key matches a registered user
5. If a match is found, authentication succeeds

> **Important:** SSH does **not send the private key**. It uses it to **sign a challenge**, proving identity.

---

Without the SSH agent:

* authentication can still work
* but SSH may:

  * prompt repeatedly for passphrase
  * try multiple keys sequentially
  * hit server-side limits (too many failed attempts)

---

**Remove Key from SSH Agent (if needed)**

```bash
# Option 1: Remove specific key
ssh-add -d ~/.ssh/Our GitHub account name

# Option 2: Remove all loaded keys
ssh-add -D
```

---

### Managing SSH Keys in Real-World Setups

At this point, your SSH setup works correctly for basic usage.
However, as soon as you move beyond a single repository or a single account, two common problems begin to surface.

---

#### 1. Keys are not persistent

The SSH agent stores keys **only in memory**, not on disk.

This means:

* when your system restarts, the agent process is reset
* all previously loaded keys are removed from memory
* your private key still exists on disk (`~/.ssh/...`), but is no longer actively used

As a result, even if your repository was working earlier, authentication may fail after a reboot until you manually run:

```bash
ssh-add ~/.ssh/Our GitHub account name
```

---

### 2. Multiple keys create ambiguity

In real-world environments, it is common to have **multiple SSH keys**, for example:

* separate keys for personal and work GitHub accounts
* different keys across machines
* project-specific or organization-specific keys

A common pattern is:

> **one identity (account) → one SSH key**

---

Now consider:

```bash
git@github.com:Our GitHub account name/Our GitHub account name-git-practice.git
git@github.com:acme-corp/payment-service.git
```

If both accounts are configured on the same machine, SSH must decide:

> **Which private key should be used for this connection?**

---

By default, SSH behaves as follows:

> It tries keys from the SSH agent first, and may also try default identity files from disk until a match is found

---

#### Why this becomes a problem

Even though SSH tries keys sequentially, this introduces real issues:

* authentication becomes slower (multiple attempts)
* order of keys affects behavior (agent order + default identities)
* servers may reject connections after too many failed attempts
* debugging becomes difficult in multi-account setups

---

What we want instead is:

> **deterministic key selection** → a specific key is always used for a specific connection

---

> **Note:**
> By default, SSH may try multiple keys (agent + default files), which can cause ambiguity.
> Using `IdentitiesOnly yes` ensures only explicitly configured keys are used.

---

#### Solution: SSH Configuration

To address both persistence and ambiguity, we use:

```bash
~/.ssh/config
```

This file allows us to define **explicit rules for how SSH selects keys**.

---

Before writing config:

```bash
git@github.com
```

* `github.com` → remote host
* `git` → SSH user (not your GitHub username)

Your identity is determined by the **SSH key presented**, not the username.

---

#### What SSH Config Solves

* **Key selection** → correct key always used
* **Reduced ambiguity** → no trial-and-error
* **Better UX** → optional auto-loading into agent

> SSH config does not persist keys, but ensures **correct key selection**

---

#### Example: Single Account Setup

```bash
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/Our GitHub account name
  AddKeysToAgent yes
```

---

#### Example: Multiple Accounts

```bash
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/personal_ed25519

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/work_ed25519
```

---

Update remotes:

```bash
git@github-personal:Our GitHub account name/Our GitHub account name-git-practice.git
git@github-work:acme-corp/payment-service.git
```


> SSH keys establish **identity**, Git platforms enforce **authorization**, and SSH config ensures the **correct identity is used consistently**.

---

### Step 2: Create a Remote Repository

Go to GitHub and create a new repository:

* Name: `Our GitHub account name-git-practice`
* Visibility: Public

> **Note:** Even if a repository is public, only users with **write access** can push changes. Everyone else can only read (clone).

After creation, GitHub shows setup commands. We will ignore them for now and instead follow our own flow.

Copy the repository URL.

---

### Step 3: Create Local Repository & Commits

Initialize your repository and create some commits:

~~~bash
git init

echo "Initialize application" > app.txt
git add app.txt
git commit -m "chore: initialize application"

echo "Improve application header styling" >> app.txt
git add app.txt
git commit -m "style: improve application header styling"

echo "Add footer section to application" >> app.txt
git add app.txt
git commit -m "feat: add footer section"
~~~

We intentionally created multiple commits so we can later observe how **history moves from local to remote**.

---

### Step 4: Add Remote

Before pushing, Git must know **where to push**.

Check existing remotes:

~~~bash
git remote
~~~

No output means no remotes are configured.

---

#### Choosing the Correct Remote URL

At this point, it is important to understand that the **remote URL determines how authentication will happen**.

There are two common formats:

##### 1. SSH (Recommended for this setup)

~~~bash
git@github.com:Our GitHub account name/Our GitHub account name-git-practice.git
~~~

* uses **SSH protocol**
* authentication happens using **SSH keys**
* aligns with the setup we just configured (ssh-agent, SSH config)


##### 2. HTTPS

~~~bash
https://github.com/Our GitHub account name/Our GitHub account name-git-practice.git
~~~

* uses **HTTPS protocol**
* authentication happens using **tokens or credential helpers**
* does **not use SSH keys**




> **Important:** The remote URL determines:
>
> * **where Git connects**
> * **how authentication happens (SSH vs HTTPS)**
>
> If you use an HTTPS URL, Git will use HTTPS-based authentication and your SSH key setup will not be used.

---

#### Add Remote (Using SSH)

Since we configured SSH, we will use the SSH URL:

~~~bash
git remote add origin git@github.com:Our GitHub account name/Our GitHub account name-git-practice.git
~~~

* **`origin`** is just a name for the remote. It is a convention, not a keyword.
* You can name it anything, but `origin` is universally used for the primary remote.

---

#### Verify Remote

~~~bash
git remote -v
~~~

You should now see:

~~~text
origin  git@github.com:Our GitHub account name/Our GitHub account name-git-practice.git (fetch)
origin  git@github.com:Our GitHub account name/Our GitHub account name-git-practice.git (push)
~~~

This confirms that Git will connect to GitHub using **SSH**, and your configured SSH keys will be used for authentication.

---

### Step 5: Push Local Commits to Remote

This is the step where **local history becomes shared history**.
Until now, all commits exist only in your **local repository**. A push operation transfers those commits to a **remote repository**, making them visible to others.


Before pushing correctly, try the most natural command:

~~~bash
git push
~~~

You will see:

~~~text
fatal: The current branch main has no upstream branch.
~~~

At this point, Git knows:

* you are on the local branch `main`
* you want to push commits

But it does **not know the destination**, i.e.:

* **which remote?** (e.g., `origin`)
* **which remote branch?** (e.g., `origin/main`)

In other words, `git push` relies on an existing **upstream configuration** (a mapping between local and remote branches). Since this mapping does not yet exist, Git cannot determine where to push.

---

#### Providing Full Push Information

Now push by explicitly specifying the destination:

~~~bash
git push origin main
~~~

This tells Git:

* **remote** → `origin` (remote repository)
* **source** → local branch `main`
* **destination** → remote branch `main` on `origin`

---

#### Internal Representation

Internally, Git interprets this as:

~~~bash 
git push origin main:main
~~~

Syntax:

~~~bash
git push <remote> <local-branch>:<remote-branch>
~~~

So here:

* first `main` → **local branch (source)**
* second `main` → **remote branch (destination)**

---

#### What Happens Internally During Push

When you run `git push origin main`, Git performs the following:

1. Identifies commits present in **local `main` but not in `origin/main`**
2. Transfers only those missing commits to the remote
3. Updates the remote reference (`origin/main`) to point to the latest commit

If the remote branch does not exist:

* Git **creates the branch automatically** on the remote
* The branch will now track your pushed commits

---

#### Key Insight

At this stage:

* Local branch → `main`
* Remote branch → `origin/main`
* Mapping between them → **not yet configured (no upstream)**

This is why `git push` initially fails and requires explicit input.

---

#### **Verification**
You can now **verify** this on **GitHub** by opening your repository and checking the **branches list**. You will see `main` on the remote, confirming that your local branch has been successfully pushed.

If you open the `main` branch, you will also see `app.txt` along with the **three commits** you created locally. This confirms that your **local history is now available on the remote repository**.

In most cases, `main` already exists on GitHub because it is the **default branch** created when the repository is initialized. If it did not exist, the push operation would have **created a new branch named `main` on the remote**.


> **Note:** The *default branch* is the primary branch of a repository (commonly `main`). It is the branch you land on when you open the repository on GitHub, and is used by default for viewing code, creating pull requests, and as the base for new branches.

---

### Setting Up Upstream (Making `git push` Work Seamlessly)

In the previous section, we successfully pushed using:

~~~bash
git push origin main
~~~

However, this introduces an important limitation.

---

### 1. The Limitation: No Upstream Mapping

Even though the push succeeds, Git does **not remember the relationship** between:

* local branch → `main`
* remote branch → `origin/main`

So if you run again:

~~~bash
git push
~~~

it will fail with:

~~~text
fatal: The current branch main has no upstream branch.
~~~

This happens because Git still does not know:

* which remote to use
* which remote branch to push to

As a result, you would need to repeat:

~~~bash
git push origin main
~~~

every time.

---

### 2. Solution: Set Upstream with `-u`

To make this relationship persistent, use:

~~~bash
git push -u origin main
~~~

This does two things:

1. Pushes the branch to the remote
2. Creates an **upstream mapping**:

> `main` → `origin/main`

---

#### What Upstream Means

Upstream is a **stored relationship** that tells Git:

* where to push (`git push`)
* where to pull from (`git pull`)

Once set, Git can resolve commands automatically:

~~~bash
git push
git pull
~~~

Internally, these become:

~~~bash
git push origin main
git pull origin main
~~~

> We will discuss **git pull** in a little while.

---

### 3. Applying the Same Concept to New Branches

Whenever you create a new branch, it has **no upstream mapping by default**.

Example:

~~~bash
git checkout -b feature/login
git push
~~~

This fails for the same reason.

---

#### Fix (Two Approaches)

**Option 1: One-time push**

~~~bash
git push origin feature/login
~~~

✔ Pushes the branch
✖ Does not store mapping

---

**Option 2: Persistent mapping (recommended)**

~~~bash
git push -u origin feature/login
~~~

This creates:

> `feature/login` → `origin/feature/login`

After this:

~~~bash
git push
git pull
~~~

work seamlessly.

---

### 4. Verifying Upstream Mapping

You can verify the mapping using:

~~~bash
git branch -vv
~~~

Example:

~~~text
* feature/login  abc1234 [origin/feature/login] commit message
~~~

This shows:

* local branch → `feature/login`
* upstream → `origin/feature/login`

---

You can also check status:

~~~bash
git status
~~~

This tells you whether your branch is:

* ahead of remote
* behind remote
* in sync

---

### 5. Advanced: Different Local and Remote Branch Names

So far, we assumed:

> local branch = remote branch

Example:

> `feature/login` → `origin/feature/login`

However, Git allows **different names**.

#### Example

~~~bash
git checkout main
git checkout -b bugfix/typo
~~~

Push with a different remote name:

~~~bash
git push -u origin bugfix/typo:bugfix/typo-corrected
~~~

Syntax:

~~~bash
git push <remote> <local-branch>:<remote-branch>
~~~

Here:

* `bugfix/typo` → local branch
* `bugfix/typo-corrected` → remote branch

This means:

> push local `bugfix/typo` as `bugfix/typo-corrected` on remote

If the remote branch does not exist, Git creates it.

---

#### Resulting Mapping

With `-u`, Git stores:

> `bugfix/typo` → `origin/bugfix/typo-corrected`

---

#### Verify Mapping

~~~bash
git branch -vv
~~~

Example:

~~~text
* bugfix/typo  abc1234 [origin/bugfix/typo-corrected] commit message
~~~


> **Note:** While Git allows different local and remote branch names, most teams prefer keeping them identical for simplicity and predictability.

---

### Step 6: Cleanup (Local and Remote Branches)

Over time, branches get merged or become unnecessary. Cleaning them up is an important part of maintaining a clean repository.

---

#### Delete local branches

Let’s delete the branches we created:

~~~bash id="d4k8mz"
git checkout main
git branch -d feature/login
git branch -d bugfix/typo
~~~

* `-d` deletes the branch safely (only if fully merged)
* use `-D` to force delete (not recommended unless necessary)

At this point, the branches are removed **only from your local repository**.

---

#### Verify local deletion

~~~bash id="p3x7ql"
git branch
~~~

You should no longer see `feature/login` or `bugfix/typo`.

Before deletion, you could also inspect the **local to remote mapping** using:

~~~bash id="r1f7nh"
git branch -vv
~~~

which shows how local branches were linked to their corresponding remote branches. This becomes especially important when local and remote names differ (like `bugfix/typo` → `bugfix/typo-corrected`).

---

#### Delete remote branches

Now remove them from the remote.

~~~bash id="v9m2wr"
git push origin --delete feature/login
git push origin --delete bugfix/typo-corrected
~~~

* `feature/login` → same name locally and remotely
* `bugfix/typo` → remote name was `bugfix/typo-corrected`, so deletion uses the **remote name**

> **Note:** You can also delete branches from the GitHub UI (branches section). However, most engineers prefer using the CLI as it is faster, scriptable, and works consistently across environments.



---

#### Verify on GitHub

Go to your repository on GitHub and check the **branches list**.
You should no longer see:

* `feature/login`
* `bugfix/typo-corrected`


> **Key Insight:** Local and remote branches are **independent**. Deleting a branch locally does **not remove it from the remote**. A branch effectively exists in two places, your local machine and the remote repository, so cleanup must be performed in both locations explicitly.

---

## Understanding `git fetch` and `git pull`

So far, we have only pushed changes from our local repository to the remote. In a real-world setup, this flow is not one-directional. Other developers will also be pushing changes, which means your local repository can become **out of sync with the remote**.

To handle this, Git provides the `git fetch` command.

When you run `git fetch`, Git connects to the remote repository and **downloads new commits and updates**, but importantly, it **does not modify your current working branch or files**. Instead, it updates **remote-tracking branches**, such as `origin/main`.

> By default, `git fetch` updates all remote-tracking branches configured for that remote (e.g., `origin/main`, `origin/dev`), not just the current branch.

This creates a clear separation:

* your **local branch (`main`)** → what you are currently working on
* **remote-tracking branch (`origin/main`)** → latest known state of the remote

After a fetch, these two can differ, and that difference allows you to **inspect changes before applying them**.


This is the key distinction:

* `git fetch` → download changes only
* `git pull` → download + apply (merge) changes

You should think of `git fetch` as a **safe, non-intrusive way to observe what has changed on the remote** before deciding what to do next.

---

### What Happens During `git pull`

When you run:

~~~bash
git pull
~~~

Git performs two steps:

**1. The Fetch Step**

By default, `git pull` first runs `git fetch` under the hood.

In most standard configurations, this updates **all remote-tracking branches** for that remote (e.g., `origin/main`, `origin/dev`), not just the current branch.

**2. The Merge/Rebase Step**

After fetching, Git integrates changes into the **currently checked-out branch**.

* If using default behavior → performs a **merge**
* If configured (`pull.rebase=true`) → performs a **rebase**

This step applies changes from the upstream branch (e.g., `origin/main`) into your local branch (`main`).

> **Key Insight:**
> `git fetch` updates your view of the remote (all tracked branches), while `git pull` applies changes only to your **current branch**.

> **Typical workflow:**
> Use `git fetch` to review incoming changes, then use `git pull` when you are ready to apply them.
> In fast-moving or trusted branches, `git pull` is often used directly for quick synchronization.

---

### Step 7: Simulate Changes from Another Developer

So far, all changes were made from our local machine and pushed to the remote. Now we simulate a real-world scenario where **another developer has made changes and pushed them to the same repository**.

Since we are working on a single machine, we simulate this using GitHub itself.

Go to your repository on GitHub, open `app.txt`, click **Edit**, and add a new line such as:

~~~text
Add search functionality
~~~

Commit this change directly from the GitHub UI.

At this point, the state is important:

* remote repository → has a new commit
* local repository → does not have this commit

Your local repository is now **out of sync**.

---

### Step 8: Observe Mismatch and Sync with Remote

Go to your local machine and run:

~~~bash
git status
~~~

~~~text
On branch main
nothing to commit, working tree clean
~~~

This can be misleading. Your working tree is clean, but your repository is **not necessarily up to date with the remote**. Git is only showing the state of your **local changes**, not whether the remote has new commits.

At this point, a natural question is:

> “How do I know if the remote has changed?”

The answer is: **you must first contact the remote**.

---

**Run:**

~~~bash
git fetch
~~~

Git now connects to the remote and **downloads new commits for all tracked branches** (not limited to the current branch), updating the corresponding **remote-tracking branches** (e.g., `origin/main`). Importantly, it still **does not modify your local branch (`main`) or working directory**.

> **Note:** By default, `git fetch` is **repository-wide** and updates all remote-tracking branches.
>
> To fetch a **specific branch:**
>
> ~~~bash
> git fetch origin main
> ~~~
>
> This fetches commits from the remote `main` branch and updates the corresponding remote-tracking branch `origin/main`.
>
> Similarly, to fetch another branch:
>
> ~~~bash
> git fetch origin feature/login
> ~~~
>
> This updates `origin/feature/login` without modifying your local branch.

You now have two views:

* `main` → your local state
* `origin/main` → latest remote state

**Check again:**

~~~bash
git status
~~~

~~~text
Your branch is behind 'origin/main' by 1 commit.
~~~

Now the picture is clear: your local branch is outdated compared to the remote.


**Inspect the exact change:**

~~~bash
git log --oneline --graph main..origin/main
~~~

This shows commits present in **remote (`origin/main`) but not in your local branch (`main`)**.

---

### Verify file-level changes (important)

Before pulling, you can **see exactly what changed inside files**.

Check your current local file:

~~~bash
cat app.txt
~~~

Compare local vs remote version:

~~~bash
git diff main origin/main -- app.txt
~~~

Or see all file changes:

~~~bash
git diff main origin/main
~~~

This helps you **understand what the remote is bringing in before applying it**.

At this stage, your repository **knows about the changes but has not applied them**.

---

**Now apply them:**

~~~bash
git pull
~~~

This fetches (if needed) and merges `origin/main` into your **current branch (`main`)**, bringing it up to date.

> **Note:** `git pull` performs a fetch followed by a merge (or rebase).
> The merge may result in a **fast-forward update** or a **3-way merge**.
> If there are conflicting changes, Git will raise a **merge conflict** that must be resolved manually.
> While the fetch step may update multiple remote-tracking branches, the **merge is applied only to the currently checked-out branch**.

---

### Verify after applying

Check updated file:

~~~bash
cat app.txt
~~~

Check commit history:

~~~bash
git log --oneline
~~~

~~~
8f1bf09 (HEAD -> main, origin/main, origin/HEAD) feat: Add search functionality to the application
2eb53ad feat: add footer section
315b489 style: improve application header styling
c66a006 chore: initialize application
~~~

The commit added by another developer (simulated via GitHub UI) is now part of your **local history**, which is exactly how collaboration works: changes made elsewhere are fetched, reviewed, and integrated into your system.

> **Note:** Commits created via GitHub UI may show a `noreply` email instead of a real email. This is intentional, GitHub masks user emails for **security and privacy**, while still correctly attributing the commit to the user.

> **Key Insight:** `git fetch` gives you **visibility without impact**, while `git pull` applies those changes to your working branch. This separation is what allows teams to work safely without accidentally overwriting or blindly merging changes.


---

### Understanding `git diff`

In Step 9, we used `git diff` to compare our **local state with the remote** and inspect what changes would be pulled. Now let’s formalize and extend that understanding.

`git diff` shows the **line-by-line differences between two states in Git. These states could be:**

* your working directory vs staging area
* staging area vs last commit
* one branch vs another
* local branch vs remote branch

In simple terms: **What exactly changed?**



**Useful Commands:**

| Comparison             | Command                     |
| ---------------------- | --------------------------- |
| working vs staging     | `git diff`                  |
| working vs last commit | `git diff HEAD`             |
| staging vs last commit | `git diff --staged`         |
| local vs remote        | `git diff HEAD origin/main` |

> **Note 1:** `git diff` compares **working directory vs staging area**. If nothing is staged, it effectively shows changes against the **last commit**, which is why it may appear similar to `git diff HEAD`.

> **Note 2:** `git diff HEAD origin/main` explicitly shows **local (`HEAD`) vs remote (`origin/main`)**. While `HEAD` is the default and can be omitted, keeping it improves clarity by making the **comparison direction explicit**.



`git diff` gives you **complete visibility across all states before you commit or merge**, helping you understand exactly what is changing at every stage.

> Mental model: `git diff` shows what will change, before you apply it.

> **Mental hack:** Think of `git diff` in four areas: working directory → staging area → commit (HEAD) → remote.
> Use `git diff` for working vs staging, `git diff --staged` for staging vs commit, `git diff HEAD` for working vs commit, and `git diff HEAD origin/main` for your current branch vs remote-tracking branch.
> Move left → right across states to compare changes between any two points.

---

### Mini Demo: `git diff`

#### Step A: Create isolated branch

To avoid polluting `main`, perform this demo in an isolated branch and discard it later.

```bash
git checkout main
git checkout -b demo/diff
```

---

#### Step B: Working directory vs last commit

Start with a clean file and commit:

```bash
echo "Initialize application config" > config.txt
git add config.txt
git commit -m "chore: add initial config"
```

Modify the file:

```bash
echo "Add environment-specific settings" >> config.txt
```

Now run:

```bash
git diff
```

This compares **working directory vs staging area**. Since nothing is staged, it effectively shows changes against the **last commit**, with:

* `+` → newly added lines
* `-` → previous content

Explicitly:

```bash
git diff HEAD
```

→ working directory vs last commit

---

#### Step C: Staging area vs last commit

Stage the change:

```bash
git add config.txt
```

```bash
git diff
```

No output → working directory and staging area are identical

```bash
git diff --staged
```

→ staging area vs last commit

At this point:

* working → same as staging
* staging → differs from commit
* commit (HEAD) → previous version

---

#### Step D: Finalize and clean up

Bring everything into a single committed state:

```bash
git add config.txt
git commit -m "chore: update config with environment settings"
```

Return to a clean baseline:

```bash
git checkout main
git branch -d demo/diff
```

If you see an error like:

```text
error: the branch 'demo/diff' is not fully merged
```

* This means the branch contains **commits that have not been merged** into the current branch
* Git blocks deletion using `-d` to **prevent accidental data loss**

In such cases, if you are sure you no longer need the branch:

```bash
git branch -D demo/diff
```

* `-D` force deletes the branch, even if commits are unmerged

> Use `-D` carefully, as it can permanently remove unmerged work

---

## Pull Requests and Code Review

So far, we have seen how multiple developers can **work on the same repository, push changes, and stay in sync** using `push`, `fetch`, and `pull`. However, this model assumes that developers are directly pushing changes to shared branches like `main`.

In real-world systems, this is almost never allowed.

The reason is simple: **any direct change to a shared branch can impact the entire system**. A broken commit pushed to `main` can immediately affect builds, deployments, and other developers. As team size and system complexity grow, this risk becomes unacceptable.

To solve this, teams introduce a controlled workflow using **Pull Requests (PRs)**.

---

### Pull Requests and the Problem They Solve

A Pull Request is a mechanism through which a developer proposes changes from one branch to another, most commonly:

> `feature branch` → `main`

Here, `main` represents a **stable line of development**, a branch that is expected to always remain in a releasable state. Because of this, changes are not applied to it directly.

Instead, a developer creates a separate branch, makes changes in isolation, pushes that branch to the remote, and requests that those changes be reviewed and merged. This introduces a controlled entry point for all changes.

Without this model:

* changes go directly into `main`
* issues are discovered after integration
* debugging becomes reactive

With Pull Requests:

* changes are **reviewed before merging**
* issues are caught early
* the main branch remains stable
* collaboration becomes structured

This is the shift:

> we move from **pushing code** to **proposing and validating changes before they enter the system**

> **Note:** Git itself does **not have Pull Requests**. Pull Requests are provided by platforms like GitHub, GitLab, and Bitbucket. Git only understands branches and commits. A Pull Request is a **platform-level abstraction** built on top of branch comparison, diffs, and merges.

---

At the center of a Pull Request is **code review**, which is typically performed by looking at the **diff between branches**, showing exactly what changed. Reviewers validate correctness, design, and intent, while also ensuring consistency and enabling knowledge sharing across the team.

In practice, you will often hear two terms:

* **PR (Pull Request)** → popularized by GitHub
* **MR (Merge Request)** → used by GitLab

Both mean the same thing. “Merge Request” is arguably more explicit, but since this course uses GitHub, we will continue with **PR**.

---

### How PR Approval Works in Practice

A Pull Request is not merged immediately. It must first be **approved**, and this typically happens in two ways depending on the system and team practices.

**1) Automated approval (CI-driven)**

A CI pipeline or automation runs checks on the proposed changes, such as:

* build validation
* tests
* linting or policy checks
* infrastructure validation

If all checks pass (green), the system allows the PR to be merged. If any check fails (red), the merge is blocked.

Here, approval is based on whether the change satisfies the **defined technical conditions**.

---

**2) Manual approval (human review)**

A reviewer examines the changes and approves the PR.

* could be a single approver or multiple
* reviewers are notified when a PR is raised
* approval is given after validating logic, design, and impact

Even in systems with strong automation, a manual approval step is often retained for **critical or production-impacting changes**.

---

>**Key Insight:** In most real-world systems, both are combined; automation ensures the change **works correctly** (builds, tests, policies), while humans ensure the change is **the right approach** (design, scalability, alignment). Automation catches failures, humans catch poor decisions. Both together ensure the system remains **correct and well-designed**.

---

### Pull Request Workflow in Production

![Alt text](/images/f2.png)

A typical production workflow follows a structured sequence where changes are introduced in a controlled manner. The diagram above visually represents this end-to-end flow.

---

1. **Create a working branch**

   * A developer creates a new branch from `main`, which represents the **stable, protected, and releasable state** of the system. Although often called a *feature branch*, this branch can represent any unit of work such as a bugfix, enhancement, or spike.
   * The key idea is **isolation from the stable branch**, ensuring ongoing work does not impact production-ready code.
   * This also ensures that incomplete or experimental changes do not affect other developers or the system state.

---

2. **Implement changes locally**

   * All development happens in this branch. The developer **develops, validates, and commits changes locally**, iterating safely without affecting others.
   * Changes are built incrementally through commits, allowing the developer to maintain a clear and traceable history.
   * At this stage, work remains **private and isolated**, and only reasonably validated changes are prepared for sharing.

---

3. **Push the branch to remote**

   * The branch is pushed to the remote repository, making the work **visible and shareable**.
   * This step transitions the work from a **local-only state to a collaborative context**.

   This enables:

   * Collaboration with the team
   * The ability to create and maintain a Pull Request
   * Integration with automation systems (CI/CD, checks, policies)

---

4. **Create a Pull Request**

   * A Pull Request (PR) is raised to propose merging the working branch into `main`.
   * This is a critical transition point where work moves from **individual ownership to team visibility and control**.

   The PR:

   * Targets a **protected `main` branch (no direct commits allowed)**
   * Shows the **diff between branches**, making changes explicit
   * Acts as the **single controlled entry point** for changes
   * Initiates the **validation and approval workflow**

---

5. **Review and validation**

   * The PR undergoes both **automated validation** and **manual review** before it can be merged.

   During this stage:

   * Checks run on every push to branches with open PRs
   * Any new commit automatically updates the PR and re-triggers checks
   * All required checks (build, test, lint, policies) must pass
   * Required approvals must be satisfied

   If issues are found:

   * Changes are fixed locally
   * Commits are pushed to the same branch
   * The PR updates automatically and re-enters validation

   This creates an **iterative feedback loop**, ensuring both **correctness (automation)** and **quality (human review)** before integration.

---

6. **Merge into `main`**

* Once all checks pass and approvals are complete, the PR is merged into `main`.

At this point:

* `main` remains a **protected branch** throughout
* Only **reviewed and approved changes** are allowed to enter
* The system state is updated in a **controlled, traceable, and auditable manner**
* This merge often **triggers automated workflows (CI/CD pipelines)** such as:
  * building the application (compile/package)
  * running tests again on the merged code
  * creating deployable artifacts (images, binaries)
  * deploying the application to environments (staging, production)
* These pipelines are called **downstream** because they run **after the code is merged**, using `main` as the **source of truth**


---

7. **Delete the working branch**

   * After a successful merge, the working branch is deleted.

   This:

   * Removes temporary development branches
   * Keeps the repository clean and focused
   * Prevents reuse of outdated branches
   * Ensures future work starts from a fresh baseline (`main`)


> **Key Insight:** A Pull Request is not about merging code, it is about **validating and controlling how code gets merged into a protected branch**.


Now that we understand how Pull Requests work in theory, let’s move to the practical setup.

---

### Step 9: Protect the `main` Branch Using Rulesets (Production Setup)

So far, we were directly pushing changes to `main`. In real-world systems, this is **not allowed**.

`main` represents the **stable line of development**, a branch expected to always remain **releasable**. Any direct, unreviewed change can break builds, deployments, or impact other developers. As systems grow, relying on discipline is not enough, enforcement is required.

> **Note:** In our case, we protect `main` because it is the primary branch. In real systems, any branch representing a **stable or release line** (`main`, `develop`, `release/*`) can be protected using rulesets.

To enforce this, GitHub provides **rulesets**, which define how branches can be modified.

Go to your repository:
[Mention Github Repo URL detail](Mention Github Repo URL detail)

Navigate to:

* **Settings → Rules → Rulesets → New ruleset**


**Configure the ruleset**

Define:

* **Ruleset name** → `protect-main`
* **Target branches** → `main`

This tells GitHub:

> “Apply these rules whenever someone tries to modify `main`”


Under rules, enable:

* **Require a pull request before merging**
* **Require approvals** (at least 1)
* **Block force pushes**
* (optional) **Require status checks to pass**

These controls enforce:

* no direct commits to `main`
* all changes must go through a **Pull Request**
* changes must be **reviewed and approved**

---

Once saved, try pushing directly to `main` and it will fail. This is intentional.

> The system is now enforcing the workflow, not relying on developer discipline.

---

### Step 10: Create a Branch, Make Changes, and Push

Now that `main` is protected, we can no longer push changes directly. We must follow the controlled workflow.

Create a branch from the latest state of `main`:

~~~bash
git checkout main
git checkout -b feature/search
~~~

This branch represents an **isolated unit of work**. All changes will happen here without impacting the stable branch.

Make a change:

~~~bash
echo "Enhance search functionality with filters" >> app.txt
git add app.txt
git commit -m "feat: enhance search functionality with filters"
~~~

At this point, the change exists only in your **local branch**.

Push it:

~~~bash
git push -u origin feature/search
~~~

Now:

* your branch exists locally
* your branch exists on the remote
* `main` remains unchanged

This is the key separation:

> work is **visible**, but not yet **part of the system**

---

### Step 11: Create and Approve a Pull Request

Go to your repository on GitHub.

You will see a prompt to create a Pull Request for `feature/search`.

Create a PR with:

* base → `main`
* compare → `feature/search`


At this point, the platform shows:

* commits in your branch
* diff between `feature/search` and `main`
* files that were modified

This becomes the **review surface**, where changes are evaluated before they are allowed into `main`. Reviewers can clearly see what changed, why it changed, and whether it should be accepted.


In this demo, you will both **raise and approve** the Pull Request.

However, it is important to understand how this works in real-world systems.

* The person who raises the PR is typically **not the one approving it**
* Reviews are done by:

  * a teammate
  * a tech lead
  * a manager or designated reviewer

> If you configure the repository to require **1 reviewer**, and you are the same person who created the PR, your approval will **not be considered valid**. This is by design, as a system should not allow self-approval for critical changes.

For the purpose of this demo:

* You may temporarily **set required reviewers to 0**, or
* Simply proceed without enforcing reviewer requirements


To approve the PR correctly:

1. Go to the **“Files changed”** tab
2. Click **“Review changes”**
3. Select **“Approve”**
4. Click **“Submit review”**

> Simply commenting “Approved” does **not** count as an approval.


Once approved, **merge the Pull Request.**

This is the only point where `main` is updated. Until this moment, all changes remain isolated in the feature branch.

> **Note:** In local Git, when there is no branch divergence, a merge results in a **fast-forward** and no merge commit is created. However, when merging a Pull Request on platforms like GitHub, a **merge commit is created by default**, even if a fast-forward is possible. This behavior is also commonly seen in platforms like GitLab and Bitbucket. If you do not want a merge commit, you can use **rebase and merge** or **squash and merge**, which we will cover later in the course.


> **Key Insight:** Earlier, we relied on discipline to avoid breaking `main`. Now, the system enforces it. Changes are isolated, reviewed, validated, and only then merged into the stable branch. This is how production systems maintain reliability at scale.

---

### Step 12: Sync Local Repository After Merge

The Pull Request was created and merged on the remote, not from your local machine. This means your **local repository is now outdated**, even though you did not make any conflicting changes locally.

Your local `main` still reflects the state **before the PR was merged**, while the remote `main` has moved forward.

To align your local repository with the updated system state, run:

~~~bash
git checkout main
git pull
~~~

This does two things:

* ensures you are on the `main` branch
* fetches the latest changes from the remote and merges them into your local `main`

At this point:

* `origin/main` → already had the merged changes
* `main` → now updated to match `origin/main`

Your local repository is now **in sync with the remote**, and you are working on the latest, stable version of the system.

> **Key Insight:** In collaborative workflows, changes often happen outside your local environment (via PRs, UI merges, or other developers). Regularly syncing ensures you are always building on the **current system state**, not an outdated view.
>
> `git pull` (or `git fetch` + merge) updates only the **branch you are currently on**. Other local branches remain unchanged. In practice, developers usually pull on `main` (or the stable branch) since it represents the **latest shared state**, and then branch off from it for new work.

---

### Step 13: Attempt Direct Push to Protected Branch

Now that your local `main` is in sync, let’s try to **make a direct change and push it to `main`**.

Create a change:

```bash
echo "Hotfix applied directly on main" >> app.txt
git add app.txt
git commit -m "fix: hotfix directly on main"
```

Attempt to push:

```bash
git push origin main
```

You will see the push **fail**.

This happens because in **Step 9**, we configured branch protection rules on `main`, which prevent:

* direct pushes to the branch
* bypassing the Pull Request workflow

---

Now try forcing the push:

```bash
git push --force origin main
```

This will also **fail**, because we explicitly enabled: **Block force pushes**

---

#### What just happened?

Even though:

* you have valid changes
* your branch is up to date
* you are the repository owner

GitHub still **rejects the push**.

This is because the system enforces:

* all changes must go through a **Pull Request**
* no one can bypass the process, even with `--force`

> **Key Insight:** Earlier, we relied on discipline to avoid breaking `main`. Now, the system enforces it. Direct pushes and force pushes are blocked, ensuring that **every change is reviewed, validated, and approved before integration**.

---

### Step 14: Cleanup (Local and Remote)

At this point, the demo is complete. Let’s clean up both the **local environment** and the **remote repository**.

---

#### Local Cleanup

Remove the Git repository and files created during the demo:

```bash
rm -rf .git
rm -rf app.txt
```

> **Caution:** The `-rf` flag forcefully deletes files and directories without confirmation.
> Always double-check your path before running this command, as it can permanently remove important data.

---

#### Remote Cleanup

Delete the repository from GitHub:

1. Go to your repository: `Our GitHub Account repo detail`
2. Navigate to **Settings**
3. Scroll to the bottom of the page
4. Click **“Delete this repository”**
5. Confirm by typing the repository name


> **Cleaning** up after demos ensures your environment stays organized and avoids accidental reuse of test repositories or configurations in future work.

---

## Git Stash

### Why Git Stash

* In real-world development, you are often working on a branch (feature, bugfix, hotfix) with **unfinished or in-progress changes** in your workspace (working directory or staging area).
* Suddenly, you may need to **switch context immediately**. For example:

  * something broke in production and you need to inspect the `main` (or `prod/release`) branch
  * a teammate asks you to quickly verify a change
  * you need to pull the latest updates before continuing your work
* In such situations, your current work is **not complete and not ready to be committed**, but you also **cannot lose it**.

---

At this point, Git evaluates safety:

* Git blocks operations like checkout or pull **only when they would overwrite your uncommitted changes**, because that would lead to data loss.

So you are left with two options:

* **Commit the changes** → not ideal, because commits should represent **clean, meaningful units of work**, not half-done code
* **Temporarily move the changes aside** → so you can safely switch context

This is where `git stash` becomes useful.

---

### What is Git Stash

```bash
git stash
```

* `git stash` **temporarily removes your uncommitted changes** from the working directory and stores them safely, without adding them to branch history.
* After stashing, your workspace is reset to match the **last committed state (HEAD)**, giving you a **clean and safe environment** to switch branches, pull changes, or investigate issues.
* Later, you can bring those changes back using `git stash pop`, which **reapplies them on top of your current branch state**.

> Stash is best understood as:
> **“Save my unfinished work, switch context safely, and come back later.”**

---

### Quick Decision Guide

* **Commit** → Use when your changes are **complete, stable, and logically meaningful**, because commits become part of project history and should be easy to understand later.
* **Stash** → Use when your changes are **incomplete or temporary**, but you don’t want to lose them and need a **clean workspace to switch context**.
* **Discard** → Use when your changes are **not needed anymore**, and you are okay with permanently removing them.

> If your work is not ready to be committed but you need to switch context safely, **stash is the right tool**.

---

## Demo: Git Stash

### Step 1: Initialize Repository and Establish a Common Base

```bash
git init git-stash-demo && cd git-stash-demo

echo "My name is Sunil Raj" > intro.txt
git add intro.txt
git commit -m "A: initial intro"

echo "My name is Sunil Raj Ramachandran" > intro.txt
git add intro.txt
git commit -m "B: stable baseline"

git checkout -b feature/intro-update
```

We now have a **meaningful history with two commits**, and both branches start from a stable baseline.

```text
A -------- B (main, feature/intro-update)
```

State:

* `intro.txt` at commit **B** → `My name is Varun`
* Both `main` and `feature/intro-update` point to **the same commit (B)**
* Your repository is in a **clean state (no local changes)**

---

### Step 2: Branch Switch Evaluation (Two Possible Behaviors)

```bash
git checkout <branch>
```

Before switching, Git evaluates:

> **“Will switching overwrite my uncommitted changes?”**

* Git does not blindly switch branches; it checks whether completing the checkout requires **modifying any files that currently have uncommitted changes**.
* The decision is based purely on **whether a file needs to be updated**, not on how big the change is or how different the branches are.
* If no file needs to be changed → safe
* If a file must be updated and is already modified → blocked

---

#### Case A: Checkout Allowed (No File Update → No Overwrite Risk)

```bash
echo "My name is Raj Sunil" > intro.txt
```

```bash
git checkout main
```

```text
Current → Target
B       → B
```

* **Current = Target**, so Git does **not need to update the file**
* Since no file update is required, your uncommitted changes are **safe**

→ ✅ Checkout allowed

```text
My name is Sunil Raj Ramachandran
```

> No overwrite risk → safe to switch

---

#### Case B: Checkout Blocked (File Update Needed → Overwrite Risk)

```bash
git checkout main
echo "I live in Coimbatore" >> intro.txt
git add intro.txt
git commit -m "C: add location"
```

```text
A -------- B -------- C (main)
           \
            (feature/intro-update)
```

```bash
git checkout feature/intro-update
echo "My name is Sunil Raj Ramachandran" > intro.txt
```

```bash
git checkout main
```

```text
Current → Target
B       → C
```

* **Current ≠ Target**, so Git **must update the file (B → C)**
* But your file already has uncommitted changes
* Updating would **overwrite your work**

→ ❌ Checkout blocked

```text
your change → overwritten by C
```

> File update needed → overwrite risk → Git blocks

---

### Step 3: Checkout Gets Blocked

```bash
git checkout main
```

```text
error: Your local changes to the following files would be overwritten by checkout:
        intro.txt
Please commit your changes or stash them before you switch branches.
Aborting
```

* Git tries to make your working directory match **commit C**
* That requires updating `intro.txt`
* Since it already has uncommitted changes, proceeding would **overwrite your work**
* Git **blocks the operation to protect your changes**

---

### Step 4: Resolve Using `git stash`

```bash
git stash
git checkout main
```

* `git stash` **temporarily removes uncommitted changes** and stores them safely
* Working directory is reset to **HEAD (commit B)**
* This removes the overwrite risk

```text
Before stash → intro.txt = My name is Varun Joshi  
After stash  → intro.txt = My name is Varun (commit B)  
```

Now:

* No local modifications exist
* Git can safely update from **B → C**

→ ✅ Checkout succeeds

---

#### Checking Stash History

```bash
git stash list
```

```text
stash@{0}: WIP on feature/intro-update: B stable baseline
```

* Stash entries are stored as a **stack (latest first)**
* Each entry represents your **saved uncommitted state**

---

### Step 5: Restore Changes

```bash
git checkout feature/intro-update
git stash pop
```

* Restores your changes back on the **same branch where they were created**
* No base change → clean and predictable restore
* This is the **most common and simplest usage of stash**


> **Note:**
> You can also apply a stash on a different branch (for example, `main`).
> In that case, Git will attempt to apply your changes on a **different base commit**, which may lead to conflicts.
> We are not doing that here to keep the behavior simple and predictable.

---

#### Applying a Specific Stash

```bash
git stash list
```

```text
stash@{0}: WIP on feature/intro-update: B stable baseline  
stash@{1}: WIP on main: C changes  
```

```bash
git stash pop stash@{1}
```

* Applies a specific stash
* Removes it after applying

```bash
git stash apply stash@{1}
```

* Applies but **keeps the stash**

---

### Key Pointers for `git stash`

* Git blocks checkout only when it must **modify a file that already has uncommitted changes**
* Checkout works by making your working directory match the **target commit**
* `git stash` removes those changes temporarily → **eliminates overwrite risk**
* After switching, `git stash pop` reapplies changes on the **new base**

> If switching would overwrite your changes, Git blocks it.
> Remove that risk, and the operation succeeds.

---

### Deep Understanding, Mental Model, and Core Principle of Git Stash

* Git blocked checkout to **protect uncommitted changes from being overwritten**

* `git stash` performs **state capture and isolation**:

  * captures working directory + staging area
  * stores as commit-like objects in Git database
  * referenced via `refs/stash`
  * maintained as a **stack (LIFO)**

* Working directory is reset to **HEAD → clean state**

* Enables safe context switching (branch change, pull, debugging production issues)

* `git stash pop`:

  * restores full working state
  * removes stash entry
  * resumes work exactly where you left

---

## Conclusion

In this Collaboration & Production Workflow, we moved from **local Git workflows to production-grade collaboration**.

We saw how:

* remote repositories act as a **shared system of record**
* authentication ensures **secure and controlled access**
* `git push`, `fetch`, and `pull` enable **synchronization across developers**
* `git diff` provides **visibility before applying changes**
* Pull Requests introduce a **controlled entry point for changes**
* branch protection enforces **system-level safety**
* `git stash` allows safe context switching without polluting history

The key shift is this:

> Git is no longer just tracking changes, it is **controlling how changes enter a stable system**.

At this stage, you are working in a model where:

* changes are **isolated**
* changes are **reviewed**
* changes are **validated**
* and only then **integrated**

In Advanced History & Integration Strategies, we will go deeper into how changes are integrated by exploring **rebase** and **squash**, and when to use them in real-world scenarios.
We will also bring everything together into a **production-grade end-to-end workflow**, covering how teams manage clean history while maintaining stability.

---

## References

- Git Documentation  
  https://git-scm.com/docs  

- GitHub Docs: SSH Authentication  
  https://docs.github.com/en/authentication/connecting-to-github-with-ssh  

- GitHub Docs: About Pull Requests  
  https://docs.github.com/en/pull-requests  

- GitHub Docs: Protected Branches & Rulesets  
  https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository  

**Recommended Reading (Deeper Understanding)**

- Pro Git Book (Free)  
  https://git-scm.com/book/en/v2  

- Git Internals (Advanced)  
  https://git-scm.com/docs/gitcore-tutorial  

---
