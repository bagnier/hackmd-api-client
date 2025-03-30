# Add ETag Support to HackMD API Client

This repository contains a set of packages for interacting with the [HackMD API](https://hackmd.io/), forked from [upstream repository](https://github.com/hackmdio/api-client).

## Summary
This repo implements ETag support for HackMD API client operations, enabling conditional requests and response caching.

## Technical Details
  - Added ETag support to GET, POST, and PATCH operations for user notes
  - ETags are now accessible in responses for `getNote`, `createNote`, `updateNote`, and `updateNoteContent`
  - Implemented proper handling of 304 Not Modified responses for conditional GET requests

## Code Examples

### 1. Creating a note
```typescript
// Create a note and receive ETag in response
const newNote = await client.createNote({
title: 'New Note',
content: 'Initial content'
})
console.log(newNote.etag) // 'W/"abc123"'
```

### 2. Reading a note
```typescript
// Get the note and store its ETag
const note = await client.getNote(newNote.id)
console.log(note.etag) // 'W/"abc123"'
```

### 3. Updating a note
```typescript
// Update the note and receive a new ETag
const updatedNote = await client.updateNoteContent(note.id, 'Updated content')
console.log(updatedNote.etag) // 'W/"xyz789"' (different from original)
```

### 4. Conditional read with original ETag (will get new content)
```typescript
// Try reading with the original ETag
const result1 = await client.getNote(note.id, {
etag: note.etag // original ETag that's now outdated
})

// Content has changed, so we get the new version
console.log(result1.status) // undefined (normal 200 response)
console.log(result1.content) // 'Updated content'
console.log(result1.etag) // 'W/"xyz789"'
```

### 5. Conditional read with updated ETag (will get 304)
```typescript
// Try reading with the latest ETag
const result2 = await client.getNote(note.id, {
etag: updatedNote.etag // current ETag
})

// Content hasn't changed, so we get 304 Not Modified
console.log(result2.status) // 304
console.log(result2.content) // undefined
console.log(result2.etag) // 'W/"xyz789"' (same as before)
```

## Limitations
  - The HackMD API server does not currently support If-Match headers in PATCH requests, limiting conditional updates
  - Current implementation focuses on user notes operations - the most commonly used  endpoints.

## Implementation Approach
  - Made ETag usage optional to preserve backward compatibility
  - Included ETags automatically in responses when available
  - Required explicit opt-in for conditional requests via the options parameter
  - Focused implementation on user notes operations only (not team notes)
  - Comprehensive test suite verifies all ETag functionality scenarios

## Security
  - Updated dependencies to fix vulnerabilities