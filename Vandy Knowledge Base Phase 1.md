#kb #vandykb 
# **Vandy Knowledge Base Phase 1: Summary of Features & APIs 🚀**

This document summarizes the proposed architecture and key functionalities for your Deno-based Markdown knowledge base, designed with a strong emphasis on **modularity**, **performance**, and **security**.

## **Core Concepts 🧠**

The knowledge base treats each **Markdown file as a record**, leveraging **frontmatter** for structured metadata. It supports **tags** and **links** for organization, with **backlinks** automatically derived to build a rich knowledge graph.

## **Backend Architecture Overview 🛠️**

The system employs a multi-layered approach:

1. **File System Layer:** Direct interaction with Markdown files on disk.
2. **Redis Indexing Layer:** A high-speed Redis database for storing parsed metadata (frontmatter, tags, links, backlinks) and ID mappings, providing fast queries and persistence. This runs in a separate LXC for resource isolation.
3. **Core API Layer:** A set of atomic JavaScript functions responsible for specific tasks (file ops, parsing, Redis interactions).
4. **Macro Processing Layer:** A two-stage system to execute dynamic JavaScript within Markdown files in a sandboxed Web Worker environment.
5. **Command Dispatcher:** A trusted module on the main thread for executing modifications requested by macros.
6. **URL/Linking System:** Stable, ID-based URLs for notes, with `[[link]]` syntax transformed into web-accessible `<a>` tags.
7. **Hierarchy & Security:** An outline system based on folder structure, using `_index.md` files for inherited permissions via ACLs.

## **Key Functions & APIs**

Here's a breakdown of the proposed JavaScript functions and their roles:

### **1. File System Operations (e.g., in** `file_system_operations.js`**)**

These functions handle direct interaction with Markdown files on the disk.

* `readFileContent(filePath)`: Reads content from a file.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<string>`
* `writeFileContent(filePath, content)`: Writes content to a file.
  * **Parameters:** `filePath` (string), `content` (string)
  * **Returns:** `Promise<void>`
* `deleteFile(filePath)`: Deletes a file.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<void>`
* `listMarkdownFiles(directoryPath)`: Recursively lists all Markdown files.
  * **Parameters:** `directoryPath` (string)
  * **Returns:** `Promise<string[]>`
* `createMarkdownFile(filePath, content)`: Creates a new Markdown file.
  * **Parameters:** `filePath` (string), `content` (string)
  * **Returns:** `Promise<void>`
* `updateMarkdownFileContent(filePath, newContent)`: Overwrites file content.
  * **Parameters:** `filePath` (string), `newContent` (string)
  * **Returns:** `Promise<void>`
* `updateMarkdownFileFrontmatter(filePath, newFrontmatterProperties)`: Updates specific frontmatter properties in a file.
  * **Parameters:** `filePath` (string), `newFrontmatterProperties` (object)
  * **Returns:** `Promise<void>`

### **2. Markdown Parsing (e.g., in** `markdown_parsers.js`**)**

These functions extract structured data from Markdown content.

* `parseFrontmatter(markdownContent)`: Extracts YAML frontmatter and remaining content.
  * **Parameters:** `markdownContent` (string)
  * **Returns:** `{ frontmatter: Object; content: string }`
* `extractTags(markdownContent, frontmatter)`: Extracts `#tag` and frontmatter `tags`.
  * **Parameters:** `markdownContent` (string), `frontmatter` (object, optional)
  * **Returns:** `string[]`
* `extractInternalLinks(markdownContent)`: Finds `[[WikiLink]]` style links.
  * **Parameters:** `markdownContent` (string)
  * **Returns:** `string[]`

### **3. Data Representation & Indexing (e.g., in** `record_manager.js`**)**

These functions build and manage the structured `MarkdownRecord` objects and interact with Redis for indexing.

* `MarkdownRecord` **Object Structure:** Represents a parsed Markdown file.
  ```
  // A JavaScript object representing a MarkdownRecord would typically look like this:
  /*
  {
      filePath: string,
      fileName: string,
      frontmatter: Object, // Includes the 'meta' object
      content: string, // Markdown content without frontmatter
      tags: Array<string>,
      links: Array<string>, // Internal links extracted
      lastModified: Date
  }
  */


  ```
* `createMarkdownRecord(filePath, rawContent)`: Parses raw content into a `MarkdownRecord`.
  * **Parameters:** `filePath` (string), `rawContent` (string)
  * **Returns:** `Promise<Object>` (a MarkdownRecord object)
* `buildLinkGraph(records)`: Creates a forward link graph (in-memory).
  * **Parameters:** `records` (Array\<Object>, array of MarkdownRecord objects)
  * **Returns:** `Map<string, string[]>` (sourceFilePath -> \[targetFilePaths])
* `buildBacklinkIndex(linkGraph)`: Derives a backlink index (in-memory).
  * **Parameters:** `linkGraph` (Map\<string, string\[]>)
  * **Returns:** `Map<string, string[]>` (targetFilePath -> \[sourceFilePaths])

### **4. Redis Integration & Indexing (e.g., in** `redis_index_store.js`**)**

These handle the connection and specific Redis operations for managing indexes.

* `connectRedis()`: Establishes connection to Redis.
  * **Returns:** `Promise<void>`
* `storeMarkdownRecordInRedis(record)`: Stores/updates a full record in Redis (as Hash or JSON string).
  * **Parameters:** `record` (Object, a MarkdownRecord object)
  * **Returns:** `Promise<void>`
* `getMarkdownRecordFromRedis(filePath)`: Retrieves a record from Redis.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<Object|null>` (a MarkdownRecord object or null)
* `updateRedisLinkGraph(sourceFilePath, targetFilePaths)`: Updates forward and backlink sets in Redis.
  * **Parameters:** `sourceFilePath` (string), `targetFilePaths` (string\[])
  * **Returns:** `Promise<void>`
* `updateRedisTagIndex(filePath, tags, oldTags)`: Updates tag sets in Redis.
  * **Parameters:** `filePath` (string), `tags` (string\[]), `oldTags` (string\[], optional)
  * **Returns:** `Promise<void>`
* `addRecordToIndex(markdownRecord)`: High-level function to add/update record and its index entries in Redis.
  * **Parameters:** `markdownRecord` (Object, a MarkdownRecord object)
  * **Returns:** `Promise<void>`
* `removeRecordFromIndex(filePath)`: Removes record and its index entries from Redis.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<void>`

### **5. Query / Read APIs (e.g., in** `query_api.js`**)**

These functions provide efficient ways to query the knowledge base, primarily using the Redis index.

* `getMarkdownRecordByPath(filePath)`: Retrieves record by file path.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<Object|null>` (a MarkdownRecord object or null)
* `getFilePathsByTag(tag)`: Gets file paths for a given tag.
  * **Parameters:** `tag` (string)
  * **Returns:** `Promise<string[]>`
* `getFilePathsByFrontmatterValue(key, value)`: Gets file paths by frontmatter property.
  * **Parameters:** `key` (string), `value` (any)
  * **Returns:** `Promise<string[]>`
* `getBacklinksForFile(filePath)`: Gets files that link to a target file.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<string[]>`
* `getLinksFromFile(filePath)`: Gets files linked by a source file.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<string[]>`

### **6. Macro Processing (e.g., in** `macro_processor.js`**)**

A two-stage process for dynamic content rendering with sandboxing.

* **Macro Syntax:**
  * **Query Macro:** `{{query: {tag: "deno", status: "published"} }}` (processed on main thread)
  * **JavaScript Macro:** `{{js: /* your JS code */ }}` (executed in Web Worker sandbox)
* `parseSimpleQuery(queryString)`: Parses simple object-literal-like query strings.
  * **Parameters:** `queryString` (string)
  * **Returns:** `Object` (e.g., `{ tag: string, status: string }`)
* `executeStoreQuery(parsedQuery, store)`: Executes a parsed query against the main store.
  * **Parameters:** `parsedQuery` (object), `store` (object)
  * **Returns:** `Array<Object>` (array of MarkdownRecord objects)
* `processQueryMacros(markdownContent, store)`: Transforms query macros into JS macros with embedded results. (Stage 1)
  * **Parameters:** `markdownContent` (string), `store` (object)
  * **Returns:** `string` (Markdown with transformed macros)
* `executeMacroCodeInSandbox(jsCode, context)`: Executes JS code within a Deno Web Worker (the sandbox). Returns HTML output and any commands.
  * **Parameters:** `jsCode` (string), `context` (object)
  * **Returns:** `Promise<Object>` (e.g., `{ html: string, commands: Array<Object> }`)
* `processJavascriptMacros(markdownContent, store, currentFilePath)`: Finds and executes JS macros (including transformed query macros) in the sandbox, collecting commands. (Stage 2)
  * **Parameters:** `markdownContent` (string), `store` (object), `currentFilePath` (string, optional)
  * **Returns:** `Promise<Object>` (e.g., `{ processedMarkdown: string, commands: Array<Object> }`)

### **7. Macro Command Dispatching (e.g., in** `command_dispatcher.js`**)**

This trusted module on the main thread safely executes changes requested by macros.

* **Command Structure:** `Object` (e.g., `{ type: string; payload: Object; }`)
* `commandDispatcher(commands, store, updateStoreCallback)`: Validates and executes collected commands.
  * **Parameters:** `commands` (Array\<Object>), `store` (object), `updateStoreCallback` (Function)
  * **Returns:** `Promise<void>`
  * **Example Command Types:** `'updateRecordFrontmatter'`, `'addTagToRecord'`, `'createNewRecord'`, `'deleteRecord'`.

### **8. URL Management & Linking (e.g., in** `url_resolver.js`**)**

* **Stable IDs:** Each note gets a unique **UUID** stored in `meta.id` in its frontmatter, which is immutable.
* **URL Structure:** `/note/{id}` is the canonical URL.
* `generateUUIDForNewNote()`: Generates a new UUID.
  * **Returns:** `string`
* `resolveLinkTargetToId(linkTarget)`: Resolves `[[link]]` names to UUIDs via Redis.
  * **Parameters:** `linkTarget` (string)
  * **Returns:** `Promise<string|null>`
* `getFilePathById(id)`: Gets file path from UUID via Redis.
  * **Parameters:** `id` (string)
  * **Returns:** `Promise<string|null>`
* `getIdByFilePath(filePath)`: Gets UUID from file path via Redis.
  * **Parameters:** `filePath` (string)
  * **Returns:** `Promise<string|null>`
* `upsertNoteIdMapping(filePath, id)`: Stores/updates bidirectional mapping in Redis.
  * **Parameters:** `filePath` (string), `id` (string)
  * **Returns:** `Promise<void>`
* **Link Transformation in Rendering:** During Markdown-to-HTML conversion, `[[link]]` syntax is replaced with `<a href="/note/{resolved-id}">Link Text</a>` using these APIs.

### **9. Hierarchy & ACLs (e.g., in** `auth_manager.js`**)**

* **Hierarchy Representation:** Defined by the file system's directory structure.
* **Folder Metadata:** Stored in `_index.md` frontmatter within each directory, particularly for `meta.acls`.
* **Frontmatter** `meta` **Object Structure:**
  ```
  meta:
    id: <UUID>             # Immutable, for stable URLs
    created: <ISOString>   # Automatically set on creation
    modified: <ISOString>  # Automatically updated on every save
    acls:
      roles: [string]      # Roles with access (e.g., 'admin', 'editor', 'public')
      users: [string]      # Specific user IDs with access
      publicRead: boolean  # Explicitly allow unauthenticated read access
    version: <string>      # Optional: semantic version
    sourceUrl: <string>    # Optional: original source URL


  ```
* **Permission Inheritance:** ACLs merge from parent `_index.md` files down to the note, with more specific (child) rules overriding general (parent) rules.
* `getEffectiveAcls(filePath, userId, userRoles)`: Calculates the combined permissions for a given note and user.
  * **Parameters:** `filePath` (string), `userId` (string or null), `userRoles` (string\[])
  * **Returns:** `Object` (e.g., `{ canRead: boolean; canWrite: boolean; }`)
* **Authorization Middleware:** In your web server, calls `getEffectiveAcls` to grant or deny access before serving note content or allowing modifications.

This comprehensive set of APIs forms the backbone of your robust Deno-powered knowledge base, enabling powerful features like dynamic content, secure modifications, and fine-grained access control.
