# RealWorld Conduit API - Workshop Demo Application

This is a TypeScript/Express.js backend implementation of the RealWorld Conduit API specification, used for Claude Code workshop training.

## Application Overview

**Domain:** Blogging platform (Medium.com clone)
**Architecture:** Monolithic REST API with Clean Architecture patterns
**Purpose:** Educational reference application for demonstrating Claude Code capabilities

## Technology Stack

- **Runtime:** Node.js
- **Framework:** Express.js 4.18
- **Language:** TypeScript 5.2
- **ORM:** Prisma 4.16
- **Database:** PostgreSQL 16
- **Testing:** Jest
- **Build Tool:** Nx 17

## Database Schema

**Main Entities:**
- **User** - Authentication, profiles, following relationships
- **Article** - Blog posts with tags and favorites
- **Comment** - Article comments
- **Tag** - Article categorization

**Key Relationships:**
- Users write Articles
- Users comment on Articles
- Users follow other Users
- Users favorite Articles
- Articles have Tags (many-to-many)

## Setup Instructions

### Prerequisites

1. **Node.js** - v18+ recommended
2. **PostgreSQL** - Running on port 5432
3. **Docker** - For PostgreSQL container

### Quick Start

**1. Start PostgreSQL:**
```bash
docker run -e "POSTGRES_PASSWORD=RealWorldPass123!" \
  -e "POSTGRES_DB=conduit" \
  -p 5432:5432 --name workshop-db-ts -d \
  postgres:16-alpine
```

**2. Install dependencies:**
```bash
npm install
```

**3. Configure environment:**

Create `.env` file:
```env
DATABASE_URL=postgresql://postgres:RealWorldPass123!@localhost:5432/conduit
JWT_SECRET=workshop-secret-key-change-in-production
NODE_ENV=development
```

**4. Setup database:**
```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate deploy

# Seed database with demo data
npx prisma db seed
```

**5. Start server:**
```bash
npx nx serve api
```

Server will be available at: **http://localhost:3000**

### Verify Installation

```bash
# Check API health
curl http://localhost:3000/api

# Get articles
curl http://localhost:3000/api/articles | jq '.articlesCount'

# Get tags
curl http://localhost:3000/api/tags
```

Expected: 144 articles, 12 users, 1,728 comments

## Development Commands

### Running the Application

```bash
# Development mode with hot reload
npx nx serve api

# Production build
npm run build

# Run production server
node dist/api/main.js
```

### Database Management

```bash
# Reset database (clean + migrate + seed)
npx prisma migrate reset --force

# Reset without seeding
npx prisma migrate reset --force --skip-seed

# View database in browser
npx prisma studio

# Check migration status
npx prisma migrate status
```

### Testing

```bash
# Run all tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run specific test file
npm test -- article.service.spec.ts

# Run with coverage
npm test -- --coverage
```

### Code Quality

```bash
# Lint code
npm run lint

# Format code
npx prettier --write "src/**/*.ts"
```

## API Endpoints

### Authentication
- `POST /api/users` - Register new user
- `POST /api/users/login` - Login
- `GET /api/user` - Get current user
- `PUT /api/user` - Update user

### Articles
- `GET /api/articles` - List articles (paginated)
- `GET /api/articles/feed` - Get user feed
- `GET /api/articles/:slug` - Get article
- `POST /api/articles` - Create article
- `PUT /api/articles/:slug` - Update article
- `DELETE /api/articles/:slug` - Delete article

### Comments
- `GET /api/articles/:slug/comments` - Get comments
- `POST /api/articles/:slug/comments` - Add comment
- `DELETE /api/articles/:slug/comments/:id` - Delete comment

### Favorites
- `POST /api/articles/:slug/favorite` - Favorite article
- `DELETE /api/articles/:slug/favorite` - Unfavorite article

### Tags
- `GET /api/tags` - Get all tags

### Profiles
- `GET /api/profiles/:username` - Get profile
- `POST /api/profiles/:username/follow` - Follow user
- `DELETE /api/profiles/:username/follow` - Unfollow user

## Workshop Usage Notes

### Common Workshop Tasks

**Explore codebase:**
```bash
# Understand project structure
ls -la src/app/routes/

# View domain models
cat src/prisma/schema.prisma
```

**Test coverage analysis:**
```bash
npm test -- --coverage --collectCoverageFrom="src/app/**/*.ts"
```

**Database debugging:**
```bash
docker exec -it postgres-workshop-ts psql -U postgres -d conduit
```

**Performance profiling:**
```bash
# Check for N+1 queries in logs
# Enable Prisma query logging in code
```

### Workshop Scenarios

This application is used to demonstrate:

1. **Test Coverage Improvement** - Add tests for Articles service
2. **N+1 Query Detection** - Find and fix Prisma query inefficiencies
3. **New Feature Implementation** - Add article bookmarking
4. **Database Performance** - Optimize slow queries with indexes
5. **API Debugging** - Trace request flow through layers
6. **Code Review** - Review PR changes for quality and security

## Architecture Layers

```
src/
├── app/
│   ├── routes/           # API routes and controllers
│   │   ├── article/      # Article CRUD, favorites
│   │   ├── auth/         # Authentication, registration
│   │   ├── comment/      # Comment management
│   │   ├── profile/      # User profiles, follows
│   │   └── tag/          # Tag operations
│   ├── utils/            # Shared utilities
│   └── middleware/       # Express middleware
├── prisma/
│   ├── schema.prisma     # Database schema
│   ├── migrations/       # SQL migration files
│   └── seed.ts          # Demo data generator
└── tests/                # Unit and integration tests
```

## Database Schema Details

### User Table
- Authentication (email, password)
- Profile (username, bio, image)
- Relationships (followers, following)

### Article Table
- Content (title, description, body)
- Metadata (slug, createdAt, updatedAt)
- Relationships (author, tags, favorites, comments)

### Comment Table
- Content (body)
- Metadata (createdAt, updatedAt)
- Relationships (author, article)

### Tag Table
- Simple tag names
- Many-to-many with Articles

## Known Issues

### Seed Script (Fixed)
- **Original issue:** Race condition with duplicate tags when creating articles in parallel
- **Fix:** Articles now created sequentially per user
- **Commit:** See latest commit on VeryBigThings fork

### Security Notes
- Demo uses simple JWT authentication
- Passwords hashed with bcryptjs
- Demo data marked with `demo: true` flag
- Not production-ready (educational only)

## Troubleshooting

### Database Connection Issues
```bash
# Check PostgreSQL is running
docker ps | grep workshop-db-ts

# Check logs
docker logs workshop-db-ts

# Restart container
docker restart workshop-db-ts
```

### Port Conflicts
```bash
# Check if port 3000 is in use
lsof -i :3000

# Kill process using port
kill -9 <PID>
```

### Migration Issues
```bash
# Reset everything
npx prisma migrate reset --force

# Or manually drop database
docker exec -it postgres-workshop-ts psql -U postgres -c "DROP DATABASE conduit;"
docker exec -it postgres-workshop-ts psql -U postgres -c "CREATE DATABASE conduit;"
npx prisma migrate deploy
```

## Resources

**Official Documentation:**
- RealWorld Spec: https://realworld-docs.netlify.app/
- Prisma Docs: https://www.prisma.io/docs
- Express.js: https://expressjs.com/
- TypeScript: https://www.typescriptlang.org/

**Repositories:**
- Original: https://github.com/gothinkster/node-express-realworld-example-app
- VeryBigThings Fork: https://github.com/VeryBigThings/workshop-demo-ts

## Workshop Integration

This application mirrors the C# demo app (eShopOnWeb) for World Emblem workshops:

| Aspect | eShopOnWeb (C#) | Conduit (TypeScript) |
|--------|-----------------|---------------------|
| Domain | E-commerce | Blogging platform |
| Framework | ASP.NET Core | Express.js |
| ORM | Entity Framework | Prisma |
| Database | SQL Server | PostgreSQL |
| Tests | xUnit | Jest |
| Language | C# | TypeScript |

Both demonstrate Clean Architecture, have tests, use relational databases, and provide realistic workshop scenarios.
