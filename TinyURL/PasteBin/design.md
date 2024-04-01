# System Design Requirements for URL Shortening Service

## Functional Requirements

1. **Generate Short URLs:** The system must generate a unique, short URL for a given long URL.
2. **URL Redirection:** Upon accessing a short URL, the system should redirect the user to the original long URL.
3. **Analytics Tracking:** Track the number of clicks per short URL for analytics purposes.

## Non-Functional Requirements

1. **High Availability:** The system must be highly available, ensuring that short URLs are accessible at all times.
2. **Scalability:** Must be able to support a large number of URL shortenings and redirects (e.g., a trillion URLs, with median URL receiving 10,000 clicks).
3. **Performance:** System must offer fast response times for both creating short URLs and redirecting to long URLs.
4. **Consistency:** Click analytics should eventually be consistent, ensuring accuracy in the number of clicks reported.
5. **Durability:** Analytics data should be durable and not lost in case of system failures.
