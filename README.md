# AttendX: Enterprise-Scale Attendance Management System

## Overview

**AttendX** is a cloud-native, high-concurrency attendance management platform designed for universities and large educational institutions. It is engineered to reliably handle thousands of simultaneous student check-ins, ensuring data integrity, zero data loss, and seamless user experience for both students and faculty.

---

## System Architecture

### 1. Frontend

- **Framework:** React (Vite)
- **Styling:** Tailwind CSS, Shadcn UI
- **State Management:** Context API (`AuthContext`, `NotificationContext`)
- **Performance Optimizations:**
  - Debounced polling (10–20s intervals) to minimize server load
  - Disabled critical buttons after submission to prevent duplicate requests
  - Feature-flagged notifications to reduce unnecessary background traffic

### 2. Backend

- **Framework:** FastAPI (Python, fully async)
- **Database Access:** SQLAlchemy (async) with PostgreSQL (AWS RDS/Neon)
- **API Gateway:** AWS API Gateway (Lambda Proxy Integration)
- **Serverless Compute:** AWS Lambda (Dockerized FastAPI app)
- **Queueing:** AWS SQS for burst traffic absorption and asynchronous processing
- **Dead Letter Queue (DLQ):** Configured for SQS to guarantee zero data loss—any failed attendance submission is automatically retried or captured for manual review

### 3. Database Schema

- **Users:** Authentication and profile data
- **Classes:** Faculty-course mapping
- **ClassEnrollments:** Many-to-many mapping of students to classes
- **AttendanceSessions:** Represents each live class period
- **AttendanceRecords:** Core ledger for student presence
- **Notifications:** Feature-flagged for performance

---

## Architecture Diagram
<img width="991" height="542" alt="Attendx-AWS-Architecture" src="https://github.com/user-attachments/assets/a6f01625-ba2c-4a45-9305-3e0f06b09d19" />


## Scalability & Reliability

- **Proven Throughput:**  
  AttendX has been load-tested using [Locust](https://locust.io/) to handle **10,000 requests per second** with a **0% failure rate**.
- **Zero Data Loss:**  
  All critical write operations (such as attendance submissions) are routed through SQS.  
  If a Lambda invocation fails, the message is automatically retried.  
  Any message that cannot be processed after the maximum retry count is moved to a **Dead Letter Queue (DLQ)** for guaranteed durability and manual intervention.
- **Client-Side Throttling:**  
  The frontend disables critical buttons after submission and uses debounced polling to minimize accidental duplicate requests.
- **Feature Flags:**  
  Non-essential features (like notifications) are programmatically disabled during high-traffic periods to ensure core attendance processing is never delayed.

---

## CI/CD Pipeline

- **GitHub Actions:**  
  Automated CI/CD pipeline using GitHub Actions with OIDC authentication to AWS.
- **Dockerized Deployments:**  
  Each push to the main branch triggers a Docker build of the FastAPI backend, which is pushed to Amazon ECR.
- **Automated Lambda Updates:**  
  The pipeline updates the AWS Lambda function to use the latest Docker image, ensuring zero-downtime deployments.
- **Security:**  
  Environment variables and secrets are managed via AWS Lambda configuration, never stored in the repository.

---

## Key Metrics

- **10,000 requests/sec** sustained throughput (Locust load test)
- **0% failure rate** under maximum load
- **0 data loss** due to SQS + DLQ architecture
- **Zero-downtime deployments** via automated CI/CD

---

## How It Works

1. **Student Attendance Submission:**  
   Students submit attendance codes via the frontend. These requests are routed through API Gateway and placed in an SQS queue, ensuring the backend can absorb massive spikes in traffic.
2. **Asynchronous Processing:**  
   AWS Lambda functions process attendance submissions from the queue, perform validation (including geolocation if enabled), and write results to the database.
3. **Dead Letter Queue:**  
   If any attendance submission fails repeatedly, it is moved to a DLQ for manual review, ensuring no data is ever lost.
4. **Faculty Dashboard:**  
   Faculty can view real-time attendance, manage sessions, and export data. Reads are optimized via read replicas and API caching.
5. **CI/CD:**  
   Every code push triggers a secure, automated pipeline that builds, tests, and deploys the backend with zero downtime.

---

## Summary

AttendX demonstrates modern, cloud-native engineering best practices for mission-critical, high-throughput applications.  
It combines serverless compute, robust queueing, and a resilient CI/CD pipeline to deliver a seamless, reliable experience for both students and faculty—even at massive scale.

---
