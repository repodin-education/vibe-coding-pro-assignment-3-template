# Pro Assignment 3: Database Integration

## Learning Objectives

By completing this assignment, you will:

- Understand database concepts and data persistence
- Learn to set up and connect to a database
- Practice CRUD operations (Create, Read, Update, Delete)
- Design and implement database schemas
- Use ORMs or query builders for database operations
- Understand data validation and error handling
- Gain experience with database tools (SQLite, PostgreSQL, or MongoDB)

---

## Prerequisites

- Completed at least Assignment 2 (E2E Hello World)
- Working server and client application
- Understanding of Node.js + Express basics
- Cursor AI installed and configured
- Basic understanding of data structures and JSON

---

## Overview

This is a **pro-level bonus assignment** worth **10 bonus points**. It's optional but highly recommended for students who want to learn data persistence and database integration.

**Goal:** Add database integration to your application to store and retrieve data persistently.

---

## Instructions

### Step 1: Choose Your Database

Select a database based on your needs:

- **SQLite** (recommended for beginners) - File-based, no setup needed
- **PostgreSQL** - Powerful, production-ready (requires setup)
- **MongoDB** - NoSQL, document-based (requires setup)

**Recommendation:** Start with **SQLite** - it's the easiest to set up and perfect for learning.

### Step 2: Choose Your Database Tool

Select a tool for working with your database:

- **better-sqlite3** (SQLite) - Simple, synchronous
- **pg** (PostgreSQL) - Official PostgreSQL client
- **Prisma** (ORM) - Modern, type-safe ORM
- **Sequelize** (ORM) - Popular, feature-rich
- **mongoose** (MongoDB) - MongoDB ODM

**Recommendation:** Start with **better-sqlite3** for SQLite - it's very easy to use.

### Step 3: Install Database Dependencies

1. **Install database library:**

   **Node.js (SQLite):**

   ```bash
   npm install better-sqlite3
   ```

   **Node.js (PostgreSQL):**

   ```bash
   npm install pg
   ```

2. **Update package.json:**

   ```json
   {
     "dependencies": {
       "better-sqlite3": "^9.0.0"
     }
   }
   ```

### Step 4: Design Your Database Schema

1. **Choose what to store:**

   - Messages (from your Hello World app)
   - User data (if you have users)
   - Application state
   - Any data that should persist

2. **Design your schema:**

   **Example: Store Messages**

   ```sql
   CREATE TABLE messages (
     id INTEGER PRIMARY KEY AUTOINCREMENT,
     text TEXT NOT NULL,
     created_at DATETIME DEFAULT CURRENT_TIMESTAMP
   );
   ```

   **Example: Store Users (if adding auth later)**

   ```sql
   CREATE TABLE users (
     id INTEGER PRIMARY KEY AUTOINCREMENT,
     username TEXT UNIQUE NOT NULL,
     email TEXT UNIQUE NOT NULL,
     created_at DATETIME DEFAULT CURRENT_TIMESTAMP
   );
   ```

### Step 5: Set Up Database Connection

1. **Create database connection file:**

   **Node.js (SQLite):**

   ```javascript
   // server/db.js
   const Database = require('better-sqlite3');
   const db = new Database('app.db');

   // Create tables
   db.exec(`
     CREATE TABLE IF NOT EXISTS messages (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       text TEXT NOT NULL,
       created_at DATETIME DEFAULT CURRENT_TIMESTAMP
     )
   `);

   module.exports = db;
   ```

### Step 6: Implement CRUD Operations

1. **Create (Insert) data:**

   **Node.js:**

   ```javascript
   // server/db.js
   function createMessage(text) {
     const stmt = db.prepare('INSERT INTO messages (text) VALUES (?)');
     const result = stmt.run(text);
     return result.lastInsertRowid;
   }
   ```

2. **Read (Select) data:**

   **Node.js:**

   ```javascript
   function getAllMessages() {
     return db.prepare('SELECT * FROM messages ORDER BY created_at DESC').all();
   }

   function getMessageById(id) {
     return db.prepare('SELECT * FROM messages WHERE id = ?').get(id);
   }
   ```

3. **Update data:**

   **Node.js:**

   ```javascript
   function updateMessage(id, text) {
     const stmt = db.prepare('UPDATE messages SET text = ? WHERE id = ?');
     return stmt.run(text, id).changes > 0;
   }
   ```

4. **Delete data:**

   **Node.js:**

   ```javascript
   function deleteMessage(id) {
     const stmt = db.prepare('DELETE FROM messages WHERE id = ?');
     return stmt.run(id).changes > 0;
   }
   ```

### Step 7: Integrate Database with Your API

1. **Update your API endpoints:**

   **Node.js:**

   ```javascript
   // server/index.js
   const db = require('./db');

   // GET /api/messages - Get all messages
   app.get('/api/messages', (req, res) => {
     const messages = db.getAllMessages();
     res.json(messages);
   });

   // POST /api/messages - Create new message
   app.post('/api/messages', (req, res) => {
     const { text } = req.body;
     if (!text) {
       return res.status(400).json({ error: 'Text is required' });
     }
     const id = db.createMessage(text);
     res.json({ id, text, success: true });
   });

   // GET /api/messages/:id - Get message by ID
   app.get('/api/messages/:id', (req, res) => {
     const message = db.getMessageById(req.params.id);
     if (!message) {
       return res.status(404).json({ error: 'Message not found' });
     }
     res.json(message);
   });
   ```

### Step 8: Update Client to Use Database

1. **Update client to fetch from database:**

   ```javascript
   // client/index.html
   async function loadMessages() {
     const response = await fetch('/api/messages');
     const messages = await response.json();
     // Display messages
   }

   async function addMessage(text) {
     const response = await fetch('/api/messages', {
       method: 'POST',
       headers: { 'Content-Type': 'application/json' },
       body: JSON.stringify({ text }),
     });
     const result = await response.json();
     // Refresh messages
     loadMessages();
   }
   ```

### Step 9: Add Data Validation

1. **Validate input:**

   **Node.js:**

   ```javascript
   function createMessage(text) {
     // Validate
     if (!text || text.trim().length === 0) {
       throw new Error('Text cannot be empty');
     }
     if (text.length > 500) {
       throw new Error('Text too long (max 500 characters)');
     }

     const stmt = db.prepare('INSERT INTO messages (text) VALUES (?)');
     return stmt.run(text.trim()).lastInsertRowid;
   }
   ```

### Step 10: Document Your Database

1. **Update README.md:**

   - Add "Database" section
   - Document schema
   - Explain CRUD operations
   - Include example queries
   - Document how to set up database

2. **Create database schema documentation:**

   ```markdown
   ## Database Schema

   ### messages table

   - `id`: INTEGER PRIMARY KEY
   - `text`: TEXT NOT NULL
   - `created_at`: DATETIME DEFAULT CURRENT_TIMESTAMP
   ```

---

## Requirements

### Required

- [ ] Database installed and configured
- [ ] Database schema created (at least one table)
- [ ] CRUD operations implemented (Create, Read, Update, Delete)
- [ ] API endpoints use database (not just in-memory data)
- [ ] Data persists between server restarts
- [ ] Data validation implemented
- [ ] Database documented in README.md

### Optional

- [ ] Multiple tables with relationships
- [ ] Database migrations (if using ORM)
- [ ] Indexes for performance
- [ ] Error handling for database operations
- [ ] Database backup strategy

---

## Acceptance Criteria

Your submission will be evaluated based on:

- [ ] Database successfully integrated
- [ ] At least one table created
- [ ] CRUD operations working
- [ ] API endpoints use database
- [ ] Data persists (survives server restart)
- [ ] Data validation implemented
- [ ] Database schema documented
- [ ] Example queries demonstrated
- [ ] Changes committed and pushed to GitHub

---

## Submission Requirements

1. **Database file:** SQLite database file (if using SQLite) or connection info
2. **Database code:** All database code in `server/db.js`
3. **Documentation:** README.md updated with database section
4. **Commit:** All changes committed and pushed to GitHub

**Commit message example:**

```bash
git commit -m "Pro Assignment 3: Database Integration"
```

---

## Grading Rubric

See [Grading Rubrics](https://repodin-education.github.io/vibe-coding-materials/grading-rubrics.html) for detailed criteria.

**Total Points:** 10 bonus points

- **Database setup:** 3 points
  - Database installed and configured: 1 point
  - Schema created: 1 point
  - Connection working: 1 point
- **CRUD operations:** 4 points
  - Create: 1 point
  - Read: 1 point
  - Update: 1 point
  - Delete: 1 point
- **Integration:** 2 points
  - API uses database: 1 point
  - Data persists: 1 point
- **Documentation:** 1 point
  - Database documented: 0.5 points
  - Schema explained: 0.5 points

---

## Tips for Success

- **Start with SQLite:** Easiest to set up, no server needed
- **Use Cursor AI:** Ask Cursor to generate database code
- **Test incrementally:** Test each CRUD operation separately
- **Handle errors:** Database operations can fail, handle errors gracefully
- **Validate data:** Always validate input before storing
- **Keep it simple:** Start with one table, expand later
- **Document as you go:** Write down what you learn

---

## Example Database Structure

```
your-project/
├── server/
│   ├── index.js
│   └── db.js
├── client/
│   └── index.html
├── app.db (SQLite database file)
├── package.json
└── README.md
```

**Example SQLite schema:**

```sql
-- messages table
CREATE TABLE messages (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  text TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- users table (optional)
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  username TEXT UNIQUE NOT NULL,
  email TEXT UNIQUE NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## Common Database Issues

### Issue: Database file not found

**Solutions:**

- Check file path is correct
- Ensure database file is created before use
- Use absolute paths or relative to project root

### Issue: SQL syntax errors

**Solutions:**

- Check SQL syntax carefully
- Use parameterized queries (prevent SQL injection)
- Test queries in database tool first

### Issue: Data not persisting

**Solutions:**

- Ensure you're committing transactions
- Check database file permissions
- Verify you're using the same database file

### Issue: Connection errors

**Solutions:**

- Check database is running (if using PostgreSQL/MongoDB)
- Verify connection string is correct
- Check firewall/network settings

---

## Getting Help

- Ask questions in the help channel
- Review database documentation:
  - [SQLite Documentation](https://www.sqlite.org/docs.html)
  - [PostgreSQL Documentation](https://www.postgresql.org/docs/)
  - [MongoDB Documentation](https://www.mongodb.com/docs/)
- Check [FAQ](https://repodin-education.github.io/vibe-coding-materials/faq.html)
- Review [Student Guide](https://repodin-education.github.io/vibe-coding-materials/student-guide.html)
- Contact your teacher if needed

---

## Resources

**Database Tools:**

- [SQLite Browser](https://sqlitebrowser.org/) - Visual database editor
- [DB Browser for SQLite](https://sqlitebrowser.org/) - GUI tool
- [PostgreSQL GUI Tools](https://www.postgresql.org/download/products/)

**Learning Resources:**

- [SQL Tutorial](https://www.w3schools.com/sql/)
- [Database Design Basics](https://www.lucidchart.com/pages/database-diagram/database-design)
- [CRUD Operations Guide](https://www.codecademy.com/article/what-is-crud)

---

## Document History

| Version | Date       | Author                 | Changes         |
| ------- | ---------- | ---------------------- | --------------- |
| 1.0     | 2025-12-25 | RepodIn Education Team | Initial version |
| 1.1     | 2025-12-29 | RepodIn Education Team | Minor text edits |

---

**Next Review Date:** 2026-03-20
