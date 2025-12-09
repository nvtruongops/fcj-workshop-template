---
title: "Week 11 Worklog"
date: 2025-11-17
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives
* Start integrating frontend with backend authenticated APIs.
* Strengthen API Gateway and Lambda error handling strategy.
* Implement frontend token storage and refresh token logic.
* Build reusable API client wrapper (Axios/Fetch) with interceptors.
* Improve logging, debugging and CloudWatch insights usage.
* Complete full-stack functionality from UI → Gateway → Lambda → Database.

---

### Tasks to be carried out this week

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|-------------|------------------|--------------------|
| 2 | Connect frontend to Cognito auth endpoints. <br>- Implement login, signup UI flows. | 17/11/2025 | 17/11/2025 | AWS Study Group |
| 3 | Implement secure token storage & token refresh mechanism. <br>- Add interceptors in API client. | 18/11/2025 | 18/11/2025 | AWS Study Group |
| 4 | Integrate frontend with API Gateway authenticated routes. <br>- Validate Lambda Authorizer flow. | 19/11/2025 | 19/11/2025 | AWS Study Group |
| 5 | Build a global error handling layer (frontend + backend). <br>- Improve API response consistency. | 20/11/2025 | 20/11/2025 | AWS Study Group |
| 6 | Strengthen monitoring using CloudWatch Logs & Metrics. <br>- Debug end-to-end flow. | 21/11/2025 | 21/11/2025 | AWS Study Group |

---

### Week 11 Achievements

#### **1. Completed Frontend Authentication Integration**
This week, I successfully connected the frontend application to Cognito, enabling user signup, login, and logout directly from the UI. I ensured token retrieval and user session management work properly.

#### **2. Implemented Secure Token Storage & Auto Refresh**
I set up secure token storage using browser memory and added an automatic token refresh flow. I also implemented Axios/Fetch interceptors to attach access tokens on every request.

#### **3. Integrated Frontend with Protected Backend APIs**
The frontend can now communicate with protected API Gateway routes using JWT tokens. This validates the entire authentication pipeline from UI → Client → Gateway → Lambda → DynamoDB.

#### **4. Built a Consistent Global Error-Handling System**
I standardized error responses in Lambda, updated API Gateway mappings, and built a global error handler on the frontend to show meaningful messages. This reduces debugging time and improves UX.

#### **5. Improved Logging & Monitoring with CloudWatch**
I analyzed Lambda logs, identified slow operations, and created CloudWatch Metrics filters to monitor request failures and latency. This improved my debugging workflow significantly.

#### **6. Achieved Full-Stack Authentication + Data Flow**
By the end of the week, I successfully executed end-to-end user actions:
UI → Authenticate → Call API → Lambda Logic → DynamoDB → Return response.  
Everything worked seamlessly with proper security & error-handling in place.

---
