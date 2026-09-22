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

