# tracehound

API contracts are lies until tested.

Most developers treat an OpenAPI schema as the source of truth for how a system behaves. It isn't. It is a wish list. Tracehound is an adversarial testing tool that turns your API documentation into a series of stress tests designed to expose implementation drift.

Standard QA suites focus on the happy path. They verify that the ID field returns a 200 OK when given a valid integer. They rarely check what happens when that ID is a null byte or a 5GB string. Tracehound closes the gap between what your documentation claims and what your code actually handles.

### How it works

Tracehound consumes a `swagger.json` or `openapi.yaml` file and maps every endpoint, parameter, and header. It then generates malformed requests based on four specific failure vectors:

*   **Type Contradiction:** Sending boolean values where strings are expected or floating-point numbers into integer fields.
*   **Boundary Violations:** Testing the extreme edges of defined minimums and maximums.
*   **Auth Stripping:** Identifying which "required" authentication headers can be omitted without triggering a 401.
*   **Schema Drift:** Flagging instances where the API returns a 500 Internal Server Error instead of a documented 400 Bad Request.

If your API crashes because of a malformed JSON body, your schema isn't just inaccurate. It is a security risk.

### MVP Features

The core tool handles the heavy lifting of request generation and execution.

1.  **Schema Parser:** Deeply inspects nested objects and recursive definitions within the spec.
2.  **Adversarial Fuzzer:** Builds a library of payloads specifically tuned to trigger unhandled exceptions.
3.  **Request Replay:** Executes the generated tests against a live staging environment.
4.  **Drift Report:** Generates a list of endpoints where actual behavior contradicts the documentation.

### Usage

Tracehound is built to run in a CI pipeline. You point it at your local schema and your development endpoint.

```bash
python3 tracehound.py --schema ./api-spec.yaml --target https://api.dev.local
```

### Why this exists

I built Tracehound because "production-ready" often just means "it works for the frontend team." Systems fail at the edges. By automating the generation of edge-case requests, you find the 500 errors before an attacker or a buggy client library finds them for you. 

Does your API handle a 64-bit integer in a 32-bit field? Let's find out.
