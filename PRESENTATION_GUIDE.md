# DBS_CLOUD Project - Presentation

## Overview

A **Node.js web application** on **Google Cloud Platform** that captures and displays visitor information (IP, browser, location) and stores it in PostgreSQL.

---

## Key Files

### **app.js** - Backend Server

- Express.js server on port 8080
- Captures visitor info: IP, browser, OS, location
- Uses ipapi.co API for geolocation
- Stores data in PostgreSQL with Sequelize
- Security: Helmet.js, rate limiting

**Main routes:**

- `GET /` - Home page
- `GET /api/client-info` - Get visitor info
- `GET /api/logs` - Get paginated visitor logs
- `GET /logs` - Logs page

### **Database Files**

- **config/config.js** - Database connection settings
- **models/visit.js** - Visitor data structure (IP, browser, location, timestamp)
- **migrations/** - Creates the Visits table

### **Frontend Files**

- **public/index.html** - Shows your IP and location
- **public/logs.html** - Shows all visitor records with pagination
- **public/js/client-info.js** - Fetches and displays visitor data
- **public/css/styles.css** - Styling

### **Configuration Files**

- **package.json** - Dependencies (express, sequelize, pg, axios, helmet)
- **app.yaml** - Google Cloud settings (Node.js 22, autoscaling)
- **cloudbuild.yaml** - Auto-deployment steps
- **Dockerfile** - Container configuration
- **docker-compose.yml** - Local development setup

---

## How It Works

1. User visits the homepage
2. JavaScript calls `/api/client-info`
3. Server captures IP, browser, OS
4. Gets location from IP using ipapi.co
5. Stores all data in PostgreSQL
6. Displays info on the page

---

## Security

- Helmet.js - Protects against vulnerabilities (XSS, clickjacking)
- Rate limiting - Max 100 requests per 15 minutes
- Input validation - Checks API parameters
- Secret Manager - Secure database password storage
- SSL/TLS - Encrypted connections to database

---

## Deployment

Deployed on Google Cloud:

- **App Engine** - Hosts the app
- **Cloud SQL** - PostgreSQL database
- **Secret Manager** - Stores credentials
- **Cloud Build** - Automated deployment

Command: `gcloud app deploy`

---

## Technologies Used

| Component        | Technology              |
| ---------------- | ----------------------- |
| Server           | Node.js 22 + Express    |
| Database         | PostgreSQL              |
| ORM              | Sequelize               |
| Hosting          | Google Cloud App Engine |
| Containerization | Docker                  |
| Security         | Helmet, Rate Limiting   |

---

## Summary

**What was built:** A web app that tracks visitor information and displays it with a logs dashboard

**Features:** Real-time visitor info capture, geolocation, paginated logs, cloud deployment, security measures

- How to build a Node.js backend with Express
- Database design and management with Sequelize
- Security best practices (Helmet, rate limiting, secret management)
- Deployment to cloud platforms (Google Cloud)
- Frontend-backend communication with APIs
- Docker containerization
- Automated CI/CD with Cloud Build
