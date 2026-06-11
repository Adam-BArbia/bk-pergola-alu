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
- **Flexibility:** Gallery items can be standalone or part of projects

### Tables at a Glance

| Table | Purpose | Records |
|-------|---------|---------|
| `admins` | Admin users | ~5 (estimated) |
| `projects` | Portfolio project metadata | ~20-50 |
| `gallery_items` | Individual photos (standalone or in projects) | ~150-300 |
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

**Purpose:** Portfolio project metadata (grouping for related gallery items)

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
    private string $title; // FR - Project name

    #[ORM\Column(type: 'text', nullable: true)]
    private ?string $description = null; // FR - Project details

    #[ORM\Column(type: 'string', length: 50)]
    private string $category; // 'pergola', 'structure', 'custom'

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $location = null; // Where built

    #[ORM\Column(type: 'date', nullable: true)]
    private ?\DateTimeInterface $completionDate = null;

    #[ORM\Column(type: 'boolean')]
    private bool $featured = false; // Show on homepage

    #[ORM\Column(type: 'integer')]
    private int $displayOrder = 0; // Sort order

    #[ORM\OneToMany(mappedBy: 'project', targetEntity: GalleryItem::class, cascade: ['persist', 'remove'])]
    #[ORM\OrderBy(['position' => 'ASC'])]
    private Collection $galleryItems; // Related photos

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
    description LONGTEXT NULL,
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
- `title` - Project name (FR) - Optional if standalone photos used
- `description` - Project details (FR) - Optional
- `category` - Type: 'pergola', 'structure', 'custom'
- `location` - Where built (city/region)
- `completion_date` - When completed
- `featured` - Show on homepage
- `display_order` - Sort order in gallery
- `created_at` - When added
- `updated_at` - Last modified

**Indexes:**
- `idx_featured` - Find featured projects quickly
- `idx_category` - Filter by category
- `idx_created_at` - Sort by date

**Use Cases:**
- Full project: "Pergola Tunis 2025" with 5+ photos
- Minimal project: Just a container for related photos (title + 1-2 photos)
- Optional: Leave title/description empty for pure gallery organization

---

## 4. Table: `gallery_items` (New - Replaces project_images)

**Purpose:** Individual photos in the gallery (standalone or grouped by project)

**Key Difference from Old Structure:**
- Old: `project_images` were tied to projects only
- New: `gallery_items` can be standalone (project_id = null) OR linked to projects
- Enables Pinterest-like flexible gallery

**Doctrine Entity:**
```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: GalleryItemRepository::class)]
#[ORM\Table(name: 'gallery_items')]
#[ORM\Index(name: 'idx_project_id', columns: ['project_id'])]
#[ORM\Index(name: 'idx_featured', columns: ['featured'])]
#[ORM\Index(name: 'idx_keywords', columns: ['keywords'])]
#[ORM\Index(name: 'idx_created_at', columns: ['created_at'])]
class GalleryItem
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;

    #[ORM\ManyToOne(targetEntity: Project::class, inversedBy: 'galleryItems')]
    #[ORM\JoinColumn(name: 'project_id', referencedColumnName: 'id', onDelete: 'CASCADE', nullable: true)]
    private ?Project $project = null; // NULL = standalone photo

    #[ORM\Column(type: 'string', length: 255)]
    private string $imagePath; // Full path: /uploads/gallery/2025/pergola-01.jpg

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $thumbnailPath = null; // Optimized thumb

    #[ORM\Column(type: 'text', nullable: true)]
    private ?string $description = null; // Photo description (FR)

    #[ORM\Column(type: 'string', length: 255, nullable: true)]
    private ?string $altText = null; // SEO alt text

    #[ORM\Column(type: 'text', nullable: true)]
    private ?string $keywords = null; // Comma-separated: "pergola,aluminium,terrasse"

    #[ORM\Column(type: 'integer')]
    private int $position = 0; // Order within project (if part of project)

    #[ORM\Column(type: 'integer', nullable: true)]
    private ?int $width = null; // Image width pixels

    #[ORM\Column(type: 'integer', nullable: true)]
    private ?int $height = null; // Image height pixels

    #[ORM\Column(type: 'boolean')]
    private bool $featured = false; // Highlight in gallery

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $uploadedAt;

    // Getters and setters...
}
```

**SQL:**
```sql
CREATE TABLE gallery_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    project_id INT NULL,
    image_path VARCHAR(255) NOT NULL,
    thumbnail_path VARCHAR(255) NULL,
    description LONGTEXT NULL,
    alt_text VARCHAR(255) NULL,
    keywords VARCHAR(255) NULL,
    position INT DEFAULT 0,
    width INT NULL,
    height INT NULL,
    featured BOOLEAN DEFAULT FALSE,
    uploaded_at DATETIME NOT NULL,
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    INDEX idx_project_id (project_id),
    INDEX idx_featured (featured),
    INDEX idx_keywords (keywords),
    INDEX idx_created_at (uploaded_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Fields:**
- `id` - Primary key
- `project_id` - Foreign key to projects (NULL = standalone photo)
- `image_path` - Full path to image file
- `thumbnail_path` - Optimized thumbnail for gallery grid
- `description` - Photo caption (FR) - Shows in lightbox
- `alt_text` - SEO + accessibility
- `keywords` - Searchable tags: "pergola,aluminium,moderne,terrasse"
- `position` - Order within project (if grouped)
- `width`, `height` - For responsive sizing
- `featured` - Pin to top of gallery
- `uploaded_at` - Upload timestamp

**Indexes:**
- `idx_project_id` - Find photos in project
- `idx_featured` - Featured photos for homepage
- `idx_keywords` - Search by keywords
- `idx_created_at` - Sort by date

**Examples:**

Standalone photo:
```sql
INSERT INTO gallery_items (project_id, image_path, description, keywords, featured, uploaded_at)
VALUES (
    NULL, -- No project
    '/uploads/gallery/2025/before-after-01.jpg',
    'Transformation avant/après d\'une installation',
    'pergola,aluminium,transformation,avant-après',
    TRUE,
    NOW()
);
```

Project photo:
```sql
INSERT INTO gallery_items (project_id, image_path, description, keywords, position, uploaded_at)
VALUES (
    1, -- Part of project 1
    '/uploads/gallery/2025/pergola-main.jpg',
    'Vue générale de la pergola installée',
    'pergola,vue-générale',
    0,
    NOW()
);
```

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

---

## 7. Entity Relationships Diagram (Updated)

```
admins (1) ────────────────────────────────────── (many) projects
           [admin creates/manages projects]

projects (1) ────────────────────────────────────── (many) gallery_items
            [1 project has many gallery items]

gallery_items
  ├─ (0..1) → projects [optional - can be standalone]
  └─ searchable by keywords

contacts [standalone - no FK]

team_members [standalone - no FK]
```

**Key Change:**
- Old: `project_images` mandatory tied to projects
- New: `gallery_items` optional tied to projects (enables standalone photos)

---

## 8. Gallery Logic (Frontend/Backend)

### Pinterest Feed Display

**GET /api/gallery** - All gallery items (for Pinterest feed)
```php
// Repository Query
$items = $em->getRepository(GalleryItem::class)
    ->createQueryBuilder('g')
    ->orderBy('g.featured', 'DESC') // Featured first
    ->addOrderBy('g.uploadedAt', 'DESC') // Then by date
    ->getQuery()
    ->getResult();

// Response: Array of 150+ gallery items (thumbnails)
```

**GET /api/gallery/{id}** - Single item with related project photos
```php
// Get the item
$item = $em->getRepository(GalleryItem::class)->find($id);

// If has project, get related images
$relatedImages = [];
if ($item->getProject()) {
    $relatedImages = $em->getRepository(GalleryItem::class)
        ->createQueryBuilder('g')
        ->where('g.project = :project')
        ->andWhere('g.id != :id')
        ->setParameter('project', $item->getProject())
        ->setParameter('id', $id)
        ->orderBy('g.position', 'ASC')
        ->getQuery()
        ->getResult();
}

// Response: { item, description, keywords, relatedImages }
```

**GET /api/gallery/search?q=pergola** - Search by keywords
```php
$query = $request->query->get('q');
$items = $em->getRepository(GalleryItem::class)
    ->createQueryBuilder('g')
    ->where('g.keywords LIKE :query')
    ->orWhere('g.description LIKE :query')
    ->setParameter('query', '%' . $query . '%')
    ->orderBy('g.uploadedAt', 'DESC')
    ->getQuery()
    ->getResult();
```

---

## 9. Admin Gallery Management (CRUD)

### Admin Can:

1. **Upload standalone photo:**
   - Image file
   - Description (FR)
   - Keywords (comma-separated)
   - Featured (yes/no)
   - *No project selected*

2. **Add photo to existing project:**
   - Image file
   - Select project
   - Description
   - Keywords
   - Position in project (drag-drop order)

3. **Create project + add photos:**
   - Project title (FR)
   - Project description (FR)
   - Category
   - Location
   - Completion date
   - Then upload 1+ photos

---

## 10. Database Migrations (Updated)

### Generate Initial Schema

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

### Migration File

```php
<?php
namespace DoctrineMigrations;

use Doctrine\DBAL\Schema\Schema;
use Doctrine\Migrations\AbstractMigration;

final class Version20260611000000 extends AbstractMigration
{
    public function up(Schema $schema): void
    {
        $this->addSql('CREATE TABLE admins (...)');
        $this->addSql('CREATE TABLE projects (...)');
        $this->addSql('CREATE TABLE gallery_items (...)'); // NEW
        $this->addSql('CREATE TABLE contacts (...)');
        $this->addSql('CREATE TABLE team_members (...)');
    }

    public function down(Schema $schema): void
    {
        $this->addSql('DROP TABLE IF EXISTS gallery_items'); // NEW
        $this->addSql('DROP TABLE IF EXISTS contacts');
        $this->addSql('DROP TABLE IF EXISTS projects');
        $this->addSql('DROP TABLE IF EXISTS admins');
        $this->addSql('DROP TABLE IF EXISTS team_members');
    }
}
```

---

## 11. Indexes for Performance (Updated)

| Table | Index | Purpose |
|-------|-------|---------|
| admins | idx_username | Login lookup |
| projects | idx_featured | Homepage projects |
| projects | idx_category | Filter by type |
| projects | idx_created_at | Sort by date |
| gallery_items | idx_project_id | Find photos in project |
| gallery_items | idx_featured | Featured gallery items |
| gallery_items | idx_keywords | Search functionality |
| gallery_items | idx_created_at | Sort by date |
| contacts | idx_status | Dashboard filters |
| contacts | idx_source | Channel breakdown |
| team_members | idx_position | Sort on page |

---

## 12. Sample Data (Updated)

### Add Standalone Gallery Item

```sql
INSERT INTO gallery_items (project_id, image_path, thumbnail_path, description, keywords, featured, uploaded_at)
VALUES (
    NULL, -- Standalone
    '/uploads/gallery/2025/before-after-pergola.jpg',
    '/uploads/thumbs/2025/before-after-pergola.jpg',
    'Transformation d\'une terrasse avec pergola aluminium',
    'pergola,aluminium,terrasse,transformation,avant-après',
    TRUE,
    NOW()
);
```

### Add Project with Photos

```sql
-- Create project
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

-- Add photos to project (project_id = 1)
INSERT INTO gallery_items (project_id, image_path, description, keywords, position, uploaded_at)
VALUES 
(1, '/uploads/gallery/2025/pergola-main.jpg', 'Vue générale de la pergola', 'pergola,vue-générale', 0, NOW()),
(1, '/uploads/gallery/2025/pergola-detail.jpg', 'Détail de la structure', 'pergola,structure,détail', 1, NOW()),
(1, '/uploads/gallery/2025/pergola-evening.jpg', 'Pergola en soirée', 'pergola,éclairage', 2, NOW());
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

## 13. Query Examples (Updated)

### Get All Gallery Items (Pinterest Feed)

```php
$items = $em->getRepository(GalleryItem::class)
    ->createQueryBuilder('g')
    ->select('g.id, g.imagePath, g.thumbnailPath, g.description')
    ->orderBy('g.featured', 'DESC')
    ->addOrderBy('g.uploadedAt', 'DESC')
    ->setMaxResults(50) // Paginate
    ->getQuery()
    ->getResult();
```

### Search Gallery by Keywords

```php
$searchTerm = 'pergola';
$results = $em->getRepository(GalleryItem::class)
    ->createQueryBuilder('g')
    ->where('g.keywords LIKE :query')
    ->orWhere('g.description LIKE :query')
    ->setParameter('query', '%' . $searchTerm . '%')
    ->orderBy('g.uploadedAt', 'DESC')
    ->getQuery()
    ->getResult();
```

### Get Photos for Single Project

```php
$project = $em->getRepository(Project::class)->find(1);
$photos = $em->getRepository(GalleryItem::class)
    ->createQueryBuilder('g')
    ->where('g.project = :project')
    ->setParameter('project', $project)
    ->orderBy('g.position', 'ASC')
    ->getQuery()
    ->getResult();
```

### Get Related Photos When Viewing a Single Item

```php
$item = $em->getRepository(GalleryItem::class)->find($itemId);
$relatedPhotos = null;

if ($item->getProject()) {
    $relatedPhotos = $em->getRepository(GalleryItem::class)
        ->createQueryBuilder('g')
        ->where('g.project = :project')
        ->andWhere('g.id != :currentId')
        ->setParameter('project', $item->getProject())
        ->setParameter('currentId', $itemId)
        ->orderBy('g.position', 'ASC')
        ->getQuery()
        ->getResult();
}
```

---

## 14. Backup & Recovery Strategy

### Automated Backup

OVHcloud provides 24-hour automatic backup (recovery point if data loss).

### Manual Backup (Weekly)

```bash
mysqldump -u username -p database_name > backup_$(date +%Y%m%d).sql
mysql -u username -p database_name < backup_20260611.sql
```

---

## 15. Performance Tips

### Lazy Loading vs Eager Loading

**Lazy (default):**
```php
$item = $repo->find(1);
$project = $item->getProject(); // Separate query
```

**Eager (optimized):**
```php
$item = $repo->createQueryBuilder('g')
    ->leftJoin('g.project', 'p')
    ->addSelect('p')
    ->where('g.id = 1')
    ->getQuery()
    ->getOneOrNullResult();
```

### Pagination for Large Galleries

```php
$page = 1;
$perPage = 30;
$offset = ($page - 1) * $perPage;

$items = $repo->createQueryBuilder('g')
    ->orderBy('g.uploadedAt', 'DESC')
    ->setFirstResult($offset)
    ->setMaxResults($perPage)
    ->getQuery()
    ->getResult();
```

---

## 16. Database Access Control

### Development
- **User:** `root`
- **Host:** `localhost`
- **Database:** `bk_pergola_dev`

### Production (OVHcloud)
- **User:** Auto-managed by OVHcloud
- **Host:** `mysql.ovh.net`
- **Database:** `db_bk_pergola_prod`

---

## 17. Monitoring & Maintenance

### Check Database Size

```sql
SELECT TABLE_NAME, ROUND((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024, 2) AS 'Size (MB)'
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'bk_pergola';
```

### Optimize Tables (Monthly)

```sql
OPTIMIZE TABLE admins;
OPTIMIZE TABLE projects;
OPTIMIZE TABLE gallery_items;
OPTIMIZE TABLE contacts;
OPTIMIZE TABLE team_members;
```

---

## 18. Key Differences: Old vs New Schema

| Aspect | Old (project_images) | New (gallery_items) |
|--------|---------------------|-------------------|
| Photos | Must belong to project | Can be standalone OR in project |
| Searchability | Not searchable | Keywords indexed for search |
| Featured | On project level | Individual photo level |
| Pinterest Layout | Not ideal | Perfect fit |
| Admin Workflow | Add project first, then photos | Add photos standalone or to project |
| Flexibility | Rigid structure | Flexible, multi-use |

---

## 19. Next Steps

1. **Migrate old schema** (if needed): Update from project_images to gallery_items
2. **Generate migrations:** `php bin/console make:migration`
3. **Apply schema:** `php bin/console doctrine:migrations:migrate`
4. **Create repositories:** GalleryItemRepository with search/filter methods
5. **Build API endpoints:** GET gallery, GET gallery/:id, search, etc.
6. **Update ADMIN_PANEL.md:** New gallery management workflow

---

**Database Version:** 2.0 (Updated with unified gallery_items)  
**Last Updated:** June 2026  
**Status:** Production-ready
