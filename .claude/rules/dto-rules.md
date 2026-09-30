---
paths:
  - "src/main/java/**/dto/**"
---

You are editing DTO code: request and response payloads.

DTOs are Java records — no Lombok, no classes with setters.
NEVER import org.springframework.*
NEVER import jakarta.persistence.*
Bean Validation annotations (jakarta.validation.*) ARE allowed.
No business logic — DTOs carry data, nothing else.
NEVER expose JPA entities from model/ as a request or response type.
BigDecimal for ALL monetary fields, scale 2.
Test with plain JUnit + AssertJ.
