# Development Guide

## Quick Start

```bash
# Clone and setup
git clone https://github.com/Mehedii364/Wafa-Nearby.git
cd Wafa-Nearby
npm install

# Setup environment
cp .env.example .env

# Start development environment
docker-compose up -d

# Run migrations
npm run migrate

# Start dev servers
npm run dev
```

## Project Structure

```
packages/
├── frontend/      # React web app (port 3000)
├── backend/       # Express API (port 5000)
└── shared/        # Shared types & utilities
```

## Frontend Development

### Start Dev Server

```bash
cd packages/frontend
npm run dev
```

Open http://localhost:3000

### Key Scripts

```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run preview      # Preview production build
npm run lint         # Run ESLint
npm run type-check   # TypeScript check
npm run test         # Run tests
```

### File Structure

```
src/
├── components/     # Reusable components
├── pages/          # Route pages
├── services/       # API client & external services
├── store/          # Redux state management
├── hooks/          # Custom React hooks
├── types/          # TypeScript types
├── utils/          # Utilities
└── App.tsx
```

### Component Template

```typescript
// src/components/MyComponent/MyComponent.tsx
import React from 'react';
import styles from './MyComponent.module.css';

interface Props {
  title: string;
  onClick?: () => void;
}

export const MyComponent: React.FC<Props> = ({ title, onClick }) => {
  return (
    <div className={styles.container}>
      <h2>{title}</h2>
      <button onClick={onClick}>Click me</button>
    </div>
  );
};
```

## Backend Development

### Start Dev Server

```bash
cd packages/backend
npm run dev
```

Server runs on http://localhost:5000

### Key Scripts

```bash
npm run dev          # Start with nodemon
npm run build        # Build TypeScript
npm run start        # Run compiled JS
npm run lint         # Run ESLint
npm run test         # Run tests
npm run migrate      # Run migrations
```

### File Structure

```
src/
├── controllers/     # Request handlers
├── services/        # Business logic
├── models/          # Database models
├── routes/          # API routes
├── middleware/      # Express middleware
├── utils/           # Utilities
├── types/           # TypeScript types
├── config/          # Configuration
└── index.ts
```

### Creating an API Endpoint

1. **Create Model** (src/models/event.ts):

```typescript
import { DataTypes, Model } from 'sequelize';
import { sequelize } from '../config/database';

export class Event extends Model {
  public id: string;
  public title: string;
  public description: string;
}

Event.init(
  {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true,
    },
    title: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    description: DataTypes.TEXT,
  },
  { sequelize, modelName: 'events' }
);
```

2. **Create Service** (src/services/eventService.ts):

```typescript
import { Event } from '../models/event';

export class EventService {
  async getEvents(limit: number, offset: number) {
    return Event.findAndCountAll({ limit, offset });
  }

  async createEvent(data: Partial<Event>) {
    return Event.create(data);
  }
}

export const eventService = new EventService();
```

3. **Create Controller** (src/controllers/eventController.ts):

```typescript
import { Request, Response } from 'express';
import { eventService } from '../services/eventService';

export class EventController {
  async getEvents(req: Request, res: Response) {
    const { limit = 20, offset = 0 } = req.query;
    const events = await eventService.getEvents(Number(limit), Number(offset));
    res.json(events);
  }

  async createEvent(req: Request, res: Response) {
    const event = await eventService.createEvent(req.body);
    res.status(201).json(event);
  }
}
```

4. **Add Routes** (src/routes/events.ts):

```typescript
import { Router } from 'express';
import { authenticate } from '../middleware/auth';
import { EventController } from '../controllers/eventController';

const router = Router();
const controller = new EventController();

router.get('/', controller.getEvents);
router.post('/', authenticate, controller.createEvent);

export default router;
```

## Database Migrations

### Create Migration

```bash
npm run migrate:create -- create_events_table
```

### Migration Template

```typescript
// migrations/XXX_create_events_table.ts
import { QueryInterface, DataTypes } from 'sequelize';

export const up = async (queryInterface: QueryInterface) => {
  await queryInterface.createTable('events', {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true,
    },
    title: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    created_at: {
      type: DataTypes.DATE,
      allowNull: false,
      defaultValue: DataTypes.NOW,
    },
  });
};

export const down = async (queryInterface: QueryInterface) => {
  await queryInterface.dropTable('events');
};
```

### Run Migrations

```bash
npm run migrate           # Run pending migrations
npm run migrate:undo     # Undo last migration
```

## Testing

### Unit Tests

```bash
cd packages/backend
npm run test
```

### Test Template

```typescript
// src/services/__tests__/eventService.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { eventService } from '../eventService';

describe('EventService', () => {
  beforeEach(() => {
    // Setup
  });

  it('should get events', async () => {
    const result = await eventService.getEvents(10, 0);
    expect(result).toBeDefined();
  });
});
```

## Debugging

### VSCode Debug Configuration

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Backend Debug",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/packages/backend/src/index.ts",
      "preLaunchTask": "npm: build",
      "outFiles": ["${workspaceFolder}/packages/backend/dist/**/*.js"]
    }
  ]
}
```

### Debugging Frontend

Open DevTools in browser (F12) and use React DevTools extension.

### Server Logs

```bash
# Tail logs
cd packages/backend
tail -f logs/app.log

# Check error logs
grep ERROR logs/app.log
```

## Common Tasks

### Add New Feature

```bash
# Create feature branch
git checkout -b feature/my-feature

# Make changes
# Test locally
npm run test

# Commit
git commit -m "feat: add my feature"

# Push
git push origin feature/my-feature

# Create PR
```

### Update Dependencies

```bash
# Check for updates
npm outdated

# Update all
npm update

# Update specific package
npm install package@latest
```

## Troubleshooting

### Port already in use

```bash
# Find process using port
lsof -i :5000

# Kill process
kill -9 <PID>
```

### Database connection issues

```bash
# Test connection
psql -h localhost -U postgres -d wafa_nearby

# Check if PostgreSQL is running
docker-compose ps
```

### Module not found

```bash
# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```
