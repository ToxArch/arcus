DRAFT - NOT LEGALLY REVIEWED
# Contributing to ARCUS

Thank you for your interest in contributing to ARCUS. Because ARCUS is a
source-available, permission-required project with specialized legal and
licensing requirements, all contributions follow a structured governance
process.

---

### 1. The Contributor License Agreement (CLA) Requirement
Before any pull request, patch, documentation improvement, or code change can be
merged, **you must review and agree to the [ARCUS Contributor License Agreement](CLA.md)**.

* Why we use a CLA: Standard open-source projects often use a simple "Developer
  Certificate of Origin" (DCO `Signed-off-by`). However, a DCO only certifies
  that you wrote the code; it does not give the project owner the legal freedom
  to relicense, commercialize, or re-structure the project in the future.
* The ARCUS CLA ensures that you retain your own copyright, but grant the
  Licensor the broad, permanent rights needed to steward and protect the
  project.

---

### 2. How Contributions Are Reviewed
1. **Scope Check:** Contributions must align directly with the project's
   technical scope (CGNAT attribution, flow telemetry, BSA 2023 evidence
   integrity, or router integration).
2. **Code Hygiene:**
   - All code must pass automated CI linters and test suites.
   - Zero emojis and zero em dashes in documentation and code comments.
   - No hardcoded secrets, external API keys, or proprietary telco dumps.
3. **Automated CLA Verification:** When you open a pull request, the automated
   CLA Assistant bot checks your signature status. If you have not signed
   previously, reply to the bot's comment on your PR with:
   ```
   I have read the CLA Document and I hereby sign the CLA
   ```
   The bot automatically records your signature and unlocks the pull request
   for review.

---

### 3. Reporting Issues
* **Bug Reports:** If you find an error in the research mathematics, formulas,
  or schemas, open a GitHub Issue using the standard template.
* **Security Issues:** Do NOT open public GitHub Issues for exploitable security
  vulnerabilities. Follow the private disclosure process in [SECURITY.md](SECURITY.md).
