# HTTP-Request-Smuggling
Practical study of HTTP Request Smuggling — CL.TE, TE.CL, HTTP/1.1 request analysis, validation, evidence, and lessons learned.
# HTTP Request Smuggling — Practical Study & Evidence Walkthrough

> **Objective:** Practically understand how front-end and back-end servers can interpret HTTP requests differently, using an authorized training lab.

---

## 1. Start With the Lab

### Goal

Before testing anything, establish a controlled environment.

Use an intentionally vulnerable lab such as:

* PortSwigger Web Security Academy


### Why this matters

Request smuggling can affect shared infrastructure and other users. Therefore, practical testing should be performed only in an authorized environment.

---

# 2. Understand the Architecture First

Before sending any unusual request, identify the expected request path.

```text
                My Request
                     │
                     ▼
              ┌─────────────┐
              │  Front-end  │
              │ Proxy / WAF │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │  Back-end   │
              │ Application  │
              └─────────────┘
```

### Questions I asked

```text
1. Is there a front-end proxy?
2. Is there a back-end application server?
3. Which HTTP version is being used?
4. How is the request body framed?
5. Does the front-end reuse connections?
```

### Learning point

The vulnerability depends on **multiple HTTP-speaking components** interpreting the same request.

---

# 3. Capture a Normal Request

Before modifying anything, capture a normal request using Burp Suite.

Example:

```http
POST / HTTP/1.1
Host: lab.example
Content-Type: application/x-www-form-urlencoded

parameter=value
```


### What I am learning

The baseline gives me something to compare against later.

Without a baseline, it becomes difficult to determine whether a later response is actually unusual.

---

# 4. Confirm HTTP/1.1

For classic CL.TE / TE.CL research, determine whether the request can be tested using HTTP/1.1.

In Burp, inspect the request protocol.

```text
HTTP/1.1
```



### Why?

HTTP request framing behaves differently across HTTP versions.

For this study, the important mechanisms are:

```text
Content-Length
        +
Transfer-Encoding
```

---

# 5. Learn Content-Length First

Before studying CL.TE, understand what `Content-Length` means.

Conceptually:

```text
Content-Length: N

        N bytes
<-------------------->
[       HTTP body     ]
```

The server uses the declared length to determine where the body ends.

### Study question

> If the server trusts Content-Length, what determines the request boundary?

Answer:

```text
Content-Length
       ↓
Number of bytes
       ↓
Request boundary
```

---

# 6. Learn Transfer-Encoding

Now understand chunked transfer encoding.

Conceptually:

```text
Transfer-Encoding: chunked

Chunk size
    ↓
Chunk data
    ↓
Next chunk
    ↓
...
    ↓
0
    ↓
End
```

### Study question

Ask:

> If the server trusts Transfer-Encoding, what determines the request boundary?

Answer:

```text
Chunked encoding
       ↓
Chunk parsing
       ↓
Request boundary
```

---

# 7. Identify the Interesting Condition

Now compare the two mechanisms.

```text
Content-Length
       │
       ▼
Length-based parsing


Transfer-Encoding
       │
       ▼
Chunk-based parsing
```

The interesting security condition appears when:

```text
Front-end
    │
    │ interprets request one way
    ▼
Back-end
    │
    │ interprets same request differently
    ▼
Different request boundary
```

This is the core concept behind request smuggling.

---

# 8. Study CL.TE

CL.TE means:

```text
CL → Front-end
TE → Back-end
```

Write this in your notes:

```text
CL.TE

Front-end:
Content-Length

Back-end:
Transfer-Encoding
```

### Diagram

```text
                    HTTP Request
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       Front-end parser       Back-end parser
              │                     │
        Uses CL                 Uses TE
              │                     │
              ▼                     ▼
        Boundary A             Boundary B
              │                     │
              └──────────┬──────────┘
                         ▼
                  Compare behavior
```

### What I am trying to learn

I am not simply trying to get an error.

I am asking:

> Does the front-end stop reading the request at a different point from the back-end?

---

# 9. Analyze the Result

After performing the authorized lab test, record what happened.

Create a table:

| Observation        | Result             |
| ------------------ | ------------------ |
| HTTP version       | HTTP/1.1           |
| Front-end behavior | Record observation |
| Back-end behavior  | Record observation |
| Response           | Record response    |
| Timing             | Record timing      |
| Reproducible?      | Yes/No             |
| Security impact    | Record impact      |



# 10. Study TE.CL

Now reverse the parsing roles.

```text
TE.CL

Front-end:
Transfer-Encoding

Back-end:
Content-Length
```

### Diagram

```text
                    HTTP Request
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       Front-end parser       Back-end parser
              │                     │
        Uses TE                 Uses CL
              │                     │
              ▼                     ▼
        Boundary A             Boundary B
              │                     │
              └──────────┬──────────┘
                         ▼
                  Compare behavior
```

### Study question

Ask:

> What changes when the front-end uses chunked parsing but the back-end uses Content-Length?

This comparison is more useful than memorizing the abbreviation.

---

# 11. Compare CL.TE and TE.CL

Now create your own comparison.

| Question              | CL.TE             | TE.CL             |
| --------------------- | ----------------- | ----------------- |
| Front-end uses        | Content-Length    | Transfer-Encoding |
| Back-end uses         | Transfer-Encoding | Content-Length    |
| Parser disagreement   | Possible          | Possible          |
| HTTP/1.1 relevance    | High              | High              |
| Main thing to observe | Request boundary  | Request boundary  |

### Memory trick

```text
CL.TE

First  = Front-end
Second = Back-end

CL → Front-end
TE → Back-end
```

Therefore:

```text
TE.CL

TE → Front-end
CL → Back-end
```

---

# 12. Use Burp to Observe, Not Just Attack

Burp Suite should be used as an observation and testing tool.

Focus on:

```text
Proxy
   ↓
HTTP history
   ↓
Repeater
   ↓
HTTP version
   ↓
Headers
   ↓
Request body
   ↓
Response
```





# 14. Validation

A finding should not be based on one strange response.

Use:

```text
Initial observation
        ↓
Repeat test
        ↓
Same behavior?
        ↓
Compare baseline
        ↓
Understand parser difference
        ↓
Determine impact
        ↓
Document finding
```



---

# 15. Write the Finding

Instead of writing:

> "HTTP Request Smuggling found."

Write something more professional:

### Finding

**HTTP Request Smuggling — CL.TE**

### Observation

The front-end and back-end components demonstrated different request-framing behavior when processing the test request.



### Impact

Depending on the application architecture, request smuggling may allow an attacker to interfere with subsequent requests or bypass security controls.

### Remediation

Ensure front-end and back-end components use consistent and secure HTTP request parsing behavior.

---

# 16. Lessons Learned

Write your own observations here after completing the lab.

```text
### What I learned

1. __________________________________

2. __________________________________

3. __________________________________

4. __________________________________

5. __________________________________
```

This is important because it demonstrates that you actually understood the lab.

---

# 17. My Evidence Flow

This should become the main diagram in the GitHub README.

```text
┌──────────────┐
│   Objective  │
└──────┬───────┘
       ↓
┌──────────────┐
│ Lab Setup    │
└──────┬───────┘
       ↓
┌──────────────┐
│ Baseline     │
│ Request      │
└──────┬───────┘
       ↓
┌──────────────┐
│ HTTP/1.1     │
│ Analysis     │
└──────┬───────┘
       ↓
┌────────────────────┐
│ Request Framing    │
│ CL / TE            │
└─────────┬──────────┘
          ↓
   ┌──────┴──────┐
   ↓             ↓
 CL.TE          TE.CL
   ↓             ↓
   └──────┬──────┘
          ↓
┌────────────────────┐
│ Compare Behavior   │
└─────────┬──────────┘
          ↓
┌────────────────────┐
│ Validation         │
└─────────┬──────────┘
          ↓
┌────────────────────┐
│ Evidence           │
└─────────┬──────────┘
          ↓
┌────────────────────┐
│ Lessons Learned    │
└────────────────────┘
```

---



# 19. The Portfolio Principle

Do not make the repository look like:

```text
"I know HTTP Request Smuggling."
```

Make it look like:

```text
I studied the architecture
        ↓
I captured the baseline
        ↓
I analyzed HTTP/1.1
        ↓
I studied CL.TE
        ↓
I studied TE.CL
        ↓
I compared parser behavior
        ↓
I validated the observation
        ↓
I documented evidence
        ↓
I understood the security impact
```

That demonstrates **practical cybersecurity thinking**, not just memorization.

---

## Final Study Formula

Remember this:

```text
HTTP Request Smuggling

        ↓

Two HTTP components

        ↓

Different parsing rules

        ↓

Different request boundary

        ↓

Unexpected interpretation

        ↓

Validate

        ↓

Collect evidence

        ↓

Explain impact

        ↓

Recommend remediation
```

