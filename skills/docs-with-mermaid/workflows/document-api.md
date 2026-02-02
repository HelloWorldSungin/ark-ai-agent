<required_reading>
Before starting, read these references as needed:
- `references/documentation-types.md` - for API doc best practices
- `references/mermaid-syntax.md` - sections on sequence diagrams, ER diagrams
- `references/common-patterns.md` - for request/response flow patterns
</required_reading>

<objective>
Create API documentation with sequence diagrams showing request lifecycles, data models, and error handling flows.
</objective>

<process>
1. **Identify API scope**
   - List all endpoints or the specific endpoints to document
   - Identify authentication/authorization flow
   - Map data models used in requests and responses

2. **Create the authentication flow diagram**
   - Sequence diagram showing auth handshake
   - Include token refresh if applicable
   - Show error paths (invalid credentials, expired tokens)

3. **Create endpoint sequence diagrams**
   - For each major endpoint, show the full request lifecycle
   - Include all services involved (gateway, auth, business logic, database)
   - Use `autonumber` for step references
   - Show `alt` blocks for success/error paths
   - Add `Note` blocks for non-obvious logic

4. **Create data model diagram**
   - ER diagram showing entities and relationships
   - Include attribute types and keys (PK, FK, UK)
   - Show cardinality (one-to-many, many-to-many)

5. **Create error handling flow** (if complex)
   - Flowchart showing error classification and handling
   - Map error types to HTTP status codes

6. **Write the documentation**
   - Use the template from `templates/api-doc.md`
   - Document each endpoint with: purpose, parameters, request/response examples
   - Reference the sequence diagram for the flow
   - Include error response documentation

7. **Validate**
   - Run through `templates/quality-checklist.md`
   - Verify all endpoints are covered
   - Check sequence diagram accuracy against actual code
</process>

<success_criteria>
- Authentication flow is diagrammed
- Major endpoints have sequence diagrams
- Data model is documented with ER diagram
- Error handling is documented
- Request/response examples are included
</success_criteria>
