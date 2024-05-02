1. What methods of authentication are supported?
2. Is caching used?
3. Are there any odd or unknown headers?
4. How is CSRF protection implemented?
5. Is there any cross-domain communication which could affect CORS and postMessage?
6. What file types are supported in file uploads and where are the files getting stored?
7. Which requests involve a database query?
8. Which features may do a request to other internal or external applications
9. Is there a reverse proxy that routes traffic to internal microservices?
10. Are there any template engines used?
11. Are there any data imports and exports that could result in an SSRF or other vulnerability?
12. Is XML used anywhere that could result in an XXE?
13. Are IaC tools like Terraform or Ansible potentially used? IaC, which stands for Infrastructure as Code, is a practice used by developers and DevOps teams to define their infrastructure in code such as VMs, storage buckets, VPCs, etc and to deploy it in an automated way.
14. What HTTP version is used? Is there potential for desync attacks?
15. How does the communication work within the web app? Is there a REST API? Is GraphQL or maybe gRPC used?
16. Are there any integrations to external applications? Or, are there any webhooks or external communication done with email?
17. How does the password reset flow work?
18. Is 2FA supported?
19. Are there any technologies used running on old and vulnerable versions which have known CVEs and exploits?
