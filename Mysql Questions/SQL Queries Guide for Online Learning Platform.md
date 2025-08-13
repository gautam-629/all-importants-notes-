

## Introduction

This document contains a comprehensive collection of SQL queries for managing an online learning platform database. The database schema includes tables for users, courses, enrollments, categories, reviews, and course categorizations. These queries cover common scenarios for analyzing user behavior, course performance, and platform statistics.

## Database Schema

### Table Creation Scripts

```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    role ENUM('student', 'instructor') NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE courses (
    course_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    description TEXT,
    instructor_id INT NOT NULL,
    price DECIMAL(10, 2) DEFAULT 0.00,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (instructor_id) REFERENCES users(user_id)
);

CREATE TABLE enrollments (
    enrollment_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    course_id INT NOT NULL,
    enrolled_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE reviews (
    review_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    course_id INT NOT NULL,
    rating INT CHECK (rating >= 1 AND rating <= 5),
    comment TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

CREATE TABLE course_categories (
    course_id INT NOT NULL,
    category_id INT NOT NULL,
    PRIMARY KEY (course_id, category_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id),
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);
```

### Schema Overview

The platform consists of six main tables:

- **users**: Stores user information with roles (student/instructor)
- **courses**: Contains course details and pricing information
- **enrollments**: Tracks student-course relationships
- **categories**: Defines course categories
- **reviews**: Stores user reviews and ratings
- **course_categories**: Links courses to their categories

---

## 👥 Users & Roles

### Basic User Statistics

**How many users have signed up so far?**

```sql
SELECT COUNT(*) FROM users;
```

**How many instructors and students are currently in the system?**

```sql
SELECT role, COUNT(*) FROM users GROUP BY role;
```

**List the latest 10 registered users**

```sql
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
```

### User Activity Analysis

**Which students haven't enrolled in any course yet?**

```sql
SELECT u.name, en.enrollment_id
FROM users u
LEFT JOIN enrollments en ON u.user_id = en.user_id 
WHERE en.enrollment_id IS NULL AND u.role = 'student';
```

**Which instructors haven't created any courses yet?**

```sql
SELECT u.name, c.instructor_id 
FROM users u
LEFT JOIN courses c ON c.instructor_id = u.user_id
WHERE c.instructor_id IS NULL AND u.role = 'instructor';
```

**Who are the most active students based on course enrollments?**

```sql
SELECT 
    u.name,
    u.email,
    COUNT(e.enrollment_id) AS enrollment_count
FROM users u 
JOIN enrollments e ON e.user_id = u.user_id 
WHERE u.role = 'student' 
GROUP BY u.user_id, u.name 
ORDER BY enrollment_count DESC 
LIMIT 3;
```

### User Engagement

**Which users have submitted at least one review?**

```sql
SELECT 
    u.name,
    u.email,
    COUNT(r.review_id) AS total_reviews
FROM users u
JOIN reviews r ON r.user_id = u.user_id 
GROUP BY u.user_id, u.name, u.email
HAVING total_reviews >= 1;
```

**List users who joined in the last 7 days**

```sql
SELECT 
    user_id,
    name,
    email,
    role,
    created_at
FROM users
WHERE created_at >= NOW() - INTERVAL 7 DAY;
```

---

## 📚 Courses & Instructors

### Course Information

**List all courses along with their instructors' names**

```sql
SELECT 
    c.title,
    c.description,
    u.name AS instructor_name,
    u.email AS instructor_email
FROM courses c
JOIN users u ON u.user_id = c.instructor_id
WHERE u.role = 'instructor';
```

### Course Performance Analysis

**Which courses have never been enrolled in?**

```sql
SELECT
    c.title,
    e.enrolled_at,
    e.enrollment_id
FROM courses c 
LEFT JOIN enrollments e ON e.course_id = c.course_id 
WHERE e.enrollment_id IS NULL;
```

**Which courses are completely free?**

```sql
SELECT 
    title,
    course_id 
FROM courses 
WHERE price = 0.00;
```

### Pricing Analysis

**What is the average price of paid courses?**

```sql
SELECT AVG(price) AS average_price
FROM courses 
WHERE price > 0;
```

**What are the top 5 most expensive courses?**

```sql
SELECT 
    price,
    title 
FROM courses 
ORDER BY price DESC 
LIMIT 5;
```

### Instructor Productivity

**Which instructors have published more than 3 courses?**

```sql
SELECT 
    u.name,
    u.email, 
    COUNT(c.course_id) AS total_courses
FROM users u 
JOIN courses c ON c.instructor_id = u.user_id
GROUP BY u.user_id, u.name 
HAVING total_courses > 3;
```

### Course Quality Control

**Which courses are missing a description?**

```sql
SELECT 
    price,
    title,
    description 
FROM courses 
WHERE description IS NULL;
```

**List all courses created in the last 30 days**

```sql
SELECT * 
FROM courses 
WHERE created_at > NOW() - INTERVAL 30 DAY;
```

---

## 📥 Enrollments

### Enrollment Analysis

**Which students are enrolled in the course 'Intro to Python'?**

```sql
SELECT 
    u.name,
    u.email,
    c.title
FROM users u 
JOIN enrollments e ON e.user_id = u.user_id 
JOIN courses c ON e.course_id = c.course_id 
WHERE c.title = 'Intro to Python';
```

---

## 💡 Tips for Using These Queries

1. **Performance Optimization**: Add indexes on frequently queried columns like `user_id`, `course_id`, and `created_at`
2. **Data Validation**: Always validate user inputs when using these queries in applications
3. **Security**: Use parameterized queries to prevent SQL injection attacks
4. **Monitoring**: Regularly run user and course statistics queries to monitor platform growth
5. **Maintenance**: Use the "missing description" and "unused courses" queries for data quality maintenance

## Common Query Patterns

- **LEFT JOIN**: Used to find records that don't have matching relationships (e.g., students without enrollments)
- **GROUP BY with HAVING**: Used for aggregating data with conditions (e.g., active users, productive instructors)
- **DATE Functions**: Used for time-based analysis (e.g., recent registrations, new courses)
- **COUNT and AVG**: Used for statistical analysis and reporting

---

_This guide serves as a reference for database administrators, developers, and analysts working with online learning platform data._