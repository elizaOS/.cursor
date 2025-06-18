---
description: Database, Postgres, PGLite, Drizzle, creating new tables, adding data to Eliza
globs:
alwaysApply: false
---

# ElizaOS Database System

The ElizaOS database system provides persistent storage capabilities for agents through a flexible adapter-based architecture. It handles memory storage, entity relationships, knowledge management, and more.

## Core Concepts

### Architecture

```
┌─────────────────────────┐
│     Agent Runtime       │
└────────────┬────────────┘
             │
┌────────────▼────────────┐
│   IDatabaseAdapter      │
└────────────┬────────────┘
             │
┌────────────▼────────────┐
│   BaseDrizzleAdapter    │
└────────────┬────────────┘
             │
    ┌────────┴────────┐
    ▼                 ▼
┌──────────┐   ┌──────────┐
│ PGLite   │   │PostgreSQL│
│ Adapter  │   │ Adapter  │
└──────────┘   └──────────┘
```

### Current Adapters

| Adapter        | Best For                    | Key Features                                |
| -------------- | --------------------------- | ------------------------------------------- |
| **PGLite**     | Local development & testing | Lightweight PostgreSQL in Node.js process   |
| **PostgreSQL** | Production deployments      | Full PostgreSQL with vector search, scaling |

## Database Operations

### Entity System

```typescript
// Create entity
await adapter.createEntity(entity);

// Get entity
const entity = await adapter.getEntityById(id);
const entities = await adapter.getEntitiesForRoom(roomId);

// Update entity
await adapter.updateEntity(entity);

// Components
await adapter.createComponent(component);
const component = await adapter.getComponent(entityId, type);
await adapter.updateComponent(component);
await adapter.deleteComponent(componentId);
```

### Memory Management

```typescript
// Create memory
const memoryId = await adapter.createMemory(memory, 'messages');

// Get memories
const memories = await adapter.getMemories({
  roomId,
  count: 50,
  unique: true,
});

// Search memories
const relevant = await adapter.searchMemories({
  embedding: vector,
  roomId,
  match_threshold: 0.8,
  count: 10,
});

// Delete memories
await adapter.deleteMemory(memoryId);
await adapter.deleteAllMemories(roomId, 'messages');
```

### Room & Participant Management

```typescript
// Rooms
await adapter.createRoom(room);
const room = await adapter.getRoom(roomId);
await adapter.updateRoom(room);
await adapter.deleteRoom(roomId);

// Participants
await adapter.addParticipant(entityId, roomId);
await adapter.removeParticipant(entityId, roomId);
const participants = await adapter.getParticipantsForRoom(roomId);
const state = await adapter.getParticipantUserState(roomId, entityId);
await adapter.setParticipantUserState(roomId, entityId, 'FOLLOWED');
```

### Relationships

```typescript
// Create relationship
await adapter.createRelationship({
  sourceEntityId,
  targetEntityId,
  tags: ['friend', 'collaborator'],
  metadata: { trust: 0.8 },
});

// Get relationships
const relationships = await adapter.getRelationships({
  entityId,
  tags: ['friend'],
});

// Update relationship
await adapter.updateRelationship(relationship);
```

### Caching

```typescript
// Set cache
await adapter.setCache('key', { data: 'value' });

// Get cache
const cached = await adapter.getCache<MyType>('key');

// Delete cache
await adapter.deleteCache('key');
```

### World & Task Management

```typescript
// Worlds
await adapter.createWorld(world);
const world = await adapter.getWorld(worldId);
await adapter.updateWorld(world);
await adapter.removeWorld(worldId);

// Tasks
await adapter.createTask(task);
const tasks = await adapter.getTasks({ roomId, tags });
await adapter.updateTask(taskId, updates);
await adapter.deleteTask(taskId);
```

## Configuration

### Environment Variables

```bash
# PostgreSQL
POSTGRES_URL=postgresql://user:pass@localhost:5432/elizaos

# PGLite (optional)
SQLITE_DATA_DIR=./.elizadb  # Default: ./sqlite
```

### Initialization

The SQL plugin handles adapter initialization:

```typescript
// Plugin automatically selects adapter based on config
const plugin = '@elizaos/plugin-sql';

// Will use PostgreSQL if POSTGRES_URL is set
// Otherwise defaults to PGLite
```

## Retry Logic

Built-in retry with exponential backoff:

```typescript
protected async withRetry<T>(operation: () => Promise<T>): Promise<T> {
  let attempt = 0;

  while (attempt < this.maxRetries) {
    try {
      return await operation();
    } catch (error) {
      if (!this.isRetryableError(error)) {
        throw error;
      }

      const delay = Math.min(
        this.baseDelay * Math.pow(2, attempt) +
        Math.random() * this.jitterMax,
        this.maxDelay
      );

      await new Promise(resolve => setTimeout(resolve, delay));
      attempt++;
    }
  }

  throw lastError;
}
```

## Vector Search

Both adapters support semantic search:

```typescript
// Store with embedding
await runtime.createMemory(
  {
    content: { text: 'Important information' },
    embedding: await runtime.useModel(ModelType.TEXT_EMBEDDING, {
      text: 'Important information',
    }),
    roomId,
  },
  'facts'
);

// Search by similarity
const embedding = await runtime.useModel(ModelType.TEXT_EMBEDDING, {
  text: 'What did we discuss about databases?',
});

const memories = await runtime.searchMemories({
  embedding,
  roomId,
  count: 5,
  match_threshold: 0.7,
});
```

### Embedding Dimensions

```typescript
// Configure embedding dimension (auto-detected)
await adapter.ensureEmbeddingDimension(1536); // OpenAI
```

## Database Schema

### Core Tables

- **entities**: Users, agents, participants
- **components**: Modular entity data
- **memories**: Conversation history and knowledge
- **relationships**: Entity connections
- **rooms**: Conversation channels
- **participants**: Room membership
- **worlds**: Room containers
- **tasks**: Scheduled operations
- **cache**: Temporary storage
- **agents**: Agent configuration

### Entity-Component System

```typescript
// Entity with components
const entity = {
  id: entityId,
  names: ['John Doe'],
  metadata: { platform: 'discord' },
};

// Add profile component
const profile = {
  entityId,
  type: 'profile',
  data: {
    bio: 'Developer',
    location: 'SF',
  },
};
```

## Performance Tips

### PostgreSQL

- Install pgvector extension
- Index frequently queried fields
- Use connection pooling
- Consider partitioning for scale

### PGLite

- Keep database under 1GB
- Regular memory cleanup
- Limit concurrent operations

## Common Patterns

### Singleton Connection

```typescript
// Managers ensure single connection per process
class PostgresManager {
  private static instance: PostgresManager;

  static getInstance(): PostgresManager {
    if (!this.instance) {
      this.instance = new PostgresManager();
    }
    return this.instance;
  }
}
```

### Memory with Metadata

```typescript
await runtime.createMemory(
  {
    content: {
      text: 'User prefers dark mode',
      metadata: {
        type: 'preference',
        confidence: 0.9,
      },
    },
    entityId: userId,
    roomId,
  },
  'facts'
);
```

### Bulk Operations

```typescript
// Create multiple entities efficiently
await adapter.createEntities(entityArray);

// Get multiple rooms
const rooms = await adapter.getRoomsByIds(roomIds);
```

## Migration Support

Future releases will support:

- MongoDB
- SQLite
- Supabase
- Qdrant
- SQL.js

The adapter interface is designed for extensibility.
