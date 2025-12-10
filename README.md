# 🧠 AGENTS.md for AEM Projects

**Context for your AI Coding Assistants (Cursor, Copilot, Cline, etc.)**

## 👋 What is this?
Let's be honest: **AEM is complex.**

General-purpose AI coding agents often struggle with the nuances of Adobe Experience Manager. They might suggest:
* Using `java.io.File` on AEM Cloud Service (which wipes data on restart).
* Writing Java logic directly in HTL files.
* Using deprecated OSGi annotations or classic Replication APIs.
* Messing up the separation between `ui.apps` (immutable) and `ui.content` (mutable).

**`AGENTS.md` is the fix.**

It is a "rules of the road" file designed to be placed in the root of your AEM Maven project. When an AI agent (like Cursor's "Composer" or GitHub Copilot Workspace) reads the context of your repository, it reads this file to understand the strict architectural constraints of AEM.

## 🚀 How to use this
1.  **Copy** the `AGENTS.md` file from this repository.
2.  **Paste** it into the root directory of your AEM Project (alongside your main `pom.xml`).
3.  **Chat with your AI:** When you ask your AI agent to "Create a new Component" or "Write a Workflow," it will now reference these guidelines to ensure the code is Cloud Service compatible and follows Maven best practices.

## 📜 What does it cover?
This context file pre-prompts the AI on:
* **AEM as a Cloud Service Compatibility:** Strict rules against stateful servers and local file I/O.
* **Maven Structure:** Understanding `core`, `ui.apps`, `ui.config`, etc.
* **Coding Standards:** Java (Sling Models, OSGi R7), HTL, and Unit Testing (wcm.io).
* **Dispatcher:** Safety rules (blocking `/crx`, etc.).
* **Build Commands:** The correct Maven flags to use for local development vs. deployment.

## 🤝 Contributing
**This is a community effort.**
AEM is constantly evolving, and so are the best practices.

* Did you find an edge case the AI still gets wrong?
* Is there a new AEMaaCS feature that needs to be whitelisted?
* Did I make a typo in the Maven commands?

**Please raise a Pull Request!**
If you are an AEM Developer, your insights are valuable. Let's make this the standard "system prompt" for AEM development.

## 🆓 License
**Free to use.**
You can copy, modify, distribute, and use this file in personal, enterprise, or client projects without restriction. No attribution required (though a star on this repo is always appreciated!).

---
*Happy Coding!*

***
