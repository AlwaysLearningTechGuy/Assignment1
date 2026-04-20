# Technical Design: Web App Project (Updated)

## 1. Project Selection
**Project Name:** The Wonderful and Mysterious Web Application

**Reasoning/Motivation:** I chose this for my project because it provides me an opportunity to learn more about the incorpation of software development tools (such as GitHub and Docker). Additionally, it will of course provide the opportunity to create software development artifacts such as AI tests (see below), etc.

## 2. Summary
The Wonderful and Mysterious Web Application is a containerized Python microservice built using the FastAPI framework. It serves as a mock environment for testing client-server communication via RESTful APIs. The application balances high-utility "Wonderful" features (Weather, Timezone, and Insights) with "Mysterious" features (Randomized fortunes and user-saved collections) to provide a diverse range of JSON data structures for frontend consumption.

## 3. System Architecture
The application utilizes a modern, asynchronous architecture designed for scalability and ease of deployment.

* **Logic Layer:** Python 3.11 with FastAPI for high-performance, asynchronous routing.
* **Data Serialization:** Pydantic is utilized for strict JSON schema enforcement and validation.
* **Persistence Layer:** A simulated in-memory data store for tracking user "Favorites."
* **Containerization:** The application is packaged via **Docker**, ensuring consistency across development, testing, and production environments.

## 4. API Contract Specifications

### 4.1 The "Wonderful" Endpoints (Utility)
| Endpoint | Method | Input | Expected Output |
| :--- | :--- | :--- | :--- |
| `/api/weather` | `GET` | `city` (string) | `{"city": "string", "temp": "22°C", "condition": "Sunny"}` |
| `/api/timezone`| `GET` | `offset` (int) | `{"timezone": "GMT+X", "local_time": "ISO-8601"}` |
| `/api/insight` | `GET` | `topic` (string) | `{"id": 101, "msg": "Strategic data point"}` |

### 4.2 The "Mysterious" Endpoints (Trivial & Stateful)
| Endpoint | Method | Input | Expected Output |
| :--- | :--- | :--- | :--- |
| `/api/fortune`  | `GET` | None | `{"id": 1, "msg": "A randomized fortune string"}` |
| `/api/submit`   | `POST` | `payload` (JSON) | `{"status": "Created", "received": {echo_of_payload}}` |
| `/api/favorites`| `POST` | `{"id": int}` | `201 Created` + Current list of Saved IDs |
| `/api/favorites`| `GET` | None | `200 OK` + List of all saved items |

---

## 5. Test Specifications and Critique

### 5.1 Legacy Test Suite (Original AI Suggestions)
| ID | Type | Scenario | Input / Method | Expected Result |
| :--- | :--- | :--- | :--- | :--- |
| **TC-01** | Functional | Theme Relevance | `GET /api/fortune` | `200 OK` + Fortune-themed message |
| **TC-02** | Functional | Data Submission | `POST /api/submit` | `201 Created` |
| **TC-03** | Functional | Schema Integrity | `GET /api/insight` | `200 OK` + Keys: `id`, `msg`, `ts` |
| **TC-04** | Boundary | Payload Limit | `POST` (1MB+ String) | `413 Payload Too Large` |
| **TC-05** | Boundary | Syntax Error | `POST` (Invalid JSON) | `400 Bad Request` |

### 5.2 Revised & Expanded Test Suite (Current Version)
The following tests define the current validation layer, incorporating refinements for testability and security:

| ID | Scenario | Method / Endpoint | Expected Result | Note on Change |
| :--- | :--- | :--- | :--- | :--- |
| **TC-01** | Connectivity | `GET /api/fortune` | `200 OK` + Non-empty message | Changed from "relevance" to objective test |
| **TC-02** | Echo Validation | `POST /api/submit` | `201 Created` + Mirrored Input | Added data integrity check |
| **TC-03** | Schema Check | `GET /api/insight` | `200 OK` + Keys: `id`, `msg` | Removed unnecessary `ts` key |
| **TC-04** | Payload Boundary| `POST /api/submit` | `413 Payload Too Large` | Retained from original design |
| **TC-05** | Syntax Validation| `POST /api/submit` | `400 Bad Request` | Retained from original design |
| **TC-06** | Rate Limiting | `GET /api/fortune` | `429 Too Many Requests` | New: Added to prevent API abuse |
| **TC-07** | Performance | `GET /api/weather` | Response time < 200ms | New: Added for operational benchmark |
| **TC-08** | Save Logic | `POST /api/favorites`| `201 Created` + Saved List | New: Added for stateful persistence |
| **TC-09** | Retrieval Logic | `GET /api/favorites` | `200 OK` + Array of items | New: Added for data retrieval |

### 5.3 Critique of AI-Generated Test Cases
The following is my original critique and assessment of the initial test cases (TC-01 through TC-05) provided by the AI:

* TC-01 created a functional test about theme relevance to the endpoint. I have no idea how one would test for relevance. I would likey change this test to simply test if a non-empty reply was returned to the API call so the feature could be actually tested.
* TC-02 appears to only test that a submission occured. To improve this limited test, I would modify this test to repeat back what was submitted so the user can see what was actually received.
* TC-03 having a standard scheme to test seems appropriate based on the limited knowledge I have on web apps. I believe the key ts it refences in the test is for timestamp, which for this program I do not see the need for, so I would remove it.
* TC-04 verifying that the size error of the string being too large is caught and returned seems appropriate and is indeed testable. No changes.
* TC-05 verifying that if JSON syntax is improper, that a standard error is sent back to the user so that they can fix it seems appropriate and indeed something that can be tested through fuzzing.
* TC-06 through TC-09 did not originally exist without some additional prompting of AI.
* Added TC-06 to incorporate a foundational security test to ensure availability.
* Added TC-07 through TC-09 based on the expanded features identified that should be included in this web application. These additional features needed functional tests.

---

## 5. Security & Error Handling
* **Structured Error Responses:** All failures return a JSON object with a human-readable detail message.
* **HTTP Status Compliance:** 200/201 (Success), 400 (Bad Request), 413 (Large Payload), 429 (Rate Limit).
* **Input Sanitization:** Automated type-checking via Pydantic to prevent malformed data from reaching core logic.

## 6. Deployment Instructions
1.  **Build the Image:** `docker build -t mysterious-web-app .`
2.  **Launch Container:** `docker run -d -p 8000:8000 mysterious-web-app`
3.  **Documentation:** Access the auto-generated Swagger UI at `http://localhost:8000/docs`.
