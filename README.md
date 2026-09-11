# 🎓 CampusConnect 2026
## Student Registration & Login Portal on AWS EC2

CampusConnect 2026 is a dynamic student event registration and
authentication web application built with **PHP, MariaDB, HTML5, and CSS**.

The application is deployed on an **AWS EC2 instance running Amazon
Linux** and served publicly through **Nginx on port 80**. PHP requests are
processed using **PHP-FPM**, while student registration and login
credentials are stored in a MariaDB database.

```
Student
   │
   ▼
Web Browser
   │
   │ HTTP :80
   ▼
AWS EC2
   │
   ▼
Nginx
   │
   ▼
PHP-FPM
   │
   ▼
PHP Application
   │
   ▼
MariaDB
   │
   ▼
studentdb.students
```

---

# 📌 Project Overview

CampusConnect 2026 provides a simple student event registration and login
system. Students can:

- Register for an event with their name, college, location, and event
- Create a password
- Log in using either their email or their auto-generated Student ID
- Receive a success message after valid authentication
- Receive an error message for invalid credentials
- Log out securely using PHP sessions

The application is designed as an educational cloud deployment project,
demonstrating the integration of **Linux, AWS EC2, Nginx, PHP, PHP-FPM,
MariaDB, SQL, and server-side authentication**.

> **Design note:** `Student_id` is an `AUTO_INCREMENT` primary key, assigned
> by MariaDB on insert — it can't also be a field a student types in on the
> registration form. So registration shows the newly generated ID back to
> the student, and login accepts either that ID or the student's email.

---

# ✨ Features

## 📝 Student Registration

The registration form contains:

| Field        | Required |
| ------------ | -------- |
| Full Name    | ✅        |
| Email        | ✅        |
| College Name | ✅        |
| Location     | ✅        |
| Event        | ✅        |
| Password     | ✅        |

Registration data is stored in the MariaDB `studentdb` database, inside the
`students` table. The application prevents duplicate email addresses using
a `UNIQUE` constraint.

## 🔐 Student Login

Students log in using:

```
Email / Student ID
Password
```

The login system checks the submitted credentials against the database.

**Successful login:**
```
Welcome to CampusConnect!
```

**Invalid login:**
```
Invalid username or password.
```

Passwords are verified using PHP's `password_verify()` function.

## 🔒 Password Security

Passwords are never stored as plain text.

- During registration, PHP uses `password_hash()`
- During login, PHP uses `password_verify()`

This means the database stores a bcrypt hash, never the original password.

## 👤 Session Management

After successful authentication, PHP creates a session containing the
logged-in student's information:

```
$_SESSION["student_id"]
$_SESSION["full_name"]
```

`logout.php` destroys the session and returns the user to the login page.

---

# 🛠️ Technology Stack

| Technology   | Purpose                        |
| ------------ | ------------------------------- |
| Amazon Linux | Cloud server operating system   |
| AWS EC2      | Cloud compute instance          |
| Nginx        | Web server                      |
| PHP          | Server-side application logic   |
| PHP-FPM      | PHP processing for Nginx        |
| MariaDB      | Relational database             |
| SQL          | Database creation and queries   |
| HTML5        | Page structure                  |
| CSS3         | Styling (single shared stylesheet) |
| SSH          | Remote server administration    |
| Linux CLI    | Server management               |

> This project uses **LEMP architecture**: Linux + Nginx + MariaDB + PHP/PHP-FPM.

---

# 📂 Project Structure

```
campusconnect/
├── index.html               # Homepage
├── register.html            # Registration form
├── login.html                # Login form
├── register.php               # Registration handler
├── login.php                   # Login handler
├── logout.php                   # Session destroy + redirect
├── config.php                    # Real DB credentials (not committed)
├── config.example.php             # Template for config.php
├── style.css                       # Shared circuit-board theme
├── nginx-campusconnect.conf         # Nginx server block
├── .gitignore
├── commands.txt
└── README.md
```

---

# 📄 File Description

**`index.html`** — the homepage, with navigation to Register and Log in.

**`register.html`** — the registration form. Submits to `register.php`.

**`register.php`** — registration processing logic:
1. Validates required fields and email format
2. Hashes the password with `password_hash()`
3. Inserts the new row into `students`
4. Catches duplicate-email errors from the `UNIQUE` constraint
5. Shows a success screen with the new Student ID, or an error screen

**`login.html`** — the login form. Submits to `login.php`.

**`login.php`** — login logic:
1. Accepts an email or a numeric Student ID
2. Looks up the matching row in `students`
3. Verifies the password with `password_verify()`
4. Starts a PHP session on success
5. Shows **"Welcome to CampusConnect!"** or **"Invalid username or password."**

**`logout.php`** — destroys the session and redirects to `login.html`.

**`config.php`** — database connection settings, using PDO. Not committed
to GitHub (see Security Notes).

**`style.css`** — the shared stylesheet used by every page.

**`nginx-campusconnect.conf`** — the Nginx server block that routes `.php`
requests to PHP-FPM.

---

# 🗄️ Database Configuration

**Database name:**
```
studentdb
```

**Table name:**
```
students
```

**Table structure:**
```sql
CREATE TABLE students (
    Student_id    INT PRIMARY KEY AUTO_INCREMENT,
    Full_Name     VARCHAR(20)  NOT NULL,
    email         VARCHAR(20)  NOT NULL UNIQUE,
    College_Name  VARCHAR(25)  NOT NULL,
    Location      VARCHAR(20)  NOT NULL,
    Event         VARCHAR(20),
    password      VARCHAR(255) NOT NULL
);
```

### Columns

| Column         | Type          | Description                     |
| -------------- | ------------- | -------------------------------- |
| `Student_id`   | INT           | Auto-increment primary key       |
| `Full_Name`    | VARCHAR(20)   | Student full name                |
| `email`        | VARCHAR(20)   | Unique email address             |
| `College_Name` | VARCHAR(25)   | College name                     |
| `Location`     | VARCHAR(20)   | Student location                 |
| `Event`        | VARCHAR(20)   | Selected event                   |
| `password`     | VARCHAR(255)  | Bcrypt password hash             |

---

# 🗃️ Database Setup

Connect to MariaDB:
```
sudo mysql
```

Create the database:
```sql
CREATE DATABASE studentdb;
```

Select it:
```sql
USE studentdb;
```

Create the table:
```sql
CREATE TABLE students (
    Student_id    INT PRIMARY KEY AUTO_INCREMENT,
    Full_Name     VARCHAR(20)  NOT NULL,
    email         VARCHAR(20)  NOT NULL UNIQUE,
    College_Name  VARCHAR(25)  NOT NULL,
    Location      VARCHAR(20)  NOT NULL,
    Event         VARCHAR(20)
);
```

Add the password column:
```sql
ALTER TABLE students ADD COLUMN password VARCHAR(255) NOT NULL AFTER Event;
```

Verify the structure:
```sql
DESC students;
```

View registered students:
```sql
SELECT Student_id, Full_Name, email, College_Name, Location, Event
FROM students;
```

---

# 👤 Dedicated Database User (recommended)

This project currently connects as `root`, which is fine for a class
exercise but not something to carry into anything real. For a hardened
setup, create a dedicated user scoped to just this database:

```sql
CREATE USER 'campususer'@'localhost' IDENTIFIED BY 'YOUR_STRONG_PASSWORD';
GRANT ALL PRIVILEGES ON studentdb.* TO 'campususer'@'localhost';
FLUSH PRIVILEGES;
```

Then update `$db_user` / `$db_pass` in `config.php` to match.

---

# ☁️ AWS EC2 Deployment

```
        Internet
           │
           │ HTTP :80
           ▼
   ┌──────────────────┐
   │      AWS EC2      │
   │   Amazon Linux    │
   └────────┬──────────┘
            │
            ▼
         Nginx
            │
            ▼
        PHP-FPM
            │
            ▼
        PHP Pages
            │
            ▼
        MariaDB
            │
            ▼
   studentdb.students
```

# 🔐 EC2 Security Group

| Type | Port | Source      | Purpose                |
| ---- | ---- | ----------- | ------------------------ |
| SSH  | 22   | Your IP     | Remote administration    |
| HTTP | 80   | `0.0.0.0/0` | Public website access    |

MariaDB doesn't need to be exposed publicly — the PHP app and MariaDB run
on the same EC2 instance.

---

# 🐧 Linux Server Setup

Connect over SSH:
```
ssh -i your-key.pem ec2-user@YOUR_EC2_PUBLIC_IP
```

Check the current user and directory:
```
whoami
pwd
```

Check the web root:
```
ls /usr/share/nginx/html
```

Expected files:
```
config.php
index.html
login.html
login.php
logout.php
register.html
register.php
style.css
```

---

# 🌐 Nginx Installation

```
sudo dnf install -y nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

Expected: `Active: active (running)`

---

# 🐘 PHP and PHP-FPM Installation

```
sudo dnf install -y php php-fpm php-mysqlnd
php -v
sudo systemctl enable --now php-fpm
sudo systemctl status php-fpm
```

`php-mysqlnd` is required — without it, PDO can't talk to MariaDB.

---

# 🗄️ MariaDB Installation

```
sudo dnf install -y mariadb105-server
sudo systemctl enable --now mariadb
sudo systemctl status mariadb
```

---

# ⚙️ Nginx Configuration

The app is served from `/usr/share/nginx/html`. `.php` requests must be
routed to PHP-FPM, or POST requests to them return **405 Not Allowed**:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    root /usr/share/nginx/html;
    index index.html index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        try_files $uri =404;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/run/php-fpm/www.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Apply it:
```
sudo cp nginx-campusconnect.conf /etc/nginx/conf.d/default.conf
sudo nginx -t
sudo systemctl reload nginx
```

Expected: `syntax is ok` / `test is successful`

---

# 📁 Web Application Directory

```
sudo cp -r * /usr/share/nginx/html/
sudo chown -R nginx:nginx /usr/share/nginx/html
sudo chmod -R 755 /usr/share/nginx/html
```

---

# 🌍 Accessing the Website

```
Homepage:     http://YOUR_EC2_PUBLIC_IP/
Registration: http://YOUR_EC2_PUBLIC_IP/register.html
Login:        http://YOUR_EC2_PUBLIC_IP/login.html
```

---

# 🔄 Application Flow

## Registration Flow
```
Student
   │
   ▼
register.html
   │
   ▼
register.php
   │
   ▼
Validate fields
   │
   ▼
password_hash()
   │
   ▼
INSERT INTO students
   │
   ▼
"You're registered" + new Student ID
```

## Login Flow
```
Student
   │
   ▼
login.html
   │
   ▼
login.php
   │
   ▼
Email or Student ID + Password
   │
   ▼
SELECT matching row
   │
   ▼
password_verify()
   │
   ├──── Incorrect ────► "Invalid username or password."
   │
   └──── Correct
            │
            ▼
     Start PHP session
            │
            ▼
   "Welcome to CampusConnect!"
```

---

# 🧪 Complete Testing Procedure

**1. Test homepage** — open `http://YOUR_EC2_PUBLIC_IP/`, confirm it loads.

**2. Test registration** — open `register.html`, fill in all fields,
submit. Expected: "You're registered" with a new Student ID.

**3. Verify in MariaDB:**
```sql
USE studentdb;
SELECT Student_id, Full_Name, email, College_Name, Location, Event
FROM students;
```
The new student should appear.

**4. Test valid login** — use the registered email (or Student ID) and
password. Expected: "Welcome to CampusConnect!"

**5. Test invalid login** — use a wrong password. Expected: "Invalid
username or password."

**6. Test logout** — confirm the session ends and you're returned to the
login page.

---

# ✅ Service Verification

```
sudo systemctl is-active nginx
sudo systemctl is-active mariadb
sudo systemctl is-active php-fpm
```
Expected: `active`, `active`, `active`

```
sudo ss -tulpn | grep ':80'
```
Nginx should be listening on port 80.

---

# 📊 Practical Verification Checklist

## AWS EC2
- [ ] EC2 instance created and running
- [ ] Public IPv4 address available
- [ ] Security Group configured (ports 22, 80)

## Linux
- [ ] SSH connection successful
- [ ] Web root populated with application files
- [ ] File permissions/ownership checked

## Nginx
- [ ] Nginx installed and running
- [ ] `nginx -t` successful
- [ ] Port 80 listening

## PHP
- [ ] PHP and PHP-FPM installed
- [ ] PHP-FPM running
- [ ] PHP pages execute through Nginx

## MariaDB
- [ ] MariaDB installed and running
- [ ] `studentdb` database created
- [ ] `students` table created with `password` column
- [ ] Registration data inserted and verified with `SELECT`

## Website
- [ ] Homepage working
- [ ] Registration form working, success screen shown
- [ ] Login form working, valid login shows welcome message
- [ ] Invalid login shows error message
- [ ] Logout tested
- [ ] Site accessible through the EC2 public IP

---

# 📸 Submission Evidence

For the practical exam, capture screenshots of each of the following.

**AWS**
1. EC2 instance creation
2. Running EC2 instance
3. Security Group with HTTP port 80 open

**Linux**
4. SSH connection
5. Linux user creation
6. Web root file listing

**LEMP**
7. Nginx installation / status
8. PHP version and PHP-FPM status
9. MariaDB status
10. `nginx -t` successful configuration test

**SQL**
11. Database creation
12. `students` table structure (`DESC students;`)
13. Registered student data (`SELECT` output)

**Website**
14. Homepage
15. Registration form
16. Successful registration
17. Login form
18. Successful login
19. Invalid login

**Final Testing**
20. Website accessed through the EC2 public IP
21. Service and port verification

---

# 🔐 Security Notes

**Never commit `config.php`** — it holds your real database password.
This repo's `.gitignore` already excludes it; commit `config.example.php`
instead, and each collaborator copies it to their own `config.php` locally.

Do not publish:
- Database passwords
- AWS access/secret keys
- Private SSH keys (`.pem` files)

If a real password has ever been shared anywhere outside this repo (chat,
email, etc.), treat it as compromised and change it before going further.

---

# 🚀 Future Improvements

- HTTPS/SSL with a domain name
- A separate, manually-entered Student/Roll ID field
- Forgot-password flow
- Admin dashboard for viewing registrations
- CSRF protection
- Rate limiting on login attempts
- Wider `VARCHAR` limits on `Full_Name` and `email`
- Automated database backups

---

# 📋 Project Information

| Category         | Details                              |
| ----------------- | -------------------------------------- |
| Project Name      | CampusConnect 2026                     |
| Project Type      | Student Registration & Login Portal    |
| Frontend          | HTML5 + CSS3                           |
| Backend           | PHP (PDO)                              |
| Database          | MariaDB                                |
| Database Name     | `studentdb`                            |
| Table Name        | `students`                             |
| Web Server        | Nginx                                  |
| PHP Processing    | PHP-FPM                                |
| Server OS         | Amazon Linux                           |
| Cloud Platform    | AWS EC2                                |
| Web Port          | 80                                     |
| Application Path  | `/usr/share/nginx/html`                |

---

# 📜 License

This project is created for educational, learning, and practical cloud
deployment purposes.
