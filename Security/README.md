# Security (DevSecOps)

Building security in, not bolting it on.

## Basic Questions

1.  **Q: What is DevSecOps?**

    **A:** DevSecOps is a cultural and technical movement about integrating security practices into every phase of the DevOps lifecycle. The goal is to make security a shared responsibility of the entire team, not just the job of a separate security team that comes in at the end. It's an extension of the "Shift Left" philosophy to security.

2.  **Q: What is "Shifting Left" in security?**

    **A:** It means moving security testing and practices to as early in the software development lifecycle as possible. Instead of waiting for a final penetration test before release, you integrate security into the process from the very beginning:
    * **Design:** Threat modeling during the design phase.
    * **Code:** Using static analysis tools to find vulnerabilities in the code as it's being written.
    * **Build:** Running dependency scanning in the CI pipeline.
    * **Test:** Including security tests in your automated test suites.
    The earlier you find a security issue, the cheaper and easier it is to fix.

3.  **Q: What is SAST (Static Application Security Testing)?**

    **A:** SAST is a "white-box" testing methodology that analyzes an application's source code, byte code, or binary code for security vulnerabilities without actually running the application. It's great for finding issues like SQL injection, buffer overflows, and incorrect cryptography usage. SAST tools can be integrated directly into a developer's IDE or into the CI pipeline to provide fast feedback.

4.  **Q: What is DAST (Dynamic Application Security Testing)?**

    **A:** DAST is a "black-box" testing methodology that tests an application from the outside by simulating attacks against it while it's running. It doesn't have access to the source code. It's good at finding runtime vulnerabilities and issues with server configuration or authentication. DAST is typically run against a deployed application in a staging environment.

5.  **Q: What is Software Composition Analysis (SCA)?**

    **A:** Modern applications are built on a huge number of open-source libraries and dependencies. SCA is the process of automatically scanning your application's dependencies to identify all the open-source components you are using and check them against a database of known vulnerabilities (CVEs). This is a critical practice because a vulnerability in one of your dependencies is a vulnerability in your application. SCA tools should be run in your CI pipeline on every build.

## Intermediate Questions

1.  **Q: How would you handle secrets management in a DevOps environment?**

    **A:** This is one of the most important security practices.
    * **Never store secrets in Git.**
    * **Use a dedicated secrets management tool:** The best practice is to use a tool like HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault.
    * **Provide identities to machines:** Your applications and CI/CD pipelines should be given an identity (e.g., an IAM role in AWS or a Kubernetes service account).
    * **Fetch secrets at runtime:** The application, using its identity, authenticates to the secrets management tool and fetches the secrets it needs just before it starts. The secrets are injected into the application's environment and are never written to disk or logs.

2.  **Q: What is the principle of "Least Privilege"?**

    **A:** The principle of least privilege states that any user, program, or process should only have the minimum set of permissions (privileges) necessary to perform its function. For example, an application that only needs to read from an S3 bucket should have an IAM role that only grants `s3:GetObject` permission, not `s3:*`. This drastically reduces the "blast radius" if a component is compromised. An attacker who compromises that application can only do what that application was allowed to do.

3.  **Q: What is "Threat Modeling" and at what stage of the lifecycle should it be done?**

    **A:** Threat modeling is a structured process for identifying potential threats and vulnerabilities in an application *before* you even write a line of code. It should be done during the **design phase**. The team gets together and asks questions like:
    * What are we building?
    * What could go wrong? (e.g., What are the attack vectors?)
    * What are we going to do about it? (e.g., What controls can we put in place?)
    A common methodology is STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege). By thinking about security upfront, you can design a much more secure system from the start.

4.  **Q: What is IAST (Interactive Application Security Testing)?**

    **A:** IAST is a hybrid approach that combines elements of both SAST and DAST. It works by using an agent that is deployed along with the application in a test environment. This agent instruments the application and observes it from the inside as it runs. When a DAST scan or a manual test is performed from the outside, the IAST agent can see exactly how the application's code behaves in response to the attack. This allows it to pinpoint the exact line of code that is vulnerable with very high accuracy and almost no false positives.

5.  **Q: How do you secure a Docker container?**

    **A:** This is a multi-layered process.
    * **Secure the Host:** The underlying host machine must be hardened and patched.
    * **Secure the Dockerfile:**
        * Use a minimal, trusted base image.
        * Don't run as the `root` user. Add a `USER` instruction to run as a non-privileged user.
        * Use multi-stage builds to keep the final image clean of build tools and source code.
    * **Scan the Image:** Use an image scanning tool (like Trivy or Snyk) in your CI pipeline to check for known vulnerabilities in the OS packages and application dependencies within the image.
    * **Secure the Runtime:**
        * Use a container orchestrator like Kubernetes to manage the container.
        * Drop unnecessary kernel capabilities.
        * Use a read-only root filesystem where possible.
        * Apply network policies to restrict what the container can communicate with.

### Advanced Questions

1.  **Q: What is a "Security Champion" program?**

    **A:** A security champion program is a way to scale a security team and embed security expertise within development teams. A security champion is a developer or engineer on a product team who has an interest in security and is given extra training and resources by the central security team. They act as the "security conscience" for their team. They help with code reviews, threat modeling, and are the first point of contact for security questions. This creates a much tighter feedback loop than having a separate, centralized security team and helps to foster a true DevSecOps culture.

2.  **Q: What is RASP (Runtime Application Self-Protection)?**

    **A:** RASP is a security technology that is built into an application or its runtime environment and is capable of controlling application execution and detecting and preventing attacks in real time. Unlike a Web Application Firewall (WAF) which sits in front of the application, a RASP agent has the full context of the application's internal data and logic. When it detects a malicious request (like a SQL injection attack), it doesn't just block the request; it can terminate the user's session, log the details of the attack, and alert security personnel, effectively allowing the application to "protect itself."

3.  **Q: How would you design a secure CI/CD pipeline?**

    **A:** A secure pipeline needs security gates at every stage.
    * **Pre-Commit:** Use pre-commit hooks to run linters and secret scanners on the developer's machine before code is even committed.
    * **Commit/PR Stage:**
        * **SAST:** Run static analysis on every pull request.
        * **SCA:** Run dependency scanning to check for vulnerable libraries.
        * **Secret Scanning:** Run a tool to ensure no new secrets have been accidentally committed.
    * **Build Stage:**
        * **Image Scanning:** After building a container image, scan it for OS vulnerabilities.
        * **Sign the Artifact:** Cryptographically sign the build artifact to ensure its integrity.
    * **Test/Staging Stage:**
        * **DAST/IAST:** Run dynamic or interactive security tests against the deployed application in a staging environment.
    * **Production Stage:**
        * **Continuous Monitoring:** Use security monitoring tools in production to detect and respond to threats.
        * **Secure the Pipeline Itself:** The CI/CD system itself must be hardened. Use the principle of least privilege for all pipeline jobs and protect access to the main branch with branch protection rules.

4.  **Q: What is "Policy as Code"?**

    **A:** Policy as Code is the practice of defining your security and compliance policies as code. Instead of having a 100-page Word document that describes your security policies, you write those policies in a machine-readable language.
    * **Example:** You can write a policy that says "No S3 bucket can be public" or "All EC2 instances must have a 'cost-center' tag."
    * **Tools:** The most popular tool for this is **Open Policy Agent (OPA)**.
    * **How it's used:** You can integrate these policy checks into your CI/CD pipeline. For example, when a developer submits a Terraform pull request, the pipeline can use OPA to check if the proposed infrastructure violates any of your policies. If it does, the build fails. This automates compliance and prevents misconfigurations before they ever reach production.

5.  **Q: You are performing a security review of a Kubernetes cluster. What are some of the key things you would look for?**

    **A:** I would look at several layers of the stack.
    * **Control Plane Security:** Is access to the Kubernetes API server locked down? Is RBAC (Role-Based Access Control) being used correctly, following the principle of least privilege? Are the master nodes properly hardened?
    * **Worker Node Security:** Are the worker nodes running a minimal, hardened OS? Is access to the kubelet port restricted?
    * **Network Security:** Are network policies in place to restrict pod-to-pod communication? Is the CNI plugin secure? Is traffic between services encrypted using a service mesh?
    * **Pod Security:** Are Pod Security Policies (or their modern replacement, Pod Security Admission) being used to prevent containers from running as root or mounting sensitive host paths?
    * **Image Security:** Is there a process to ensure that only trusted and scanned container images are allowed to run in the cluster? This can be enforced using an admission controller like OPA Gatekeeper.
    * **Secrets Management:** How are secrets being stored and accessed? Are they encrypted at rest? Is a tool like Vault being used?
    
    A secure cluster requires a defense-in-depth approach that addresses all of these areas.