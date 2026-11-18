# Step-Up Authentication Samples

Reference implementations demonstrating how APIs can require stronger or more recent authentication for sensitive operations without requiring that level of authentication for every request.

This repository demonstrates the same step-up authentication scenario in:

- C#
- Go
- Rust
- Python

Each implementation follows the same behavioral model while using patterns and libraries appropriate to that language.

## The Problem

Not every operation performed by an application has the same security requirements.

Consider an API that exposes two operations:

- `GetRecord("123")`
- `DeleteRecord("123")`

Reading a record may only require the user to be authenticated.

Deleting the record is more sensitive. The application may require the user to authenticate using a stronger authentication method, such as multi-factor authentication (MFA), before allowing the operation.

Requiring MFA when the user initially signs in would work, but it requires stronger authentication even when the user only performs low-risk operations.

Instead, the application can require stronger authentication only when it is actually needed.

This is **step-up authentication**.

## The Solution

The protected resource determines the authentication requirements for an operation.

A normal request might look like:

    Client
       |
       | access token
       v
    Get Record
       |
       v
    Authentication requirement satisfied
       |
       v
    "Record 123 was retrieved."

A sensitive operation may require stronger authentication:

    Client
       |
       | access token
       v
    Delete Record
       |
       v
    Authentication requirement not satisfied
       |
       v
    Step-up authentication required

The client can then authenticate again using the required authentication strength and retry the operation.

    Client
       |
       v
    Authorization Server
       |
       | stronger authentication
       v
    New access token
       |
       v
    Delete Record
       |
       v
    Authentication requirement satisfied
       |
       v
    "Record 123 was deleted."

The important distinction is that the **operation determines the required authentication assurance**.

The client does not decide that deleting a record requires MFA.

The protected resource does.

## OAuth 2.0 Step-Up Authentication

The samples are based on **RFC 9470: OAuth 2.0 Step Up Authentication Challenge Protocol**.

RFC 9470 addresses situations where a resource server requires a different authentication strength or more recent authentication based on the characteristics of a request.

When the authentication associated with an access token does not satisfy those requirements, the resource server can signal that stronger or more recent authentication is necessary.

This enables applications to enforce stronger authentication only for the operations that require it.

## Example Scenario

Every language implementation uses the same example.

### Get a Record

    GET /records/123

Result:

    Record 123 was retrieved.

This operation requires normal authentication.

### Delete a Record

    DELETE /records/123

Deleting a record is considered a sensitive operation.

It therefore requires the `SensitiveAction` policy.

When the current authentication satisfies that policy:

    Record 123 was deleted.

When the current authentication does not satisfy the policy, the operation is not performed.

Instead, the resource indicates that step-up authentication is required.

## Policy-Based Authentication Requirements

The implementations use policies to separate an operation from the details of how its authentication requirements are evaluated.

Conceptually:

    Delete Record
           |
           v
    SensitiveAction Policy
           |
           +---- authentication strength
           |
           +---- authentication recency
           |
           v
    Policy Evaluation

Application code identifies the policy protecting an operation.

The policy determines what authentication characteristics are required.

This separation prevents application code from containing authentication-specific logic.

Instead of writing logic such as:

    if user used MFA
        delete record

the operation declares:

    requires SensitiveAction

The authentication infrastructure determines what `SensitiveAction` requires and whether the current authentication satisfies those requirements.

## Authentication Strength

A policy may require a particular authentication context.

For example:

    SensitiveAction

    requires:
        authentication context = mfa

A token representing ordinary authentication would not satisfy that policy.

A token representing an authentication event satisfying the required context would.

## Authentication Recency

Authentication strength is not the only consideration.

A sensitive operation may also require that authentication occurred recently.

Conceptually:

    SensitiveAction

    requires:
        authentication context = mfa
        authentication age <= configured limit

This allows an application to require the user to authenticate again when the previous authentication event is too old, even if that authentication originally satisfied the required authentication strength.

## Why This Matters for Agents

The same pattern becomes particularly useful when APIs are invoked by AI agents.

An agent may be able to autonomously perform many routine operations on behalf of a user.

A sensitive operation should not automatically inherit that same level of trust.

For example:

    User
      |
      v
    Agent
      |
      +---- GetRecord
      |        |
      |        +---- allowed
      |
      +---- DeleteRecord
               |
               +---- stronger authentication required

The agent attempts the operation normally.

The protected resource determines that the authentication associated with the request is insufficient.

The sensitive operation is not performed.

The client can then involve the user in satisfying the additional authentication requirement before retrying the operation.

The agent therefore does not need to decide which operations require MFA or another stronger authentication mechanism.

That decision remains with the resource protecting the operation.

## Mock Authentication

The samples do not require a real identity provider.

Instead, they simulate authentication results so the repository can focus on policy evaluation and step-up behavior.

Two authentication states are used:

    password

and:

    mfa

The `password` identity represents an authenticated user whose authentication does not satisfy the sensitive-action policy.

The `mfa` identity represents an authenticated user whose authentication does satisfy the policy.

These authentication mechanisms are intentionally fake and exist only for demonstration and testing.

They must not be treated as production authentication mechanisms.

A production application would receive authentication information from its configured identity and authorization infrastructure.

## Repository Structure

The repository contains equivalent implementations in four languages.

    .
    ├── csharp/
    ├── go/
    ├── rust/
    └── python/

Each implementation demonstrates the same behavior while using conventions appropriate to that language and ecosystem.

The implementations are not intended to be line-for-line translations of one another.

### C#

The C# implementation demonstrates:

- ASP.NET Core Controllers
- ASP.NET Core Minimal APIs
- MCP tools
- ASP.NET Core policy-based authorization

### Go

The Go implementation demonstrates the same step-up authentication behavior using idiomatic Go HTTP and MCP patterns.

### Rust

The Rust implementation demonstrates the same step-up authentication behavior using idiomatic Rust HTTP and MCP patterns.

### Python

The Python implementation demonstrates the same step-up authentication behavior using idiomatic Python web and MCP patterns.

## Common Behavior

Regardless of language or framework, the samples demonstrate the same logical operations.

### Normal Operation

    GetRecord("123")

returns:

    Record 123 was retrieved.

### Sensitive Operation

    DeleteRecord("123")

requires:

    SensitiveAction

and, after successful policy evaluation, returns:

    Record 123 was deleted.

Keeping the business behavior intentionally simple makes it easier to compare the authentication implementations across languages.

## Authentication vs. Authorization

Step-up authentication does not replace authorization.

They answer different questions.

**Authorization**

> Is this caller allowed to perform this operation?

**Authentication assurance**

> Is the authentication associated with this caller sufficient for this operation?

A caller that is not authorized to delete a record remains unauthorized regardless of how strongly or recently the caller authenticated.

Successful step-up authentication must not grant permissions that the caller did not already have.

## Repository Goals

This project demonstrates:

- Step-up authentication for sensitive operations
- Policy-based authentication requirements
- Authentication strength requirements
- Authentication recency requirements
- Consistent enforcement across different endpoint styles
- Step-up authentication for APIs invoked by agents
- MCP tool protection
- Equivalent implementations across multiple programming languages

## Non-Goals

This repository is not intended to:

- Implement an identity provider
- Implement MFA
- Replace OAuth or OpenID Connect libraries
- Introduce a new authorization framework
- Provide production user management
- Implement application-specific permissions
- Demonstrate complex business logic

The samples build on the authentication and authorization capabilities available in each language ecosystem.

## Standards

The primary protocol demonstrated by this repository is:

**RFC 9470: OAuth 2.0 Step Up Authentication Challenge Protocol**

RFC 9470 defines a mechanism for resource servers to indicate that the authentication event associated with an access token does not satisfy the authentication requirements of the current request and to communicate requirements for stronger or more recent authentication.

## Status

This repository is a reference and learning project.

The implementations prioritize clarity, interoperability concepts, and comparison between languages over production infrastructure.