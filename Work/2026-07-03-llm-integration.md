# LLM Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Wire Azure OpenAI into the existing Node.js helpdesk system to handle three scenarios: new-ticket diagnostic summaries with assignee suggestions, stale-ticket reminder messages for techs, and interpretation of inbound Teams replies to take Autotask actions.

**Architecture:** A thin `src/services/llmService.ts` wraps the already-installed `openai` SDK for Azure. Two new external-data services (`itGlueService.ts`, `dattoRmmService.ts`) supply enrichment context. Three `src/llm/` prompt modules hold system prompts and I/O parsers — one per scenario. The LLM is called from `evaluateNewTickets.ts` (Scenario 1), a new `staleTicketNotification` job (Scenario 2), and the Teams `message` event handler in `index.ts` (Scenario 3).

**Tech Stack:** TypeScript, Node.js, `openai` ^6.39.0 (Azure), `pg` (PostgreSQL), `@microsoft/teams.apps`, `pg-boss`, `vitest` ^2.0.0 (new — no tests exist today)

## Global Constraints

- Azure OpenAI deployment names come from env vars; never hardcode model names
- All LLM calls use `response_format: { type: 'json_object' }` — every scenario returns JSON
- All external service calls (IT Glue, Datto RMM) must fail gracefully: return empty/null rather than throw, so the ticket flow is not blocked if enrichment is unavailable
- Follow the existing `fetch()` pattern for all HTTP calls — no axios or other HTTP libs
- Follow the existing error pattern: `console.error(...)` and return/continue on failure
- No DB schema changes required — all needed data already exists in `tickets`, `ticket_statuses`, `ticket_priorities`, and `conversations` tables
- Scenario 1 uses `gpt-4o` (complex reasoning over KB + device data); Scenario 2 uses `gpt-4o-mini` (simple summarization, runs frequently); Scenario 3 uses `gpt-4o` (intent classification must be accurate to avoid wrong Autotask writes)

---

## New Environment Variables

Add these to `.env` / Docker Compose:

```
AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
AZURE_OPENAI_API_KEY=<key>
AZURE_OPENAI_GPT4O_DEPLOYMENT=gpt-4o
AZURE_OPENAI_GPT4O_MINI_DEPLOYMENT=gpt-4o-mini
ITGLUE_API_KEY=<key>
ITGLUE_BASE_URL=https://api.itglue.com
DATTO_RMM_API_KEY=<key>
DATTO_RMM_SECRET_KEY=<secret>
DATTO_RMM_PLATFORM_URL=https://<instance>.centrastage.net
STALE_TICKET_DAYS=3
STALE_TICKET_MAX_PER_RUN=5
```

---

## File Map

| Action | Path | Responsibility |
|--------|------|---------------|
| **Create** | `src/services/llmService.ts` | Azure OpenAI client, `callLlmJson<T>()` |
| **Create** | `src/services/itGlueService.ts` | IT Glue Documents search |
| **Create** | `src/services/dattoRmmService.ts` | Datto RMM OAuth token + device lookup |
| **Create** | `src/llm/newTicketPrompt.ts` | Scenario 1 prompt + parser |
| **Create** | `src/llm/staleTicketPrompt.ts` | Scenario 2 prompt + parser |
| **Create** | `src/llm/techReplyPrompt.ts` | Scenario 3 prompt + parser |
| **Create** | `src/jobs/staleTicketNotification.ts` | Scenario 2 job |
| **Create** | `vitest.config.ts` | Test runner config |
| **Create** | `tests/services/llmService.test.ts` | Unit tests for LLM service |
| **Create** | `tests/services/itGlueService.test.ts` | Unit tests for IT Glue service |
| **Create** | `tests/services/dattoRmmService.test.ts` | Unit tests for Datto RMM service |
| **Create** | `tests/llm/newTicketPrompt.test.ts` | Unit tests for Scenario 1 prompt |
| **Create** | `tests/llm/staleTicketPrompt.test.ts` | Unit tests for Scenario 2 prompt |
| **Create** | `tests/llm/techReplyPrompt.test.ts` | Unit tests for Scenario 3 prompt |
| **Modify** | `src/types.ts` | Add `AutotaskResource`, `Resource`, 4 LLM I/O types |
| **Modify** | `src/autotaskApiCalls.ts` | Add `getResources`, `addTicketNote`, `updateTicketStatus`, `updateTicketAssignee` |
| **Implement** | `src/evaluateNewTickets.ts` | Currently 0 bytes — implements Scenario 1 orchestration |
| **Modify** | `src/jobs/newTicketNotification.ts` | Call `evaluateNewTicket`, add AI section to card |
| **Modify** | `src/jobs/jobScheduler.ts` | Register `stale-ticket-notification` job |
| **Modify** | `src/index.ts` | Update Teams message handler for Scenario 3 |
| **Modify** | `package.json` | Add vitest, test script |

---

## Task 1: Testing Infrastructure + LLM Service

**Files:**
- Create: `vitest.config.ts`
- Modify: `package.json` (add vitest, test script)
- Create: `src/services/llmService.ts`
- Create: `tests/services/llmService.test.ts`

**Interfaces:**
- Produces: `callLlmJson<T>(deployment, systemPrompt, userContent): Promise<T>`, `GPT4O: string`, `GPT4O_MINI: string`

---

- [x] **Step 1.1: Add vitest to package.json**

Edit `package.json`. Add `"test": "vitest run"` to scripts and `"vitest": "^2.0.0"` to devDependencies:

```json
{
  "name": "ai-helpdesk-app",
  "version": "1.0.0",
  "description": "",
  "keywords": [],
  "license": "ISC",
  "author": "",
  "type": "commonjs",
  "main": "index.js",
  "scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon src/index.ts",
    "build": "tsc",
    "test": "vitest run"
  },
  "dependencies": {
    "@microsoft/teams.apps": "^2.0.12",
    "botbuilder": "^4.23.3",
    "botframework-connector": "^4.23.3",
    "dotenv": "^17.4.2",
    "express": "^5.2.1",
    "node-cron": "^4.2.1",
    "openai": "^6.39.0",
    "pg": "^8.21.0",
    "pg-boss": "^12.18.2"
  },
  "devDependencies": {
    "@types/express": "^5.0.6",
    "@types/node": "^25.9.1",
    "@types/node-cron": "^3.0.11",
    "@types/pg": "^8.20.0",
    "nodemon": "^3.1.14",
    "ts-node": "^10.9.2",
    "typescript": "^6.0.3",
    "vitest": "^2.0.0"
  }
}
```

- [x] **Step 1.2: Create vitest.config.ts**

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'node',
    globals: true,
  },
});
```

- [x] **Step 1.3: Install vitest**

Run: `npm install`

- [x] **Step 1.4: Write the failing test for llmService**

Create `tests/services/llmService.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

// Mock the openai module before importing llmService
vi.mock('openai', () => {
  const mockCreate = vi.fn();
  return {
    AzureOpenAI: vi.fn().mockImplementation(() => ({
      chat: { completions: { create: mockCreate } },
    })),
    _mockCreate: mockCreate,
  };
});

import { callLlmJson, GPT4O, GPT4O_MINI } from '../../src/services/llmService';

describe('llmService', () => {
  beforeEach(() => {
    process.env.AZURE_OPENAI_ENDPOINT = 'https://test.openai.azure.com/';
    process.env.AZURE_OPENAI_API_KEY = 'test-key';
    process.env.AZURE_OPENAI_GPT4O_DEPLOYMENT = 'gpt-4o';
    process.env.AZURE_OPENAI_GPT4O_MINI_DEPLOYMENT = 'gpt-4o-mini';
  });

  it('returns parsed JSON from LLM response', async () => {
    const { AzureOpenAI } = await import('openai') as any;
    const mockInstance = new AzureOpenAI();
    mockInstance.chat.completions.create.mockResolvedValueOnce({
      choices: [{ message: { content: JSON.stringify({ foo: 'bar' }) } }],
    });

    const result = await callLlmJson<{ foo: string }>('gpt-4o', 'system', 'user');
    expect(result).toEqual({ foo: 'bar' });
  });

  it('throws when LLM returns empty content', async () => {
    const { AzureOpenAI } = await import('openai') as any;
    const mockInstance = new AzureOpenAI();
    mockInstance.chat.completions.create.mockResolvedValueOnce({
      choices: [{ message: { content: null } }],
    });

    await expect(callLlmJson('gpt-4o', 'system', 'user')).rejects.toThrow('LLM returned empty response');
  });

  it('GPT4O reads from env var', () => {
    expect(GPT4O).toBe('gpt-4o');
  });

  it('GPT4O_MINI reads from env var', () => {
    expect(GPT4O_MINI).toBe('gpt-4o-mini');
  });
});
```

- [x] **Step 1.5: Run test to confirm it fails**

Run: `npm test`
Expected: FAIL — `Cannot find module '../../src/services/llmService'`

- [x] **Step 1.6: Create src/services/llmService.ts**

```typescript
import { AzureOpenAI } from 'openai';

export const GPT4O = process.env.AZURE_OPENAI_GPT4O_DEPLOYMENT ?? 'gpt-4o';
export const GPT4O_MINI = process.env.AZURE_OPENAI_GPT4O_MINI_DEPLOYMENT ?? 'gpt-4o-mini';

function getClient(): AzureOpenAI {
  return new AzureOpenAI({
    endpoint: process.env.AZURE_OPENAI_ENDPOINT!,
    apiKey: process.env.AZURE_OPENAI_API_KEY!,
    apiVersion: '2024-08-01-preview',
  });
}

export async function callLlmJson<T>(
  deployment: string,
  systemPrompt: string,
  userContent: string,
): Promise<T> {
  const client = getClient();
  const response = await client.chat.completions.create({
    model: deployment,
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userContent },
    ],
    temperature: 0.2,
    response_format: { type: 'json_object' },
  });
  const raw = response.choices[0]?.message?.content;
  if (!raw) throw new Error('LLM returned empty response');
  return JSON.parse(raw) as T;
}
```

- [x] **Step 1.7: Run test to confirm it passes**

Run: `npm test`
Expected: PASS — all 4 tests green

- [x] **Step 1.8: Commit**

```bash
git add package.json vitest.config.ts src/services/llmService.ts tests/services/llmService.test.ts
git commit -m "feat: add vitest and Azure OpenAI LLM service"
```

---

## Task 2: IT Glue Service

**Files:**
- Create: `src/services/itGlueService.ts`
- Create: `tests/services/itGlueService.test.ts`

**Interfaces:**
- Produces: `searchItGlueArticles(keywords: string, maxResults?: number): Promise<ItGlueArticle[]>`
- Produces: `interface ItGlueArticle { title: string; content: string; }`
- Consumes: env `ITGLUE_API_KEY`, `ITGLUE_BASE_URL`

---

- [ ] **Step 2.1: Write the failing test**

Create `tests/services/itGlueService.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';

describe('itGlueService', () => {
  beforeEach(() => {
    process.env.ITGLUE_API_KEY = 'test-key';
    process.env.ITGLUE_BASE_URL = 'https://api.itglue.com';
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('returns mapped articles from IT Glue response', async () => {
    const mockResponse = {
      data: [
        { attributes: { name: 'Outlook Setup Guide', content: 'Install Outlook and configure SMTP...' } },
        { attributes: { name: 'VPN Troubleshooting', content: 'Step 1: Check network adapter...' } },
      ],
    };
    vi.stubGlobal('fetch', vi.fn().mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve(mockResponse),
    }));

    const { searchItGlueArticles } = await import('../../src/services/itGlueService');
    const articles = await searchItGlueArticles('Outlook');
    expect(articles).toHaveLength(2);
    expect(articles[0].title).toBe('Outlook Setup Guide');
    expect(articles[0].content).toBe('Install Outlook and configure SMTP...');
  });

  it('truncates article content to 2000 characters', async () => {
    const longContent = 'x'.repeat(5000);
    vi.stubGlobal('fetch', vi.fn().mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ data: [{ attributes: { name: 'Long Article', content: longContent } }] }),
    }));

    const { searchItGlueArticles } = await import('../../src/services/itGlueService');
    const articles = await searchItGlueArticles('test');
    expect(articles[0].content).toHaveLength(2000);
  });

  it('returns empty array when API key is not set', async () => {
    delete process.env.ITGLUE_API_KEY;
    const { searchItGlueArticles } = await import('../../src/services/itGlueService');
    const articles = await searchItGlueArticles('test');
    expect(articles).toEqual([]);
  });

  it('returns empty array on non-ok response', async () => {
    vi.stubGlobal('fetch', vi.fn().mockResolvedValueOnce({ ok: false, status: 401, statusText: 'Unauthorized' }));
    const { searchItGlueArticles } = await import('../../src/services/itGlueService');
    const articles = await searchItGlueArticles('test');
    expect(articles).toEqual([]);
  });
});
```

- [ ] **Step 2.2: Run test to confirm it fails**

Run: `npm test`
Expected: FAIL — `Cannot find module '../../src/services/itGlueService'`

- [ ] **Step 2.3: Create src/services/itGlueService.ts**

```typescript
export interface ItGlueArticle {
  title: string;
  content: string;
}

function getHeaders(): Record<string, string> {
  return {
    'x-api-key': process.env.ITGLUE_API_KEY!,
    'Content-Type': 'application/vnd.api+json',
  };
}

export async function searchItGlueArticles(
  keywords: string,
  maxResults = 3,
): Promise<ItGlueArticle[]> {
  if (!process.env.ITGLUE_API_KEY) return [];

  const baseUrl = process.env.ITGLUE_BASE_URL ?? 'https://api.itglue.com';
  const params = new URLSearchParams({
    'filter[name]': keywords,
    'page[size]': String(maxResults),
  });

  try {
    const response = await fetch(`${baseUrl}/documents?${params}`, { headers: getHeaders() });

    if (!response.ok) {
      console.error(`IT Glue API error: ${response.status} ${response.statusText}`);
      return [];
    }

    const data = await response.json() as {
      data: Array<{ attributes: { name: string; content: string } }>;
    };

    return (data.data ?? []).map(d => ({
      title: d.attributes.name,
      content: (d.attributes.content ?? '').slice(0, 2000),
    }));
  } catch (error) {
    console.error('IT Glue service error:', error);
    return [];
  }
}
```

- [ ] **Step 2.4: Run test to confirm it passes**

Run: `npm test`
Expected: PASS

- [ ] **Step 2.5: Commit**

```bash
git add src/services/itGlueService.ts tests/services/itGlueService.test.ts
git commit -m "feat: add IT Glue KB article search service"
```

---

## Task 3: Datto RMM Service

**Files:**
- Create: `src/services/dattoRmmService.ts`
- Create: `tests/services/dattoRmmService.test.ts`

**Interfaces:**
- Produces: `getDeviceByHostname(hostname: string): Promise<DattoDevice | null>`
- Produces: `interface DattoDevice { hostname: string; lastLoggedInUser: string; operatingSystem: string; lastSeen: string; online: boolean; }`
- Consumes: env `DATTO_RMM_API_KEY`, `DATTO_RMM_SECRET_KEY`, `DATTO_RMM_PLATFORM_URL`

---

- [ ] **Step 3.1: Write the failing test**

Create `tests/services/dattoRmmService.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';

describe('dattoRmmService', () => {
  beforeEach(() => {
    process.env.DATTO_RMM_API_KEY = 'test-api-key';
    process.env.DATTO_RMM_SECRET_KEY = 'test-secret';
    process.env.DATTO_RMM_PLATFORM_URL = 'https://test.centrastage.net';
    vi.resetModules();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('fetches token then returns device by hostname', async () => {
    const mockDevice = {
      hostname: 'DESKTOP-ABC123',
      lastLoggedInUser: 'jsmith',
      operatingSystem: 'Windows 11',
      lastSeen: '2026-07-03T10:00:00Z',
      online: true,
    };

    const fetchMock = vi.fn()
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ access_token: 'tok123', expires_in: 3600 }),
      })
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ devices: [mockDevice] }),
      });
    vi.stubGlobal('fetch', fetchMock);

    const { getDeviceByHostname } = await import('../../src/services/dattoRmmService');
    const device = await getDeviceByHostname('DESKTOP-ABC123');
    expect(device).toEqual(mockDevice);
    expect(fetchMock).toHaveBeenCalledTimes(2);
    expect(fetchMock.mock.calls[0][0]).toContain('/auth/oauth/token');
    expect(fetchMock.mock.calls[1][0]).toContain('DESKTOP-ABC123');
  });

  it('returns null when no devices match', async () => {
    vi.stubGlobal('fetch', vi.fn()
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ access_token: 'tok123', expires_in: 3600 }),
      })
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ devices: [] }),
      }),
    );

    const { getDeviceByHostname } = await import('../../src/services/dattoRmmService');
    const device = await getDeviceByHostname('UNKNOWN-HOST');
    expect(device).toBeNull();
  });

  it('returns null when API key is not set', async () => {
    delete process.env.DATTO_RMM_API_KEY;
    const { getDeviceByHostname } = await import('../../src/services/dattoRmmService');
    const device = await getDeviceByHostname('DESKTOP-ABC123');
    expect(device).toBeNull();
  });

  it('returns null and logs error when API call fails', async () => {
    const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});
    vi.stubGlobal('fetch', vi.fn()
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ access_token: 'tok123', expires_in: 3600 }),
      })
      .mockResolvedValueOnce({ ok: false, status: 500 }),
    );

    const { getDeviceByHostname } = await import('../../src/services/dattoRmmService');
    const device = await getDeviceByHostname('DESKTOP-ABC123');
    expect(device).toBeNull();
    consoleSpy.mockRestore();
  });
});
```

- [ ] **Step 3.2: Run test to confirm it fails**

Run: `npm test`
Expected: FAIL — `Cannot find module '../../src/services/dattoRmmService'`

- [ ] **Step 3.3: Create src/services/dattoRmmService.ts**

```typescript
export interface DattoDevice {
  hostname: string;
  lastLoggedInUser: string;
  operatingSystem: string;
  lastSeen: string;
  online: boolean;
}

interface CachedToken {
  accessToken: string;
  expiresAt: number;
}

let cachedToken: CachedToken | null = null;

async function getAccessToken(): Promise<string> {
  if (cachedToken && Date.now() < cachedToken.expiresAt - 60_000) {
    return cachedToken.accessToken;
  }

  const platformUrl = process.env.DATTO_RMM_PLATFORM_URL!;
  const apiKey = encodeURIComponent(process.env.DATTO_RMM_API_KEY!);
  const secretKey = encodeURIComponent(process.env.DATTO_RMM_SECRET_KEY!);

  const response = await fetch(
    `${platformUrl}/auth/oauth/token?grant_type=password&username=${apiKey}&password=${secretKey}`,
    { method: 'POST' },
  );

  if (!response.ok) {
    throw new Error(`Datto RMM auth failed: ${response.status}`);
  }

  const data = await response.json() as { access_token: string; expires_in: number };
  cachedToken = {
    accessToken: data.access_token,
    expiresAt: Date.now() + data.expires_in * 1000,
  };
  return cachedToken.accessToken;
}

export async function getDeviceByHostname(hostname: string): Promise<DattoDevice | null> {
  if (!process.env.DATTO_RMM_API_KEY) return null;

  try {
    const token = await getAccessToken();
    const platformUrl = process.env.DATTO_RMM_PLATFORM_URL!;
    const response = await fetch(
      `${platformUrl}/api/v2/account/devices?hostname=${encodeURIComponent(hostname)}`,
      { headers: { Authorization: `Bearer ${token}` } },
    );

    if (!response.ok) {
      console.error(`Datto RMM API error: ${response.status}`);
      return null;
    }

    const data = await response.json() as { devices: DattoDevice[] };
    return data.devices[0] ?? null;
  } catch (error) {
    console.error('Datto RMM service error:', error);
    return null;
  }
}
```

- [ ] **Step 3.4: Run test to confirm it passes**

Run: `npm test`
Expected: PASS

- [ ] **Step 3.5: Commit**

```bash
git add src/services/dattoRmmService.ts tests/services/dattoRmmService.test.ts
git commit -m "feat: add Datto RMM device lookup service"
```

---

## Task 4: Autotask API Extensions + New Types

**Files:**
- Modify: `src/types.ts` (append new types)
- Modify: `src/autotaskApiCalls.ts` (append 4 new functions)
- Create: `tests/autotaskApiCalls.test.ts`

**Interfaces:**
- Produces (types): `AutotaskResource`, `Resource`, `LlmNewTicketInput`, `LlmNewTicketOutput`, `LlmStaleTicketInput`, `LlmStaleTicketOutput`, `LlmTechReplyInput`, `LlmTechReplyOutput`
- Produces (functions): `getResources(): Promise<Resource[]>`, `addTicketNote(ticketId, title, description): Promise<void>`, `updateTicketStatus(ticketId, statusId): Promise<void>`, `updateTicketAssignee(ticketId, resourceId): Promise<void>`

---

- [ ] **Step 4.1: Write the failing tests**

Create `tests/autotaskApiCalls.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';

describe('autotaskApiCalls — new functions', () => {
  beforeEach(() => {
    process.env.AUTOTASK_REGION = '1';
    process.env.AUTOTASK_TRACKING_ID = 'track-id';
    process.env.AUTOTASK_API_USER = 'user@test.com';
    process.env.AUTOTASK_API_SECRET = 'secret';
  });

  afterEach(() => vi.restoreAllMocks());

  it('getResources fetches active resources and maps to Resource[]', async () => {
    vi.stubGlobal('fetch', vi.fn().mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({
        items: [
          { id: 1, firstName: 'Alice', lastName: 'Smith', isActive: true },
          { id: 2, firstName: 'Bob', lastName: 'Jones', isActive: true },
        ],
      }),
    }));

    const { getResources } = await import('../src/autotaskApiCalls');
    const resources = await getResources();
    expect(resources).toHaveLength(2);
    expect(resources[0]).toEqual({ id: 1, name: 'Alice Smith', isActive: true });
    expect(resources[1]).toEqual({ id: 2, name: 'Bob Jones', isActive: true });
  });

  it('addTicketNote POSTs to TicketNotes endpoint', async () => {
    const fetchMock = vi.fn().mockResolvedValueOnce({ ok: true, json: () => Promise.resolve({}) });
    vi.stubGlobal('fetch', fetchMock);

    const { addTicketNote } = await import('../src/autotaskApiCalls');
    await addTicketNote(12345, 'Update from Tech', 'Rebooted the device.');

    expect(fetchMock).toHaveBeenCalledOnce();
    const [url, opts] = fetchMock.mock.calls[0];
    expect(url).toContain('/TicketNotes');
    expect(opts.method).toBe('POST');
    const body = JSON.parse(opts.body);
    expect(body.ticketID).toBe(12345);
    expect(body.description).toBe('Rebooted the device.');
  });

  it('updateTicketStatus PATCHes the Tickets endpoint with statusId', async () => {
    const fetchMock = vi.fn().mockResolvedValueOnce({ ok: true, json: () => Promise.resolve({}) });
    vi.stubGlobal('fetch', fetchMock);

    const { updateTicketStatus } = await import('../src/autotaskApiCalls');
    await updateTicketStatus(12345, 5);

    const [url, opts] = fetchMock.mock.calls[0];
    expect(url).toContain('/Tickets/12345');
    expect(opts.method).toBe('PATCH');
    expect(JSON.parse(opts.body)).toEqual({ id: 12345, status: 5 });
  });

  it('updateTicketAssignee PATCHes the Tickets endpoint with resourceId', async () => {
    const fetchMock = vi.fn().mockResolvedValueOnce({ ok: true, json: () => Promise.resolve({}) });
    vi.stubGlobal('fetch', fetchMock);

    const { updateTicketAssignee } = await import('../src/autotaskApiCalls');
    await updateTicketAssignee(12345, 7);

    const [url, opts] = fetchMock.mock.calls[0];
    expect(url).toContain('/Tickets/12345');
    expect(opts.method).toBe('PATCH');
    expect(JSON.parse(opts.body)).toEqual({ id: 12345, assignedResourceID: 7 });
  });

  it('addTicketNote throws on non-ok response', async () => {
    vi.stubGlobal('fetch', vi.fn().mockResolvedValueOnce({ ok: false, status: 400 }));
    const { addTicketNote } = await import('../src/autotaskApiCalls');
    await expect(addTicketNote(1, 'title', 'desc')).rejects.toThrow('Failed to add ticket note');
  });
});
```

- [ ] **Step 4.2: Run test to confirm it fails**

Run: `npm test`
Expected: FAIL — `getResources is not a function` (and similar for the other 3)

- [ ] **Step 4.3: Append new types to src/types.ts**

Add at the end of `src/types.ts`:

```typescript
export type AutotaskResource = {
  id: number;
  firstName: string;
  lastName: string;
  isActive: boolean;
};

export type Resource = {
  id: number;
  name: string;
  isActive: boolean;
};

export type LlmNewTicketInput = {
  ticket: {
    id: number;
    ticketNumber: string;
    title: string;
    description: string;
    status: string;
    priority: string;
    queue: string | null;
  };
  notes: Array<{ createdAt: string; title: string; description: string }>;
  timeEntries: Array<{ startDateTime: string; hoursWorked: number; summaryNotes: string }>;
  itGlueArticles: Array<{ title: string; content: string }>;
  deviceData: {
    hostname: string;
    lastLoggedInUser: string;
    operatingSystem: string;
    lastSeen: string;
    online: boolean;
  } | null;
  availableResources: Array<{ id: number; name: string }>;
};

export type LlmNewTicketOutput = {
  diagnosticSummary: string;
  suggestedAssignee: {
    resourceId: number;
    name: string;
    reason: string;
  } | null;
};

export type LlmStaleTicketInput = {
  ticket: {
    id: number;
    ticketNumber: string;
    title: string;
    description: string;
    status: string;
    daysSinceLastActivity: number;
  };
  notes: Array<{ createdAt: string; title: string; description: string }>;
  timeEntries: Array<{ startDateTime: string; hoursWorked: number; summaryNotes: string }>;
};

export type LlmStaleTicketOutput = {
  reminderMessage: string;
};

export type LlmTechReplyInput = {
  techMessage: string;
  ticket: {
    id: number;
    ticketNumber: string;
    title: string;
    description: string;
    currentStatus: string;
  };
  availableStatuses: Array<{ id: number; name: string }>;
  availableResources: Array<{ id: number; name: string }>;
};

export type LlmTechReplyOutput = {
  action: 'add_note' | 'change_status' | 'reassign' | 'add_note_and_change_status' | 'no_action';
  note: string | null;
  statusId: number | null;
  newAssigneeId: number | null;
  explanation: string;
};
```

- [ ] **Step 4.4: Append 4 new functions to src/autotaskApiCalls.ts**

Add at the end of `src/autotaskApiCalls.ts` (after the last export):

```typescript
export async function getResources(): Promise<import('./types').Resource[]> {
  const region = process.env.AUTOTASK_REGION;
  const trackingId = process.env.AUTOTASK_TRACKING_ID;
  const apiUser = process.env.AUTOTASK_API_USER;
  const apiSecret = process.env.AUTOTASK_API_SECRET;
  const url = `https://webservices${region}.autotask.net/atservicesrest/v1.0/Resources/query`;

  if (!trackingId || !apiUser || !apiSecret) {
    throw new Error('Missing Autotask environment variables');
  }

  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'ApiIntegrationCode': trackingId,
      'UserName': apiUser,
      'Secret': apiSecret,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ filter: [{ op: 'eq', field: 'isActive', value: true }] }),
  });

  const data = await response.json() as { items: import('./types').AutotaskResource[] };
  return (data.items ?? []).map(r => ({
    id: r.id,
    name: `${r.firstName} ${r.lastName}`.trim(),
    isActive: r.isActive,
  }));
}

export async function addTicketNote(
  ticketId: number,
  title: string,
  description: string,
): Promise<void> {
  const region = process.env.AUTOTASK_REGION;
  const trackingId = process.env.AUTOTASK_TRACKING_ID;
  const apiUser = process.env.AUTOTASK_API_USER;
  const apiSecret = process.env.AUTOTASK_API_SECRET;
  const url = `https://webservices${region}.autotask.net/atservicesrest/v1.0/TicketNotes`;

  if (!trackingId || !apiUser || !apiSecret) {
    throw new Error('Missing Autotask environment variables');
  }

  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'ApiIntegrationCode': trackingId,
      'UserName': apiUser,
      'Secret': apiSecret,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      ticketID: ticketId,
      title,
      description,
      noteType: 1,
      publish: 1,
    }),
  });

  if (!response.ok) {
    throw new Error(`Failed to add ticket note: ${response.status}`);
  }

  console.log(`Added note to ticket ${ticketId}`);
}

export async function updateTicketStatus(ticketId: number, statusId: number): Promise<void> {
  const region = process.env.AUTOTASK_REGION;
  const trackingId = process.env.AUTOTASK_TRACKING_ID;
  const apiUser = process.env.AUTOTASK_API_USER;
  const apiSecret = process.env.AUTOTASK_API_SECRET;
  const url = `https://webservices${region}.autotask.net/atservicesrest/v1.0/Tickets/${ticketId}`;

  if (!trackingId || !apiUser || !apiSecret) {
    throw new Error('Missing Autotask environment variables');
  }

  const response = await fetch(url, {
    method: 'PATCH',
    headers: {
      'ApiIntegrationCode': trackingId,
      'UserName': apiUser,
      'Secret': apiSecret,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ id: ticketId, status: statusId }),
  });

  if (!response.ok) {
    throw new Error(`Failed to update ticket status: ${response.status}`);
  }

  console.log(`Updated ticket ${ticketId} status to ${statusId}`);
}

export async function updateTicketAssignee(ticketId: number, resourceId: number): Promise<void> {
  const region = process.env.AUTOTASK_REGION;
  const trackingId = process.env.AUTOTASK_TRACKING_ID;
  const apiUser = process.env.AUTOTASK_API_USER;
  const apiSecret = process.env.AUTOTASK_API_SECRET;
  const url = `https://webservices${region}.autotask.net/atservicesrest/v1.0/Tickets/${ticketId}`;

  if (!trackingId || !apiUser || !apiSecret) {
    throw new Error('Missing Autotask environment variables');
  }

  const response = await fetch(url, {
    method: 'PATCH',
    headers: {
      'ApiIntegrationCode': trackingId,
      'UserName': apiUser,
      'Secret': apiSecret,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ id: ticketId, assignedResourceID: resourceId }),
  });

  if (!response.ok) {
    throw new Error(`Failed to update ticket assignee: ${response.status}`);
  }

  console.log(`Updated ticket ${ticketId} assignee to resource ${resourceId}`);
}
```

- [ ] **Step 4.5: Run test to confirm it passes**

Run: `npm test`
Expected: PASS

- [ ] **Step 4.6: Commit**

```bash
git add src/types.ts src/autotaskApiCalls.ts tests/autotaskApiCalls.test.ts
git commit -m "feat: add resource types, LLM I/O types, and Autotask write operations"
```

---

## Task 5: Scenario 1 — New Ticket Diagnostic Summary

**Files:**
- Create: `src/llm/newTicketPrompt.ts`
- Implement: `src/evaluateNewTickets.ts` (currently 0 bytes)
- Modify: `src/jobs/newTicketNotification.ts` (enhance card + call evaluateNewTicket)
- Create: `tests/llm/newTicketPrompt.test.ts`

**Interfaces:**
- Consumes: `callLlmJson` from `llmService`, `GPT4O` from `llmService`, `LlmNewTicketInput`, `LlmNewTicketOutput` from `types`
- Produces: `buildNewTicketLlmResult(input: LlmNewTicketInput): Promise<LlmNewTicketOutput>`
- Produces: `evaluateNewTicket(ticket: Ticket): Promise<LlmNewTicketOutput | null>`

---

- [ ] **Step 5.1: Write the failing test for newTicketPrompt**

Create `tests/llm/newTicketPrompt.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

vi.mock('../../src/services/llmService', () => ({
  callLlmJson: vi.fn(),
  GPT4O: 'gpt-4o',
}));

describe('newTicketPrompt', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('calls callLlmJson with GPT4O and returns parsed output', async () => {
    const { callLlmJson } = await import('../../src/services/llmService');
    const mockOutput = {
      diagnosticSummary: 'Device cannot connect to the VPN. Likely misconfigured split tunneling.',
      suggestedAssignee: { resourceId: 3, name: 'Alice Smith', reason: 'Network specialist.' },
    };
    (callLlmJson as any).mockResolvedValueOnce(mockOutput);

    const { buildNewTicketLlmResult } = await import('../../src/llm/newTicketPrompt');
    const input = {
      ticket: { id: 1, ticketNumber: 'T20260703.0001', title: 'VPN not connecting', description: '', status: 'New', priority: 'High', queue: 'Help Desk' },
      notes: [],
      timeEntries: [],
      itGlueArticles: [],
      deviceData: null,
      availableResources: [{ id: 3, name: 'Alice Smith' }],
    };

    const result = await buildNewTicketLlmResult(input as any);
    expect(result).toEqual(mockOutput);
    expect(callLlmJson).toHaveBeenCalledWith('gpt-4o', expect.stringContaining('MSP'), expect.stringContaining('VPN not connecting'));
  });

  it('passes availableResources in the user content', async () => {
    const { callLlmJson } = await import('../../src/services/llmService');
    (callLlmJson as any).mockResolvedValueOnce({ diagnosticSummary: 'test', suggestedAssignee: null });

    const { buildNewTicketLlmResult } = await import('../../src/llm/newTicketPrompt');
    const input = {
      ticket: { id: 1, ticketNumber: 'T20260703.0001', title: 'Test', description: '', status: 'New', priority: 'Low', queue: null },
      notes: [],
      timeEntries: [],
      itGlueArticles: [{ title: 'KB Article', content: 'Fix: restart...' }],
      deviceData: { hostname: 'DESKTOP-001', lastLoggedInUser: 'jdoe', operatingSystem: 'Windows 11', lastSeen: '2026-07-03T10:00:00Z', online: true },
      availableResources: [{ id: 5, name: 'Bob Jones' }],
    };

    await buildNewTicketLlmResult(input as any);
    const userContent = (callLlmJson as any).mock.calls[0][2];
    expect(userContent).toContain('Bob Jones');
    expect(userContent).toContain('KB Article');
    expect(userContent).toContain('DESKTOP-001');
  });
});
```

- [ ] **Step 5.2: Run test to confirm it fails**

Run: `npm test`
Expected: FAIL — `Cannot find module '../../src/llm/newTicketPrompt'`

- [ ] **Step 5.3: Create src/llm/newTicketPrompt.ts**

```typescript
import { LlmNewTicketInput, LlmNewTicketOutput } from '../types';
import { callLlmJson, GPT4O } from '../services/llmService';

const SYSTEM_PROMPT = `You are an IT support assistant for a managed service provider (MSP).
Given a newly created helpdesk ticket enriched with knowledge base articles and device telemetry, produce:
1. A concise 2–4 sentence diagnostic summary identifying the likely root cause and recommended first steps.
2. A suggested assignee from the available technicians, or null if there is insufficient context to make a recommendation.

Respond ONLY with valid JSON in this exact format:
{
  "diagnosticSummary": "...",
  "suggestedAssignee": { "resourceId": 1, "name": "Tech Name", "reason": "One sentence reason." }
}

If you cannot suggest an assignee, set "suggestedAssignee" to null.`;

export async function buildNewTicketLlmResult(input: LlmNewTicketInput): Promise<LlmNewTicketOutput> {
  return callLlmJson<LlmNewTicketOutput>(GPT4O, SYSTEM_PROMPT, JSON.stringify(input));
}
```

- [ ] **Step 5.4: Run test to confirm it passes**

Run: `npm test`
Expected: PASS

- [ ] **Step 5.5: Implement src/evaluateNewTickets.ts**

Replace the empty file with:

```typescript
import { getTicketNotes, getTimeEntries, getResources } from './autotaskApiCalls';
import { searchItGlueArticles } from './services/itGlueService';
import { getDeviceByHostname } from './services/dattoRmmService';
import { buildNewTicketLlmResult } from './llm/newTicketPrompt';
import { LlmNewTicketInput, LlmNewTicketOutput, Ticket } from './types';
import { query } from './db';

function extractPossibleHostname(title: string, description: string): string | null {
  // Autotask ticket titles often include device hostnames like "DESKTOP-ABC123" or "LAPTOP-XYZ"
  const match = (title + ' ' + description).match(/\b([A-Z][A-Z0-9-]{4,})\b/);
  return match ? match[1] : null;
}

export async function evaluateNewTicket(ticket: Ticket): Promise<LlmNewTicketOutput | null> {
  try {
    const [notes, timeEntries, resources] = await Promise.all([
      getTicketNotes(ticket.id),
      getTimeEntries(ticket.id),
      getResources(),
    ]);

    const keywords = ticket.title.split(' ').slice(0, 5).join(' ');
    const itGlueArticles = await searchItGlueArticles(keywords);

    const possibleHostname = extractPossibleHostname(ticket.title, ticket.description ?? '');
    const deviceData = possibleHostname ? await getDeviceByHostname(possibleHostname) : null;

    const [statusResult, priorityResult] = await Promise.all([
      query('SELECT name FROM ticket_statuses WHERE status_id = $1', [ticket.status]),
      query('SELECT name FROM ticket_priorities WHERE priority_id = $1', [ticket.priority]),
    ]);
    const statusName = statusResult.rows[0]?.name ?? String(ticket.status);
    const priorityName = priorityResult.rows[0]?.name ?? String(ticket.priority);

    const input: LlmNewTicketInput = {
      ticket: {
        id: ticket.id,
        ticketNumber: ticket.ticketNumber,
        title: ticket.title,
        description: ticket.description ?? '',
        status: statusName,
        priority: priorityName,
        queue: ticket.queue != null ? String(ticket.queue) : null,
      },
      notes: notes.slice(0, 10).map(n => ({
        createdAt: n.createDateTime.toISOString(),
        title: n.title,
        description: n.description,
      })),
      timeEntries: timeEntries.slice(0, 5).map(t => ({
        startDateTime: t.startDateTime.toISOString(),
        hoursWorked: t.hoursWorked,
        summaryNotes: t.summaryNotes,
      })),
      itGlueArticles,
      deviceData: deviceData ?? null,
      availableResources: resources.map(r => ({ id: r.id, name: r.name })),
    };

    return await buildNewTicketLlmResult(input);
  } catch (error) {
    console.error(`Error evaluating new ticket ${ticket.id}:`, error);
    return null;
  }
}
```

- [ ] **Step 5.6: Modify src/jobs/newTicketNotification.ts**

Add the import at the top (after existing imports):

```typescript
import { evaluateNewTicket } from '../evaluateNewTickets';
import { LlmNewTicketOutput } from '../types';
```

Change `buildTicketAlertCard` signature to accept an optional AI result and add an AI section to the card body:

Replace the existing `function buildTicketAlertCard(ticket: any, hoursInNewStatus: number)` with:

```typescript
function buildTicketAlertCard(ticket: any, hoursInNewStatus: number, aiResult?: LlmNewTicketOutput | null) {
    const createdAt = new Date(ticket.created_at).toLocaleString();
    const openTicketUrl = buildOpenTicketUrl(ticket);
    const description = typeof ticket.description === 'string' && ticket.description.trim().length > 0
        ? ticket.description.trim()
        : 'No description provided.';

    const card: any = {
        type: 'AdaptiveCard',
        $schema: 'http://adaptivecards.io/schemas/adaptive-card.json',
        version: '1.4',
        body: [
            {
                type: 'Container',
                style: hoursInNewStatus >= 6 ? 'attention' : 'emphasis',
                items: [
                    {
                        type: 'TextBlock',
                        text: 'Helpdesk Ticket Alert',
                        weight: 'bolder',
                        size: 'Large',
                        wrap: true
                    },
                    {
                        type: 'TextBlock',
                        text: `Ticket ${ticket.ticket_number} has remained in New status for ${hoursInNewStatus}+ hours.`,
                        spacing: 'Small',
                        wrap: true
                    }
                ]
            },
            {
                type: 'Container',
                spacing: 'Medium',
                items: [
                    {
                        type: 'TextBlock',
                        text: ticket.title,
                        weight: 'bolder',
                        size: 'Medium',
                        wrap: true
                    },
                    {
                        type: 'FactSet',
                        facts: [
                            { title: 'Priority', value: String(ticket.priority_name ?? 'Unknown') },
                            { title: 'Status', value: String(ticket.status_name ?? 'Unknown') },
                            { title: 'Queue', value: String(ticket.queue_name ?? 'Unknown') },
                            { title: 'Created', value: createdAt }
                        ]
                    },
                    {
                        type: 'TextBlock',
                        text: description,
                        wrap: true,
                        maxLines: 4,
                        isSubtle: true,
                        spacing: 'Small'
                    }
                ]
            }
        ]
    };

    if (aiResult) {
        const aiFacts: Array<{ title: string; value: string }> = [];
        if (aiResult.suggestedAssignee) {
            aiFacts.push(
                { title: 'Suggested Assignee', value: aiResult.suggestedAssignee.name },
                { title: 'Reason', value: aiResult.suggestedAssignee.reason },
            );
        }
        card.body.push({
            type: 'Container',
            style: 'accent',
            spacing: 'Medium',
            items: [
                { type: 'TextBlock', text: 'AI Diagnostic Summary', weight: 'Bolder', size: 'Small', wrap: true },
                { type: 'TextBlock', text: aiResult.diagnosticSummary, wrap: true, isSubtle: true, spacing: 'Small' },
                ...(aiFacts.length > 0 ? [{ type: 'FactSet', facts: aiFacts }] : []),
            ],
        });
    }

    if (openTicketUrl) {
        card.actions = [
            {
                type: 'Action.OpenUrl',
                title: 'Open Ticket',
                url: openTicketUrl
            }
        ];
    }

    return card;
}
```

In the `notifyAboutNewTicket` function, add the AI evaluation call after the status check (after the line `if (status_name !== 'New') { ... }`), and pass `aiResult` to `buildTicketAlertCard`:

Find this block (around line 119):
```typescript
    console.log(`New ticket: ${ticket.ticket_number} ...`);

    const conversations = await getRegisteredConversations();
    var alerted = false;
    for (const conversationId of conversations) {
        try {
            const alertCard = buildTicketAlertCard(ticket, hoursInNewStatus);
```

Replace with:
```typescript
    console.log(`New ticket: ${ticket.ticket_number} - ${ticket.title} (Status: ${status_name}, Priority: ${ticket.priority_name}, Queue: ${ticket.queue_name ?? 'Unknown'})`);

    // Call AI evaluation — failure here must not block the notification
    let aiResult: LlmNewTicketOutput | null = null;
    try {
        aiResult = await evaluateNewTicket(updatedTicket);
    } catch (error) {
        console.error(`AI evaluation failed for ticket ${ticket.ticket_number}, sending card without AI summary:`, error);
    }

    const conversations = await getRegisteredConversations();
    var alerted = false;
    for (const conversationId of conversations) {
        try {
            const alertCard = buildTicketAlertCard(ticket, hoursInNewStatus, aiResult);
```

- [ ] **Step 5.7: Run tests to confirm everything still passes**

Run: `npm test`
Expected: PASS — all tests green (newTicketNotification is not unit-tested yet, but no regressions)

- [ ] **Step 5.8: Commit**

```bash
git add src/llm/newTicketPrompt.ts src/evaluateNewTickets.ts src/jobs/newTicketNotification.ts src/types.ts tests/llm/newTicketPrompt.test.ts
git commit -m "feat: Scenario 1 — new ticket diagnostic summary with AI enrichment"
```

---

## Task 6: Scenario 2 — Stale Ticket Reminder

**Files:**
- Create: `src/llm/staleTicketPrompt.ts`
- Create: `src/jobs/staleTicketNotification.ts`
- Modify: `src/jobs/jobScheduler.ts` (register new job)
- Create: `tests/llm/staleTicketPrompt.test.ts`

**Interfaces:**
- Consumes: `callLlmJson`, `GPT4O_MINI` from `llmService`; `LlmStaleTicketInput`, `LlmStaleTicketOutput` from `types`
- Produces: `buildStaleTicketReminderMessage(input: LlmStaleTicketInput): Promise<LlmStaleTicketOutput>`

---

- [ ] **Step 6.1: Write the failing test**

Create `tests/llm/staleTicketPrompt.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

vi.mock('../../src/services/llmService', () => ({
  callLlmJson: vi.fn(),
  GPT4O_MINI: 'gpt-4o-mini',
}));

describe('staleTicketPrompt', () => {
  beforeEach(() => vi.clearAllMocks());

  it('calls callLlmJson with GPT4O_MINI and returns reminderMessage', async () => {
    const { callLlmJson } = await import('../../src/services/llmService');
    const mockOutput = { reminderMessage: 'Ticket T20260630.0042 has been open for 4 days...' };
    (callLlmJson as any).mockResolvedValueOnce(mockOutput);

    const { buildStaleTicketReminderMessage } = await import('../../src/llm/staleTicketPrompt');
    const input = {
      ticket: { id: 1, ticketNumber: 'T20260630.0042', title: 'Outlook not syncing', description: '', status: 'In Progress', daysSinceLastActivity: 4 },
      notes: [{ createdAt: '2026-06-30T09:00:00Z', title: 'Initial contact', description: 'Called customer' }],
      timeEntries: [],
    };

    const result = await buildStaleTicketReminderMessage(input);
    expect(result).toEqual(mockOutput);
    expect(callLlmJson).toHaveBeenCalledWith('gpt-4o-mini', expect.any(String), expect.stringContaining('T20260630.0042'));
  });
});
```

- [ ] **Step 6.2: Run test to confirm it fails**

Run: `npm test`
Expected: FAIL — `Cannot find module '../../src/llm/staleTicketPrompt'`

- [ ] **Step 6.3: Create src/llm/staleTicketPrompt.ts**

```typescript
import { LlmStaleTicketInput, LlmStaleTicketOutput } from '../types';
import { callLlmJson, GPT4O_MINI } from '../services/llmService';

const SYSTEM_PROMPT = `You are an IT support assistant for a managed service provider.
Write a concise (2–3 sentence) professional reminder message for the technician responsible for a stale helpdesk ticket.
Include the ticket number, title, days since last activity, and a brief summary of the last noted action.
Do not include a greeting or sign-off — just the reminder text.
Respond ONLY with valid JSON: { "reminderMessage": "..." }`;

export async function buildStaleTicketReminderMessage(
  input: LlmStaleTicketInput,
): Promise<LlmStaleTicketOutput> {
  return callLlmJson<LlmStaleTicketOutput>(GPT4O_MINI, SYSTEM_PROMPT, JSON.stringify(input));
}
```

- [ ] **Step 6.4: Run test to confirm it passes**

Run: `npm test`
Expected: PASS

- [ ] **Step 6.5: Create src/jobs/staleTicketNotification.ts**

```typescript
import getStaleTickets from '../getStaleTickets';
import { getTicketNotes, getTimeEntries } from '../autotaskApiCalls';
import { buildStaleTicketReminderMessage } from '../llm/staleTicketPrompt';
import { getRegisteredConversations, sendMessageToConversation } from '../conversations';
import { LlmStaleTicketInput } from '../types';
import { query } from '../db';
import { isWithinBusinessHours } from '../utils/businessHours';

const STALE_DAYS = parseInt(process.env.STALE_TICKET_DAYS ?? '3', 10);
const MAX_PER_RUN = parseInt(process.env.STALE_TICKET_MAX_PER_RUN ?? '5', 10);

export default async function staleTicketNotification(): Promise<void> {
  if (!isWithinBusinessHours(new Date())) return;

  console.log('Running stale ticket notification job');

  const conversations = await getRegisteredConversations();
  if (conversations.length === 0) {
    console.log('No registered conversations — skipping stale ticket notifications');
    return;
  }

  const staleTickets = await getStaleTickets(STALE_DAYS);
  const ticketsToProcess = staleTickets.slice(0, MAX_PER_RUN);
  console.log(`Processing ${ticketsToProcess.length} of ${staleTickets.length} stale tickets`);

  for (const ticket of ticketsToProcess) {
    try {
      const [notes, timeEntries, statusResult] = await Promise.all([
        getTicketNotes(ticket.id),
        getTimeEntries(ticket.id),
        query('SELECT name FROM ticket_statuses WHERE status_id = $1', [ticket.status]),
      ]);

      const statusName = statusResult.rows[0]?.name ?? String(ticket.status);
      const daysSinceLastActivity = Math.floor(
        (Date.now() - ticket.lastActivityDate.getTime()) / (1000 * 60 * 60 * 24),
      );

      const input: LlmStaleTicketInput = {
        ticket: {
          id: ticket.id,
          ticketNumber: ticket.ticketNumber,
          title: ticket.title,
          description: ticket.description ?? '',
          status: statusName,
          daysSinceLastActivity,
        },
        notes: notes.slice(0, 5).map(n => ({
          createdAt: n.createDateTime.toISOString(),
          title: n.title,
          description: n.description,
        })),
        timeEntries: timeEntries.slice(0, 5).map(t => ({
          startDateTime: t.startDateTime.toISOString(),
          hoursWorked: t.hoursWorked,
          summaryNotes: t.summaryNotes,
        })),
      };

      const result = await buildStaleTicketReminderMessage(input);

      for (const conversationId of conversations) {
        await sendMessageToConversation(conversationId, result.reminderMessage, true);
      }

      console.log(`Sent stale ticket reminder for ${ticket.ticketNumber}`);
    } catch (error) {
      console.error(`Error processing stale ticket ${ticket.ticketNumber}:`, error);
    }
  }
}
```

- [ ] **Step 6.6: Register the job in src/jobs/jobScheduler.ts**

Add the import at the top of `src/jobs/jobScheduler.ts` (after the existing imports):

```typescript
import staleTicketNotification from './staleTicketNotification';
```

Add the job registration inside `initializeJobScheduler()`, after the `automated-ticket-notification` block and before the closing `}`:

```typescript
        // Every hour during business hours, send reminders for stale tickets
        await boss.createQueue('stale-ticket-notification');
        await boss.work('stale-ticket-notification', async () => {
            try {
                await staleTicketNotification();
                console.log('Successfully ran stale ticket notification job');
            } catch (error) {
                console.error('Error running stale ticket notification job:', error);
            }
        });
        await boss.schedule('stale-ticket-notification', '0 * * * *', {}, { tz: timezone, retryLimit: 3 });
```

- [ ] **Step 6.7: Run tests**

Run: `npm test`
Expected: PASS

- [ ] **Step 6.8: Commit**

```bash
git add src/llm/staleTicketPrompt.ts src/jobs/staleTicketNotification.ts src/jobs/jobScheduler.ts tests/llm/staleTicketPrompt.test.ts
git commit -m "feat: Scenario 2 — stale ticket reminder via gpt-4o-mini"
```

---

## Task 7: Scenario 3 — Tech Reply Interpretation

**Files:**
- Create: `src/llm/techReplyPrompt.ts`
- Modify: `src/index.ts` (replace echo handler with intent-based handler)
- Create: `tests/llm/techReplyPrompt.test.ts`

**Interfaces:**
- Consumes: `callLlmJson`, `GPT4O` from `llmService`; `LlmTechReplyInput`, `LlmTechReplyOutput` from `types`
- Produces: `interpretTechReply(input: LlmTechReplyInput): Promise<LlmTechReplyOutput>`

---

- [ ] **Step 7.1: Write the failing test**

Create `tests/llm/techReplyPrompt.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

vi.mock('../../src/services/llmService', () => ({
  callLlmJson: vi.fn(),
  GPT4O: 'gpt-4o',
}));

describe('techReplyPrompt', () => {
  beforeEach(() => vi.clearAllMocks());

  it('calls callLlmJson with GPT4O and returns parsed LlmTechReplyOutput', async () => {
    const { callLlmJson } = await import('../../src/services/llmService');
    const mockOutput = {
      action: 'add_note_and_change_status',
      note: 'Called customer. Issue resolved after reboot.',
      statusId: 5,
      newAssigneeId: null,
      explanation: 'Tech reports resolution, closing ticket.',
    };
    (callLlmJson as any).mockResolvedValueOnce(mockOutput);

    const { interpretTechReply } = await import('../../src/llm/techReplyPrompt');
    const input = {
      techMessage: 'T20260703.0001 — called the customer, issue fixed after reboot. Closing.',
      ticket: { id: 1, ticketNumber: 'T20260703.0001', title: 'Outlook not opening', description: '', currentStatus: 'In Progress' },
      availableStatuses: [{ id: 5, name: 'Complete' }],
      availableResources: [],
    };

    const result = await interpretTechReply(input);
    expect(result.action).toBe('add_note_and_change_status');
    expect(result.statusId).toBe(5);
    expect(callLlmJson).toHaveBeenCalledWith('gpt-4o', expect.stringContaining('managed service provider'), expect.stringContaining('T20260703.0001'));
  });

  it('passes all input fields to the LLM', async () => {
    const { callLlmJson } = await import('../../src/services/llmService');
    (callLlmJson as any).mockResolvedValueOnce({ action: 'no_action', note: null, statusId: null, newAssigneeId: null, explanation: 'No clear action.' });

    const { interpretTechReply } = await import('../../src/llm/techReplyPrompt');
    const input = {
      techMessage: 'T20260703.0002 reassign to Bob',
      ticket: { id: 2, ticketNumber: 'T20260703.0002', title: 'Slow PC', description: 'Machine takes 5 min to boot', currentStatus: 'New' },
      availableStatuses: [{ id: 1, name: 'New' }, { id: 5, name: 'Complete' }],
      availableResources: [{ id: 9, name: 'Bob Jones' }],
    };

    await interpretTechReply(input);
    const userContent = (callLlmJson as any).mock.calls[0][2];
    expect(userContent).toContain('Bob Jones');
    expect(userContent).toContain('Slow PC');
    expect(userContent).toContain('Complete');
  });
});
```

- [ ] **Step 7.2: Run test to confirm it fails**

Run: `npm test`
Expected: FAIL — `Cannot find module '../../src/llm/techReplyPrompt'`

- [ ] **Step 7.3: Create src/llm/techReplyPrompt.ts**

```typescript
import { LlmTechReplyInput, LlmTechReplyOutput } from '../types';
import { callLlmJson, GPT4O } from '../services/llmService';

const SYSTEM_PROMPT = `You are an IT service desk assistant for a managed service provider.
A technician has sent a natural language message referencing a helpdesk ticket.
Determine the intended action to take in the ticketing system and return a structured response.

Valid actions:
- "add_note": Add a note to the ticket (tech provides an update but no status change)
- "change_status": Change ticket status only (no additional note text)
- "reassign": Reassign ticket to a different technician
- "add_note_and_change_status": Add a note AND change the ticket status (most common for closures/updates)
- "no_action": Nothing to do (question, vague message, or acknowledgment only)

Rules:
- Set "note" to the substantive update text extracted from the tech's message (or null if none)
- Set "statusId" only to an id from availableStatuses (or null)
- Set "newAssigneeId" only to an id from availableResources (or null)
- Unused fields must be null — never omit them

Respond ONLY with valid JSON:
{
  "action": "<one of the five actions>",
  "note": "<note text or null>",
  "statusId": <number or null>,
  "newAssigneeId": <number or null>,
  "explanation": "<one sentence explaining the decision>"
}`;

export async function interpretTechReply(input: LlmTechReplyInput): Promise<LlmTechReplyOutput> {
  return callLlmJson<LlmTechReplyOutput>(GPT4O, SYSTEM_PROMPT, JSON.stringify(input));
}
```

- [ ] **Step 7.4: Run test to confirm it passes**

Run: `npm test`
Expected: PASS

- [ ] **Step 7.5: Update Teams message handler in src/index.ts**

Add imports after the existing imports in `src/index.ts`:

```typescript
import { interpretTechReply } from './llm/techReplyPrompt';
import { getResources, addTicketNote, updateTicketStatus, updateTicketAssignee } from './autotaskApiCalls';
import { LlmTechReplyInput } from './types';
```

Replace the entire `teamsApp.on('message', ...)` block with:

```typescript
const TICKET_NUMBER_RE = /\bT\d{8}\.\d{4}\b/i;

teamsApp.on('message', async ({ send, activity }) => {
    console.log(`Received message from ${activity.from.name}: ${activity.text}`);

    await addConversationIfNotExists(activity.conversation.id);
    addMessageToConversation(activity.conversation.id, activity.text, activity.from.name).catch(error => {
        console.error('Error adding message to conversation:', error);
    });

    const match = activity.text?.match(TICKET_NUMBER_RE);
    if (!match) {
        const helpMsg = `To update a ticket, include its number in your message — e.g., "T20260703.0001 — called customer, issue resolved, closing."`;
        await send(helpMsg);
        addMessageToConversation(activity.conversation.id, helpMsg, 'Bot').catch(() => {});
        return;
    }

    const ticketNumber = match[0].toUpperCase();
    try {
        const ticketResult = await query(
            `SELECT t.ticket_id, t.ticket_number, t.title, t.description, t.status_id, s.name AS status_name
             FROM tickets t
             JOIN ticket_statuses s ON t.status_id = s.status_id
             WHERE UPPER(t.ticket_number) = $1`,
            [ticketNumber],
        );

        if (ticketResult.rowCount === 0) {
            const notFoundMsg = `Ticket ${ticketNumber} was not found. Please verify the ticket number and try again.`;
            await send(notFoundMsg);
            addMessageToConversation(activity.conversation.id, notFoundMsg, 'Bot').catch(() => {});
            return;
        }

        const dbTicket = ticketResult.rows[0];
        const [statusesResult, resources] = await Promise.all([
            query('SELECT status_id AS id, name FROM ticket_statuses'),
            getResources(),
        ]);

        const input: LlmTechReplyInput = {
            techMessage: activity.text,
            ticket: {
                id: dbTicket.ticket_id,
                ticketNumber: dbTicket.ticket_number,
                title: dbTicket.title,
                description: dbTicket.description ?? '',
                currentStatus: dbTicket.status_name,
            },
            availableStatuses: statusesResult.rows,
            availableResources: resources.map(r => ({ id: r.id, name: r.name })),
        };

        const result = await interpretTechReply(input);
        console.log(`LLM action for ${ticketNumber}: ${result.action} — ${result.explanation}`);

        if ((result.action === 'add_note' || result.action === 'add_note_and_change_status') && result.note) {
            await addTicketNote(dbTicket.ticket_id, `Update from ${activity.from.name}`, result.note);
        }
        if ((result.action === 'change_status' || result.action === 'add_note_and_change_status') && result.statusId) {
            await updateTicketStatus(dbTicket.ticket_id, result.statusId);
        }
        if (result.action === 'reassign' && result.newAssigneeId) {
            await updateTicketAssignee(dbTicket.ticket_id, result.newAssigneeId);
        }

        const confirmMsg = result.action === 'no_action'
            ? `Understood — no changes made to ${ticketNumber}. (${result.explanation})`
            : `Done! ${result.explanation}`;
        await send(confirmMsg);
        addMessageToConversation(activity.conversation.id, confirmMsg, 'Bot').catch(() => {});
    } catch (error) {
        console.error(`Error processing tech reply for ${ticketNumber}:`, error);
        const errMsg = `Something went wrong while updating ${ticketNumber}. Please check the ticket manually.`;
        await send(errMsg);
        addMessageToConversation(activity.conversation.id, errMsg, 'Bot').catch(() => {});
    }
});
```

- [ ] **Step 7.6: Run all tests**

Run: `npm test`
Expected: PASS — all tests across all tasks green

- [ ] **Step 7.7: Build to verify TypeScript compilation**

Run: `npm run build`
Expected: no TypeScript errors

- [ ] **Step 7.8: Commit**

```bash
git add src/llm/techReplyPrompt.ts src/index.ts tests/llm/techReplyPrompt.test.ts
git commit -m "feat: Scenario 3 — interpret tech Teams replies and act on Autotask"
```

---

## Gaps and Known Limitations

### Not addressed in this plan (conscious scope decisions)

| Gap | Reason not included | How to address later |
|-----|---------------------|----------------------|
| IT Glue has no company-scoped search | `companyId` not stored in `tickets` table | Add `company_id` column, populate from Autotask `Companies` endpoint, pass to `searchItGlueArticles` |
| Datto RMM device matching is heuristic | No device-to-ticket link in Autotask data | Add `configurationItemId` to ticket IncludeFields; look up device UID directly |
| `addHoursToDate` in `businessHours.ts` has a bug | Out of scope for LLM integration | The bug: line 45 adds hours a second time; replace with a single `new Date(date.getTime() + hours * 3600000)` |
| Duplicate `test-job` registration in `jobScheduler.ts` | Out of scope | Remove the second registration block (lines 53–59) |
| Stale ticket notification sends to all conversations | No per-tech routing | Build 1:1 conversation lookup keyed by `assignedResourceId` when tech contact info is available |
| No rate-limit handling on Azure OpenAI | Acceptable at current volume | Add exponential backoff with `retry-after` header parsing if 429s occur |
| The `node-cron` dependency is installed but unused | Out of scope | Remove from `package.json` |

---

## Self-Review

**Spec coverage check:**
- ✅ Scenario 1 (new ticket diagnostic summary + assignee suggestion): Task 5
- ✅ Scenario 2 (stale ticket Teams reminder): Task 6
- ✅ Scenario 3 (inbound Teams reply → Autotask action): Task 7
- ✅ IT Glue KB article lookup: Task 2 + used in Task 5
- ✅ Datto RMM device data: Task 3 + used in Task 5
- ✅ Azure OpenAI via HTTP (using `openai` SDK already in package.json): Task 1
- ✅ gpt-4o for Scenarios 1 and 3; gpt-4o-mini for Scenario 2: confirmed in prompt files
- ✅ Structured LLM output so backend can parse and act: all three prompt modules
- ✅ Where to insert LLM calls: Tasks 5–7

**Placeholder scan:** None found. All steps contain runnable code.

**Type consistency check:**
- `LlmNewTicketInput` defined in Task 4, consumed in Tasks 5 ✅
- `LlmStaleTicketInput` defined in Task 4, consumed in Task 6 ✅
- `LlmTechReplyInput` defined in Task 4, consumed in Task 7 ✅
- `callLlmJson`, `GPT4O`, `GPT4O_MINI` defined in Task 1, consumed in Tasks 5–7 ✅
- `getResources`, `addTicketNote`, `updateTicketStatus`, `updateTicketAssignee` defined in Task 4, consumed in Tasks 5 and 7 ✅
- `searchItGlueArticles` defined in Task 2, consumed in Task 5 ✅
- `getDeviceByHostname` defined in Task 3, consumed in Task 5 ✅
