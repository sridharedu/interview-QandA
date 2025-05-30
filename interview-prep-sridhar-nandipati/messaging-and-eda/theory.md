3. Inside each topic folder, include a `theory.md` file with the following characteristics:

    *   **Purpose**: To guide an experienced engineer (10+ years) in preparing for in-depth senior Java engineer/architect technical interviews.

    *   **Structure and Formatting**:
        *   Organize the content using Markdown `##` (H2) headings for major sections. For example, use headings like `## Core Concepts`, `## Key Principles`, `## Advanced Topics`, `## Real-world Applications`, `## Common Pitfalls`, or other relevant section titles like `## Overview` and `## Deep Dive`.

    *   **Content Style**:
        *   Write in a clear, concise, and interview-focused style.
        *   Prioritize practical, real-world understanding and application of concepts over dry, textbook definitions. The content should resonate with an experienced engineer and be directly applicable to senior Java engineer/architect interview scenarios.
        *   Cover topics comprehensively, from foundational basics to advanced aspects, ensuring depth where necessary for in-depth technical discussions.

    *   **Placeholders for Customization**:
        *   Embed `<!-- TODO: ... -->` style inline comments throughout the content. These are crucial placeholders where specific concepts, detailed explanations, Sridhar's personal project examples, or further elaborations should be added by the user. Make these TODOs actionable and specific.
        *   **Personalization with Sridhar's Experience**: Especially within `<!-- TODO: ... -->` comments, prompt the user to incorporate examples and learnings from their specific background. Key areas from Sridhar's resume include: high-performance asynchronous systems (Vert.x at BOFA), event-driven architectures with Kafka (TOPS data pipelines), AEM component development, microservices on AWS, CI/CD automation, and observability with ELK/Prometheus. The generated content should guide Sridhar to reflect on and insert these experiences.
        *   **Examples of effective TODO comments to guide Sridhar**:
            *   `<!-- TODO: Reflect on the high-throughput processing challenges with Vert.x at BOFA. How were non-blocking asynchronous principles applied? What were the key performance wins or lessons learned? -->`
            *   `<!-- TODO: Detail the Kafka integration with Debezium for PostgreSQL in the TOPS project. What were the primary use cases for real-time data streaming, and what considerations were made for message ordering or error handling? -->`
            *   `<!-- TODO: Describe a complex custom AEM component or workflow you developed for Herc Rentals. What problem did it solve, and what were the key AEM APIs or practices leveraged? -->`
