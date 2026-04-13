# Pratik Chandlekar
Software Engineer | Mumbai, India  
+91 9075467719 • pratikchandlekar280@gmail.com • 
[LinkedIn](https://www.linkedin.com/in/pratik280) • [GitHub](https://github.com/Pratik280) • 
[Portfolio](https://pratik280.hashnode.dev/page/about) • [Blog](https://pratik280.hashnode.dev/)


## Summary

Software Engineer with ~4 years of experience building scalable microservices and event-driven systems using Java, Spring Boot, and Apache Kafka. Experienced in designing secure, high-performance REST APIs, multitenant architectures, and real-time transaction processing systems. Contributed to system design, code reviews, and delivery of reliable distributed systems, with exposure to cloud-native environments and modern deployment practices.

## Experience
<div class="experience">

**Software Engineer — In-Solution Global | Mumbai**  
Jun 2022 – Present

### Projects & Contributions:

- **Designed and built a distributed payment processing platform across multiple Spring Boot microservices** — covering payment link creation, transaction management, and query APIs — using an event-driven Kafka pipeline where each service reacts independently to transaction events; multi-step payment workflows are coordinated via the Saga choreography pattern to maintain data consistency, and the Outbox pattern ensuring no events are lost between the database and Kafka.

- **Architected a multi-tenant data layer supporting multiple clients on a single application instance** — isolating each client's data at the database schema level using Hibernate JPA; evaluated and chose schema-based multitenancy over database sharding after analyzing operational trade-offs.

- **Built an internal monitoring platform aggregating real-time health and transaction data across multiple datacenters** — polling service health via Actuator, tracking Kafka consumer lag, and consuming transaction events to expose success rates and failure codes through a single dashboard API; data is collected via scheduled aggregation and cached in Redis, separating collection from serving for fast, scalable responses.

- **Led the migration of the Kafka messaging infrastructure from a legacy ZooKeeper-based dual-cluster setup to a unified KRaft-mode 6-broker cluster spanning two datacenters** — configured rack-aware replication, replication factor 6 with min.insync.replicas=4, and DNS-based client routing, ensuring zero message loss even if one datacenter goes down.

- **Designed and deployed the API gateway layer for the microservices platform using a 3-node Spring Cloud Gateway cluster** — handling request routing, RBAC token validation via custom filters, and Eureka-based service discovery, ensuring new services can be onboarded without any changes to the client-facing entry point.

- **Partnered with DevOps to build a fully automated end-to-end CI/CD pipeline triggered by GitLab tags** — on trigger, the pipeline spins up all microservices with their dependencies (Kafka, Redis, PostgreSQL) using Docker Compose, runs the full integration test suite, and only on success builds and pushes Docker images to the registry, updates Helm chart values in the repository, which triggers Argo CD to automatically deploy the updated services to the Kubernetes cluster; Prometheus and Grafana dashboards provide continuous observability post-deployment.

- Mentored junior engineers through code reviews and design discussions, contributed to architectural decisions and actively participated in sprint planning and retrospectives within an Agile/Scrum delivery model.

</div>

<div class="skills">

## Skills



- **Languages:** Java, JavaScript, TypeScript, SQL
- **Backend:** Spring Framework (Spring Boot, Spring MVC, Spring Cloud, Spring Security), Hibernate/JPA, REST APIs, Maven, JSON, HTTP
- **Cloud & DevOps:** AWS(EC2, S3, IAM, Lambda), Azure, Docker, Kubernetes (AKS), CI/CD (Jenkins), Helm, Nexus, Nginx
- **Databases & Cache:** PostgreSQL, MySQL, Redis
- **Messaging & Streaming:** Apache Kafka (KRaft), Event-driven architecture, Saga Pattern (Choreography), Publisher–Subscriber patterns, Message partitioning, Consumer groups
- **Core Java:** Object-Oriented Programming (OOP), Collections, Exception Handling, Multithreading & Concurrency
- **Testing**: JUnit, Unit Testing, Integration Testing
- **Observability:** ELK Stack (Elasticsearch, Logstash, Kibana), Prometheus, Grafana
- **OS & Tools** : Linux, Bash, Git, Claude, ChatGPT, Copilot
- **Practices** : Agile/Scrum, Software Development Life Cycle (SDLC)

</div>


## Certifications
- AWS Certified Cloud Practitioner — Jun 2024 (Valid till Jun 2027) — [Credential](https://www.credly.com/badges/bee394d0-95b9-492f-bf05-17682aa871f6/linked_in_profile)


## Education
**B.E. Computer Science — Mumbai University**  
2018 – 2022 | CGPA: 8.68