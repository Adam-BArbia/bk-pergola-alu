# Complete Database Schema - BK Pergola et Alu

**Document Purpose:** Complete MySQL schema, relationships, migrations, and data design.

---

## 1. Database Overview

### Design Principles

- **Normalization:** 3NF (Third Normal Form) to avoid data redundancy
- **Relationships:** 1-to-Many, Many-to-Many as needed
- **Scalability:** Indexed on frequently queried columns
- **Integrity:** Foreign keys ensure referential integrity
- **Type Safety:** Doctrine ORM handles type conversion

### Tables at a Glance

| Table | Purpose | Records |
|-------|---------|---------|
| `admins` | Admin users | ~5 (estimated) |
| `projects` | Portfolio projects | ~20-50 |
| `project_images` | Images per project | ~100-200 |
| `contacts` | Lead submissions | ~100-300/month |
| `team_members` | Team profiles | ~5 |

---

## 2. Table: `admins`

**Purpose:** Admin user accounts with authentication

**Doctrine Entity:**
```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: AdminRepository::class)]
#[ORM\Table(name: 'admins')]
#[ORM\Index(name: 'idx_username', columns: ['username'])]
class Admin
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;

    #[ORM\Column(type: 'string', length: 255, unique: true)]
    #[Assert\NotBlank]
    #[Assert\Length(min: 3, max: 50)]
    private string $username;

    #[ORM\Column(type: 'string', length: 255)]
    #[Assert\Email]
    private string $email;

    #[ORM\Column(type: 'string', length: 255)]
    private string $passwordHash; // bcrypt hashed

    #[ORM\Column(type: 'boolean')]
    private bool $isActive = true;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $createdAt;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $lastLogin = null;

    // Getters and setters...
}
```

**SQL:**
```sql
CREATE TABLE admins (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL,
    last_login DATETIME NULL,
    INDEX idx_username (username),
    CREATED_AT TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Fields:**
- `id` - Primary key
- `username` - Unique login username (3-50 chars)
- `email` - Contact email
- `password_hash` - bcrypt hashed password ($2y$10$...)
- `is_active` - Soft delete (deactivate without removing)
- `created_at` - Account creation timestamp
- `last_login` - Last successful login (for audit)

**Indexes:**
- `idx_username` - Fast lookup during login

---

## 3. Table: `projects`

**Purpose:** Portfolio projects displayed on gallery

**Doctrine Entity:**
```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\Collection;

#[ORM\Entity(repositoryClass: ProjectRepository::class)]
#[ORM\Table(name: 'projects')]
#[ORM\Index(name: 'idx_featured', columns: ['featured'])]
#[ORM\Index(name: 'idx_category', columns: ['category'])]
#[ORM\Index(name: 'idx_created_at', columns: ['created_at'])]
class Project
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;

    #[ORM\Column(type: 'string', length: 255)]
    private string $title; // FR

    #[ORM\Column(type: 'text')]
    private string $description; // FR

    #[ORM\Column(type: 'string', length: 50)]
    private string $category; // 'pergola', 'structure', 'custom'

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $location = null;

    #[ORM\Column(type: 'date', nullable: true)]
    private ?\DateTimeInterface $completionDate = null;

    #[ORM\Column(type: 'boolean')]
    private bool $featured = false;

    #[ORM\Column(type: 'integer')]
    private int $displayOrder = 0;

    #[ORM\OneToMany(mappedBy: 'project', targetEntity: ProjectImage::class, cascade: ['persist', 'remove'])]
    #[ORM\OrderBy(['position' => 'ASC'])]
    private Collection $images;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $createdAt;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $updatedAt;

    // Getters and setters...
}
```

**SQL:**
```sql
CREATE TABLE projects (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description LONGTEXT NOT NULL,
    category VARCHAR(50) NOT NULL,
    location VARCHAR(255) NULL,
    completion_date DATE NULL,
    featured BOOLEAN DEFAULT FALSE,
    display_order INT DEFAULT 0,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_featured (featured),
    INDEX idx_category (category),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Fields:**
- `id` - Primary key
- `title` - Project name (FR)
- `description` - Project details (FR)
- `category` - Type: 'pergola', 'structure', 'custom'
- `location` - Where built (city/region)
- `completion_date` - When completed
- `featured` - Show on homepage
- `display_order` - Sort order in gallery
- `created_at` - When added to portfolio
- `updated_at` - Last modified

**Indexes:**
- `idx_featured` - Find featured projects quickly
- `idx_category` - Filter by category
- `idx_created_at` - Sort by date

**Example Data:**
```sql
INSERT INTO projects (title, description, category, location, completion_date, featured, display_order, created_at, updated_at)
VALUES (
    'Pergola Aluminium Résidence Tunis',
    'Pergola moderne en aluminium pour résidence privée. Dimensions: 5m x 3m. Finition: blanc mat.',
    'pergola',
    'Tunis',
    '2025-03-15',
    TRUE,
    1,
    NOW(),
    NOW()
);
```

---

## 4. Table: `project_images`

**Purpose:** Images associated with projects (1 project → many images)

**Doctrine Entity:**
```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ProjectImageRepository::class)]
#[ORM\Table(name: 'project_images')]
class ProjectImage
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;

    #[ORM\ManyToOne(targetEntity: Project::class, inversedBy: 'images')]
    #[ORM\JoinColumn(name: 'project_id', referencedColumnName: 'id', onDelete: 'CASCADE')]
    private Project $project;

    #[ORM\Column(type: 'string', length: 255)]
    private string $imagePath; // Relative path: projects/2025/pergola-01.jpg

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $thumbnailPath = null; // thumbs/2025/pergola-01.jpg

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $altText = null; // For accessibility

    #[ORM\Column(type: 'integer')]
    private int $position = 0; // Display order within project

    #[ORM\Column(type: 'integer', nullable: true)]
    private ?int $width = null; // Image dimensions

    #[ORM\Column(type: 'integer', nullable: true)]
    private ?int $height = null;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $uploadedAt;

    // Getters and setters...
}
```

**SQL:**
```sql
CREATE TABLE project_images (
    id INT AUTO_INCREMENT PRIMARY KEY,
    project_id INT NOT NULL,
    image_path VARCHAR(255) NOT NULL,
    thumbnail_path VARCHAR(255) NULL,
    alt_text VARCHAR(255) NULL,
    position INT DEFAULT 0,
    width INT NULL,
    height INT NULL,
    uploaded_at DATETIME NOT NULL,
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    INDEX idx_project_id (project_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Fields:**
- `id` - Primary key
- `project_id` - Foreign key to projects
- `image_path` - Full path (e.g., `/uploads/projects/2025/pergola-01.jpg`)
- `thumbnail_path` - Optimized thumbnail path
- `alt_text` - SEO + accessibility (describe image)
- `position` - Order within gallery
- `width`, `height` - For responsive sizing
- `uploaded_at` - Upload timestamp

**Indexes:**
- `idx_project_id` - Find images by project

---

## 5. Table: `contacts`

**Purpose:** Lead submissions from contact form

**Doctrine Entity:**
```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ContactRepository::class)]
#[ORM\Table(name: 'contacts')]
#[ORM\Index(name: 'idx_status', columns: ['status'])]
#[ORM\Index(name: 'idx_source', columns: ['source'])]
#[ORM\Index(name: 'idx_created_at', columns: ['created_at'])]
class Contact
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;

    #[ORM\Column(type: 'string', length: 255)]
    private string $name;

    #[ORM\Column(type: 'string', length: 255)]
    private string $email;

    #[ORM\Column(type: 'string', length: 20, nullable: true)]
    private ?string $phone = null;

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $subject = null;

    #[ORM\Column(type: 'longtext')]
    private string $message;

    #[ORM\Column(type: 'string', length: 50)]
    private string $status = 'new'; // 'new', 'contacted', 'converted', 'spam'

    #[ORM\Column(type: 'string', length: 50)]
    private string $source = 'form'; // 'form', 'whatsapp', 'phone'

    #[ORM\Column(type: 'boolean')]
    private bool $replied = false;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $createdAt;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $repliedAt = null;

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $ipAddress = null; // For spam detection

    // Getters and setters...
}
```

**SQL:**
```sql
CREATE TABLE contacts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(20) NULL,
    subject VARCHAR(255) NULL,
    message LONGTEXT NOT NULL,
    status VARCHAR(50) DEFAULT 'new',
    source VARCHAR(50) DEFAULT 'form',
    replied BOOLEAN DEFAULT FALSE,
    created_at DATETIME NOT NULL,
    replied_at DATETIME NULL,
    ip_address VARCHAR(255) NULL,
    INDEX idx_status (status),
    INDEX idx_source (source),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Fields:**
- `id` - Primary key
- `name` - Visitor name
- `email` - Contact email
- `phone` - Phone number (optional)
- `subject` - Message subject (optional)
- `message` - Full message text
- `status` - Tracking: 'new', 'contacted', 'converted', 'spam'
- `source` - How they contacted: 'form', 'whatsapp', 'phone'
- `replied` - Admin marked as replied
- `created_at` - Submission timestamp
- `replied_at` - When admin replied
- `ip_address` - Source IP (spam prevention)

**Indexes:**
- `idx_status` - Filter by status in dashboard
- `idx_source` - See breakdown by channel
- `idx_created_at` - Sort by date

**Example Data:**
```sql
INSERT INTO contacts (name, email, phone, subject, message, status, source, created_at, ip_address)
VALUES (
    'Ahmed Ben Ali',
    'ahmed@example.tn',
    '+21622334455',
    'Demande de devis pergola',
    'Bonjour, je suis intéressé par une pergola pour ma terrasse...',
    'new',
    'form',
    NOW(),
    '192.168.1.1'
);
```

---

## 6. Table: `team_members`

**Purpose:** Team member profiles on /team page

**Doctrine Entity:**
```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: TeamMemberRepository::class)]
#[ORM\Table(name: 'team_members')]
#[ORM\Index(name: 'idx_position', columns: ['position'])]
class TeamMember
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;

    #[ORM\Column(type: 'string', length: 255)]
    private string $name;

    #[ORM\Column(type: 'string', length: 255)]
    private string $title; // Job title (FR)

    #[ORM\Column(type: 'text', nullable: true)]
    private ?string $bio = null; // Short biography (FR)

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $imagePath = null; // Profile photo

    #[ORM\Column(type: 'integer')]
    private int $position = 0; // Display order

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $email = null;

    #[ORM\Column(type: 'string', length: 20, nullable: true)]
    private ?string $phone = null;

    #[ORM\Column(type: 'boolean')]
    private bool $isActive = true;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $createdAt;

    // Getters and setters...
}
```

**SQL:**
```sql
CREATE TABLE team_members (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    title VARCHAR(255) NOT NULL,
    bio TEXT NULL,
    image_path VARCHAR(255) NULL,
    position INT DEFAULT 0,
    email VARCHAR(255) NULL,
    phone VARCHAR(20) NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL,
    INDEX idx_position (position)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Fields:**
- `id` - Primary key
- `name` - Full name
- `title` - Job title (FR)
- `bio` - Short biography (FR)
- `image_path` - Profile photo path
- `position` - Display order on team page
- `email` - Contact email
- `phone` - Phone number
- `is_active` - Show/hide on website
- `created_at` - Added to team

---

## 7. Entity Relationships Diagram

```
admins (1) ────────────────────────────────────── (many) projects
           [admin creates/manages projects]

projects (1) ────────────────────────────────────── (many) project_images
            [1 project has multiple images]

contacts (0..1) ────────────────────────────────── (many) admins
               [Contact assigned to admin - future feature]

team_members [standalone - no foreign keys]
```

**Relationship Summary:**

| From | To | Type | Cascade |
|------|-----|------|---------|
| projects | project_images | 1:N | ON DELETE CASCADE |
| admins | projects | 1:N | - |
| admins | contacts | 1:N | - (optional) |

---

## 8. Database Migrations (Doctrine)

### Generate Initial Schema

```bash
# Generate migration file (from entities)
php bin/console make:migration

# Apply to database
php bin/console doctrine:migrations:migrate
```

### Migration File Structure

```php
<?php

namespace DoctrineMigrations;

use Doctrine\DBAL\Schema\Schema;
use Doctrine\Migrations\AbstractMigration;

final class Version20260611000000 extends AbstractMigration
{
    public function getDescription(): string
    {
        return 'Create initial tables';
    }

    public function up(Schema $schema): void
    {
        $this->addSql('CREATE TABLE admins (...)');
        $this->addSql('CREATE TABLE projects (...)');
        $this->addSql('CREATE TABLE project_images (...)');
        $this->addSql('CREATE TABLE contacts (...)');
        $this->addSql('CREATE TABLE team_members (...)');
    }

    public function down(Schema $schema): void
    {
        $this->addSql('DROP TABLE IF EXISTS contacts');
        $this->addSql('DROP TABLE IF EXISTS project_images');
        $this->addSql('DROP TABLE IF EXISTS projects');
        $this->addSql('DROP TABLE IF EXISTS admins');
        $this->addSql('DROP TABLE IF EXISTS team_members');
    }
}
```

---

## 9. Indexes for Performance

### Current Indexes

| Table | Index | Purpose | Query Speed |
|-------|-------|---------|-------------|
| admins | idx_username | Login lookup | O(log n) |
| projects | idx_featured | Homepage featured projects | O(log n) |
| projects | idx_category | Filter by category | O(log n) |
| projects | idx_created_at | Sort by date | O(log n) |
| project_images | idx_project_id | Find images by project | O(log n) |
| contacts | idx_status | Dashboard filters | O(log n) |
| contacts | idx_source | Channel breakdown | O(log n) |
| contacts | idx_created_at | Sort by date | O(log n) |
| team_members | idx_position | Sort on page | O(log n) |

### Future Indexes (If Scaling)

```sql
-- If full-text search on projects
CREATE FULLTEXT INDEX ft_title_description ON projects(title, description);

-- If many contacts
CREATE INDEX idx_email ON contacts(email); -- Find duplicate submissions
CREATE INDEX idx_phone ON contacts(phone);

-- If caching by date range
CREATE INDEX idx_created_status ON contacts(created_at, status);
```

---

## 10. Constraints & Validation

### Primary Key Constraints
- All `id` columns: AUTO_INCREMENT, NOT NULL, UNIQUE

### Foreign Key Constraints
```sql
ALTER TABLE project_images ADD CONSTRAINT fk_project_images_project_id
FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE;
```

**ON DELETE CASCADE:** If project deleted, all images deleted automatically

### Unique Constraints
- `admins.username` - No duplicate usernames
- `admins.email` - No duplicate emails (best practice)

### NOT NULL Constraints

| Table | Field | Required |
|-------|-------|----------|
| admins | username, email, password_hash, created_at | Yes |
| projects | title, description, category, created_at, updated_at | Yes |
| project_images | project_id, image_path, uploaded_at | Yes |
| contacts | name, email, message, created_at | Yes |
| team_members | name, title, created_at | Yes |

---

## 11. Data Types & Field Sizes

### String Fields

| Field | Type | Size | Rationale |
|-------|------|------|-----------|
| username | VARCHAR | 255 | Short, indexed |
| email | VARCHAR | 255 | Standard email length |
| title (project) | VARCHAR | 255 | Reasonable limit |
| description | LONGTEXT | - | Full content, no limit |
| category | VARCHAR | 50 | Fixed options |
| status | VARCHAR | 50 | Fixed options |
| source | VARCHAR | 50 | Fixed options |
| phone | VARCHAR | 20 | International format +216XXXXXXXX |

### Numeric Fields

| Field | Type | Rationale |
|-------|------|-----------|
| id | INT AUTO_INCREMENT | Standard primary key |
| position | INT | Sort order (0-1000) |
| display_order | INT | Gallery sort order |
| width, height | INT | Image dimensions in pixels |

### Date/Time Fields

| Field | Type | Rationale |
|-------|------|-----------|
| created_at | DATETIME | Record creation |
| updated_at | DATETIME | Last modification |
| last_login | DATETIME | Optional, nullable |
| completion_date | DATE | No time needed |

### Boolean Fields

| Field | Type | Default |
|-------|------|---------|
| featured | BOOLEAN | FALSE |
| is_active | BOOLEAN | TRUE |
| replied | BOOLEAN | FALSE |

---

## 12. Sample Data (Seeding)

### Add Sample Admin

```sql
INSERT INTO admins (username, email, password_hash, created_at)
VALUES ('adam', 'adam@bk-pergola.tn', '$2y$10$...', NOW());
```

**Generate bcrypt hash in PHP:**
```php
$hash = password_hash('secure_password_here', PASSWORD_BCRYPT);
// Output: $2y$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcg7b3XeKeUxWdeS86E36joDSv2
```

### Add Sample Project

```sql
INSERT INTO projects (title, description, category, location, completion_date, featured, display_order, created_at, updated_at)
VALUES (
    'Pergola Moderne Tunis 2025',
    'Magnifique pergola en aluminium blanc mat. Dimensions: 6m x 4m. Parfait pour ombrage et détente.',
    'pergola',
    'La Marsa, Tunis',
    '2025-02-20',
    TRUE,
    1,
    NOW(),
    NOW()
);
```

### Add Images for Project

```sql
INSERT INTO project_images (project_id, image_path, thumbnail_path, alt_text, position, width, height, uploaded_at)
VALUES 
(1, '/uploads/projects/2025/pergola-main.jpg', '/uploads/thumbs/2025/pergola-main.jpg', 'Pergola vue de face', 0, 1200, 800, NOW()),
(1, '/uploads/projects/2025/pergola-detail.jpg', '/uploads/thumbs/2025/pergola-detail.jpg', 'Detail de la structure', 1, 1200, 800, NOW());
```

### Add Team Member

```sql
INSERT INTO team_members (name, title, bio, position, email, phone, created_at)
VALUES (
    'Mohamed Ben Khalifa',
    'Directeur & Fondateur',
    'Expert en structures aluminium avec 15 ans d\'expérience',
    0,
    'mohamed@bk-pergola.tn',
    '+21622334455',
    NOW()
);
```

---

## 13. Backup & Recovery Strategy

### Automated Backup

OVHcloud provides 24-hour automatic backup (recovery point if data loss).

### Manual Backup (Weekly)

```bash
# Export database to SQL file
mysqldump -u username -p database_name > backup_$(date +%Y%m%d).sql

# Restore from backup
mysql -u username -p database_name < backup_20260611.sql
```

### Backup Retention

- Keep last 4 weekly backups (1 month)
- Store locally on laptop
- Test restore monthly

---

## 14. Query Examples (Common Use Cases)

### Get All Featured Projects

```php
// Doctrine Query Language
$projects = $em->getRepository(Project::class)
    ->createQueryBuilder('p')
    ->where('p.featured = true')
    ->orderBy('p.displayOrder', 'ASC')
    ->getQuery()
    ->getResult();
```

### Get Images for a Project

```php
$project = $em->getRepository(Project::class)->find(1);
$images = $project->getImages(); // Automatic via relationship
```

### Filter Contacts by Status

```php
$newContacts = $em->getRepository(Contact::class)
    ->createQueryBuilder('c')
    ->where('c.status = :status')
    ->setParameter('status', 'new')
    ->orderBy('c.createdAt', 'DESC')
    ->setMaxResults(50)
    ->getQuery()
    ->getResult();
```

### Count Leads by Source

```php
$stats = $em->getRepository(Contact::class)
    ->createQueryBuilder('c')
    ->select('c.source, COUNT(c.id) as count')
    ->groupBy('c.source')
    ->getQuery()
    ->getResult();
// Result: [['source' => 'form', 'count' => 42], ['source' => 'whatsapp', 'count' => 15]]
```

---

## 15. Database Configuration

### .env File

```dotenv
# database/mysql
DATABASE_URL="mysql://user:password@localhost:3306/bk_pergola?serverVersion=8.0&charset=utf8mb4"
```

### Connection Pooling (Future)

If scaling to multiple servers:
```php
// symfony.yaml - future config
doctrine:
    dbal:
        connections:
            default:
                url: "%env(resolve:DATABASE_URL)%"
                pool_size: 20 # Connection pooling
```

---

## 16. Performance Tips

### Lazy Loading vs Eager Loading

**Lazy (default):**
```php
$project = $repo->find(1);
$images = $project->getImages(); // Executes separate query
```

**Eager (optimized):**
```php
$project = $repo->createQueryBuilder('p')
    ->leftJoin('p.images', 'i')
    ->addSelect('i')
    ->where('p.id = 1')
    ->getQuery()
    ->getOneOrNullResult();
// Images loaded in single query
```

### Use Pagination for Large Lists

```php
$page = 1;
$perPage = 20;
$offset = ($page - 1) * $perPage;

$contacts = $repo->createQueryBuilder('c')
    ->orderBy('c.createdAt', 'DESC')
    ->setFirstResult($offset)
    ->setMaxResults($perPage)
    ->getQuery()
    ->getResult();
```

### Cache Query Results (Future)

```php
$cache = $app->get('cache.app');
$projects = $cache->get('featured_projects', function($item) use ($repo) {
    $item->expiresAfter(3600); // 1 hour
    return $repo->findFeatured();
});
```

---

## 17. Schema Versioning

### Current Schema Version: 1.0

**Last Updated:** June 2026

**Migration History:**
- v1.0 (June 2026): Initial schema (admins, projects, project_images, contacts, team_members)

### Future Versions

- v1.1: Add `projects.price` field (for quotes)
- v1.2: Add `contacts.rating` (client satisfaction)
- v2.0: Add testimonials table

---

## 18. Database Access Control

### Development

- **User:** `root`
- **Host:** `localhost`
- **Database:** `bk_pergola_dev`

### Production (OVHcloud)

- **User:** Auto-managed by OVHcloud
- **Host:** `mysql.ovh.net` (or private host)
- **Database:** `db_bk_pergola_prod`
- **Password:** Stored in `.env` (production server only)

### Security

- ✅ No hardcoded credentials
- ✅ Least privilege (read-only users for reports)
- ✅ SSL/TLS connection to database
- ✅ Regular password rotation

---

## 19. Monitoring & Maintenance

### Check Database Size

```sql
SELECT TABLE_NAME, ROUND((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024, 2) AS 'Size (MB)'
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'bk_pergola';
```

### Slow Query Log

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2; -- Queries > 2 seconds logged
```

### Optimize Tables (Monthly)

```sql
OPTIMIZE TABLE admins;
OPTIMIZE TABLE projects;
OPTIMIZE TABLE project_images;
OPTIMIZE TABLE contacts;
OPTIMIZE TABLE team_members;
```

---

## 20. Next Steps

1. **Generate migrations:** `php bin/console make:migration`
2. **Apply schema:** `php bin/console doctrine:migrations:migrate`
3. **Seed data:** Insert sample records (see section 12)
4. **Test queries:** Use repository methods in controllers
5. **Monitor:** Check slow queries, backup regularly

---

**Database Version:** 1.0  
**Last Updated:** June 2026  
**Status:** Production-ready
