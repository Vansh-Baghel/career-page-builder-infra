# Career Page Builder - Instructions

This project consists of three repositories that work together to provide a complete career page builder solution.

## 📦 Repositories

- **Frontend**: [career-page-builder-fe](https://github.com/Vansh-Baghel/career-page-builder-fe)
- **Backend**: [career-page-builder-be](https://github.com/Vansh-Baghel/career-page-builder-be)
- **Infrastructure**: [career-page-builder-infra](https://github.com/Vansh-Baghel/career-page-builder-infra)

## 🚀 Quick Start (Recommended: Docker)

### Prerequisites
- Docker and Docker Compose installed on your system
- Git


## Environment Setup

The backend includes separate environment files for different deployment scenarios:

- **`.env`**: Configuration for running the backend inside Docker
- **`.env.local`**: Configuration for running the backend locally (without Docker)
- For Frontend, just change the env.local to .env to run the frontend.
- I have added env.example if you prefer running via Docker, and env.local.example if you wish to proceed without Docker.
- Only values you may need to setup are: 
`JWT_SECRET`, `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`.

### Setup Steps

1. **Clone all repositories into a single folder**:

```bash
mkdir career-page-builder
cd career-page-builder

git clone https://github.com/Vansh-Baghel/career-page-builder-fe
git clone https://github.com/Vansh-Baghel/career-page-builder-be
git clone https://github.com/Vansh-Baghel/career-page-builder-infra
```

2. **Navigate to the infrastructure folder and start Docker**:

```bash
cd career-page-builder-infra
docker compose up --build
```

3. **Run database migrations** (in a new terminal while Docker is running):

```bash
docker compose exec backend npm run migration:run
```

4. **Access the application**:
   - Frontend will be available at the configured port (typically `http://localhost:3000`)
   - Backend API will be available at the configured port (typically `http://localhost:4000`)

## 🔧 Alternative Setup (Without Docker)

If you encounter issues with Docker, you can run the frontend and backend separately.

### Prerequisites
- Node.js and npm installed
- Database setup (if required)
- Add .env for both backend and frontend as shared in above instructions.

### Setup Steps

1. **Clone only fe and be repositories**:

```bash
mkdir career-page-builder
cd career-page-builder

git clone https://github.com/Vansh-Baghel/career-page-builder-fe
git clone https://github.com/Vansh-Baghel/career-page-builder-be
# git clone https://github.com/Vansh-Baghel/career-page-builder-infra
```

2. **Setup and run Backend**:

```bash
cd career-page-builder-be
npm install
npm run dev
```

3. **Setup and run Frontend** (in a new terminal):

```bash
cd career-page-builder-fe
npm install
npm run migrate:run
npm run dev
```


# Career Page Builder - Frontend

## 📁 Project Structure

```
career-page-builder-fe/
└── src/
    ├── app/                # Next.js App Router pages
    │   ├── careers/        # Public careers pages
    │   ├── (protected)/   # Recruiter dashboard pages
        └── (auth)/        # Login pages
    ├── components/        # Reusable UI components (Shadcn)
    └── lib/               # API clients, helpers, types
```

## ⚙️ Tech Stack

- Next.js 16 (App Router)
- TypeScript
- Tailwind CSS
- Shadcn UI
- Zustand (which I removed later on) 
- TanStack Query
- Axios
- Lucide Icons
- Sonner (toast notifications)
- Zod for Input forms 

# Career Page Builder - Backend

## 📁 Project Structure

```
career-page-builder-be/
└── src/
    ├── entities/          # TypeORM entities (Company, Job, Recruiter, etc.)
    ├── controllers/      # Express route controllers
    ├── routes/           # API route definitions
    ├── config/
    │   └── data-source.ts # TypeORM configuration
    ├── migrations/       # DB migrations
    └── server.ts         # Express entry point
```

## ⚙️ Tech Stack

- Express
- TypeScript
- PostgreSQL
- TypeORM
- JWT Authentication
- bcrypt
- dotenv
- Cloudinary (image uploads)
- Multer (multipart handling)
- nodemon + ts-node

# Future scope
- We could add a Draft DB, to store the changes in Draft, which I did add, but removed later on due to time contraint, and to keep it simple.
- Can implement cursor pagination in the frontend if the jobs and company list increases.
- Can create partitions and indexes in jobs table to partition by location, or company if necessary. 
- Can move the localStorage tokens with HttpOnly cookie sessions for improved security.
- Implement redis layer in the backend for caching.
- Option to add multiple jobs at once, adding jobs in batches, or option to drop a CSV file for importing the jobs.
- Can add `expires_at` in backend job DB, and could run a Cron job every hour or once a day to check the status of the job. 
- Unit and E2E testing for filters, job publishes, etc.
- Can add full-text search on title/department/location using PostgreSQL text indexes or ElasticSearch.
- And sure there can be many more 😄

# Instructions to import the sample data
- We will first create a new temporary table, add the CSV data via import, clean the data fields according to our backend values, then post it in our jobs table. 
- This create a layer of safety into our primary table.

```SQL
CREATE TABLE jobs_import (
  title TEXT,
  work_policy TEXT,
  location TEXT,
  department TEXT,
  employment_type TEXT,
  experience_level TEXT,
  job_type TEXT,
  salary_range TEXT,
  job_slug TEXT,
  posted_days_ago text
);

-- import CSV in the above, then clean the data, 

UPDATE jobs_import
SET work_policy = 'onsite'
WHERE LOWER(work_policy) IN ('on-site');

UPDATE jobs_import
SET employment_type = REPLACE(LOWER(employment_type), ' ', '-');

UPDATE jobs_import
SET employment_type = 'full-time'
WHERE LOWER(employment_type) IN ('full time');

UPDATE jobs_import
SET employment_type = 'part-time'
WHERE LOWER(employment_type) IN ('part time');

UPDATE jobs_import
SET experience_level = 'mid'
WHERE LOWER(experience_level) IN ('mid-level');

UPDATE jobs_import
SET job_type = LOWER(job_type);

-- After this, copy the values into our jobs table by insert, please note that I have given the company_id of an existing company in my DB. Please copy id of a company which got created on your login from your database.

INSERT INTO jobs (title, job_slug, posted_days_ago, job_type, department, experience_level, location, salary, employment_type, work_policy, company_id)
SELECT
  title,
  job_slug,
  posted_days_ago,
  job_type,
  department,
  experience_level,
  location,
  salary_range,
  employment_type,
  work_policy,
  '1beaf289-8e64-4404-a7c9-335e8c549576'
FROM jobs_import;

-- To verify, you can run
SELECT DISTINCT work_policy FROM jobs_import;
SELECT DISTINCT employment_type FROM jobs_import;
SELECT DISTINCT experience_level FROM jobs_import;
SELECT DISTINCT job_type FROM jobs_import;

-- Drop the temporary table (optional)
DROP TABLE jobs_import

```

## 📝 Notes

- All three repositories should be cloned into the same parent directory for Docker setup to work correctly.
- Database migrations must be run after the Docker containers are up and running.
