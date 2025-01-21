# Social Media Scheduler Database

## Overview
The Social Media Scheduler Database is a relational database designed to integrate Figma with popular social media platforms. It enables users to design content in Figma and schedule posts directly, streamlining the content creation process while ensuring compliance with platform-specific constraints such as image sizes and formats.

---

## Project Details

### **Introduction**
In today’s world where social media can become one’s career, the needs of content creators are growing. This project proposes a database for an integrated platform that connects Figma with popular social media networks, allowing users to design and schedule posts directly within the Figma interface. By simplifying the content creation process and ensuring compliance with platform-specific requirements, this solution enhances user efficiency and effectiveness. The project includes detailed entity definitions, relationships, and business rules to ensure data integrity and efficient management of user designs, posts, and platforms.

---

### **Business Case**
Social media content creators use various tools to create content, including Figma. This project develops an integrated platform allowing users to design in Figma and schedule posts on platforms like Facebook, Instagram, and LinkedIn, adhering to platform-specific requirements.

#### **Entities**
- **User:** Represents a user who creates designs and schedules posts.
- **Design:** Represents the designs created in Figma by users.
- **Post:** Contains the details of the scheduled posts.
- **Platform:** Represents social media platforms where the posts will be published (e.g., Facebook, Instagram).
- **Scheduler:** Handles the scheduling of posts to specific platforms.

#### **Business Rules**
- A User can create multiple Designs in Figma.
- A Design can be used in multiple Posts, but each Post can have only one Design.
- A User can schedule multiple Posts to be published on different Platforms.
- Each Post must be associated with one Platform, but a Platform can have many Posts.
- A Scheduler must handle the timing and order of posts for each Platform.
- Posts must include attributes like scheduled date, time, and platform-specific constraints (image size, format).

#### **Cardinality and Relationship Details**
- **User-Design:** A User is mandatory and can create many Designs, but a Design belongs to only one User (1-to-Many, mandatory on both sides).
- **Design-Post:** A Post uses one Design, and a Design can be used in multiple Posts (1-to-Many).
- **User-Post:** A User can create multiple Posts, but each Post is created by only one User (1-to-Many).
- **Post-Platform:** A Post is published on one Platform, and a Platform can host multiple Posts (1-to-Many).
- **Post-Scheduler:** A Scheduler is responsible for scheduling a Post to a platform at a specific time (1-to-1).

---

## Database Design

### **Entities and Attributes**
#### **User:**
- UserID (Primary Key)
- UserName
- Email
- Password

#### **Design:**
- DesignID (Primary Key)
- UserID (Foreign Key)
- DesignName
- CreatedDate

#### **Post:**
- PostID (Primary Key)
- DesignID (Foreign Key)
- UserID (Foreign Key)
- PlatformID (Foreign Key)
- PostContent
- ScheduledDateTime

#### **Platform:**
- PlatformID (Primary Key)
- PlatformName
- ImageSizeConstraint

#### **Scheduler:**
- SchedulerID (Primary Key)
- PostID (Foreign Key)
- ScheduledDateTime

### **Design Decisions and Justifications**
- **Primary Keys (PKs):** Ensure unique identification of each record.
- **Foreign Keys (FKs):** Establish relationships between tables to maintain data integrity.
- **Relationships:** Enforce mandatory relationships to link users, designs, posts, platforms, and schedulers.

---

## DB Implementation
### **SQL Schema**
```sql
CREATE DATABASE SocialMediaScheduler;
USE SocialMediaScheduler;

CREATE TABLE User (
    UserID INT PRIMARY KEY AUTO_INCREMENT,
    UserName VARCHAR(100) NOT NULL,
    Email VARCHAR(100) NOT NULL UNIQUE,
    Password VARCHAR(255) NOT NULL
);

CREATE TABLE Design (
    DesignID INT PRIMARY KEY AUTO_INCREMENT,
    UserID INT NOT NULL,
    DesignName VARCHAR(100) NOT NULL,
    CreatedDate DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (UserID) REFERENCES User(UserID) ON DELETE CASCADE
);

CREATE TABLE Platform (
    PlatformID INT PRIMARY KEY AUTO_INCREMENT,
    PlatformName VARCHAR(50) NOT NULL,
    ImageSizeConstraint VARCHAR(50) NOT NULL
);

CREATE TABLE Post (
    PostID INT PRIMARY KEY AUTO_INCREMENT,
    DesignID INT NOT NULL,
    UserID INT NOT NULL,
    PlatformID INT NOT NULL,
    PostContent TEXT NOT NULL,
    ScheduledDateTime DATETIME NOT NULL,
    FOREIGN KEY (DesignID) REFERENCES Design(DesignID) ON DELETE CASCADE,
    FOREIGN KEY (UserID) REFERENCES User(UserID) ON DELETE SET NULL,
    FOREIGN KEY (PlatformID) REFERENCES Platform(PlatformID) ON DELETE CASCADE
);

CREATE TABLE Scheduler (
    SchedulerID INT PRIMARY KEY AUTO_INCREMENT,
    PostID INT NOT NULL,
    ScheduledDateTime DATETIME NOT NULL,
    FOREIGN KEY (PostID) REFERENCES Post(PostID) ON DELETE CASCADE
);
```

---

## Sample Data
### **Insert Statements**
```sql
INSERT INTO User (UserName, Email, Password)
VALUES ('JohnDoe', 'john@example.com', 'hashedpassword1'),
       ('JaneSmith', 'jane@example.com', 'hashedpassword2');

INSERT INTO Platform (PlatformName, ImageSizeConstraint)
VALUES ('Facebook', '1080x1080'),
       ('Instagram', '1080x1080'),
       ('LinkedIn', '1200x627');

INSERT INTO Design (UserID, DesignName, CreatedDate)
VALUES (1, 'Summer Campaign Design', '2024-10-01'),
       (2, 'Winter Campaign Design', '2024-10-05');

INSERT INTO Post (DesignID, UserID, PlatformID, PostContent, ScheduledDateTime)
VALUES (1, 1, 1, 'Check out our summer sale!', '2024-10-25 10:00:00'),
       (2, 2, 2, 'Winter deals are here!', '2024-11-01 14:00:00');

INSERT INTO Scheduler (PostID, ScheduledDateTime)
VALUES (1, '2024-10-25 10:00:00'),
       (2, '2024-11-01 14:00:00');
```

---

## Query Example
Retrieve posts scheduled by a specific user:
```sql
SELECT Post.PostContent, Design.DesignName, Platform.PlatformName, Post.ScheduledDateTime
FROM Post
JOIN Design ON Post.DesignID = Design.DesignID
JOIN Platform ON Post.PlatformID = Platform.PlatformID
JOIN User ON Post.UserID = User.UserID
WHERE User.UserName = 'JohnDoe';
```

---

## Demo
The project includes a PHP script to showcase dynamic data handling:

```php
<?php
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "SocialMediaScheduler";

$conn = new mysqli($servername, $username, $password, $dbname);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$sql = "SELECT Post.PostContent, Design.DesignName, Platform.PlatformName, Post.ScheduledDateTime 
        FROM Post
        JOIN Design ON Post.DesignID = Design.DesignID
        JOIN Platform ON Post.PlatformID = Platform.PlatformID
        JOIN User ON Post.UserID = User.UserID
        WHERE User.UserName = 'JohnDoe';";

$result = $conn->query($sql);
if ($result->num_rows > 0) {
    while ($row = $result->fetch_assoc()) {
        echo "Post Content: " . $row["PostContent"] . " - Design Name: " . $row["DesignName"] .
             " - Platform: " . $row["PlatformName"] . " - Scheduled Date/Time: " . $row["ScheduledDateTime"] . "<br>";
    }
} else {
    echo "0 results";
}
$conn->close();
?>
```

---

## Conclusion
The database design effectively captures the essential components of the integrated platform, ensuring that relationships between users, designs, posts, and social media platforms are well-defined. By enforcing key constraints and utilizing foreign keys, the database maintains data integrity while supporting scalability for future enhancements. Overall, this structured approach provides a solid foundation for the platform, enabling efficient content management.
