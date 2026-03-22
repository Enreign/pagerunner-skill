# Advanced Topics

Expert patterns, optimization, and integration deep-dives.

---

## Multi-Agent Coordination via KV Store

### Pattern: Request-Response Queue

Agent A publishes a request, Agent B processes it, stores result.

```javascript
// Agent A: Request
await kv_set("workflow", `request_${Date.now()}`, JSON.stringify({
  type: "scrape_url",
  url: "https://example.com",
  timestamp: new Date().toISOString()
}));

// Agent B: Poll and process
async function processRequests() {
  const requests = await kv_list("workflow", prefix: "request_");
  
  for (const { key, value } of requests) {
    const request = JSON.parse(value);
    
    // Process
    const result = await scrapeUrl(request.url);
    
    // Store result with same key
    await kv_set("workflow", key.replace("request_", "result_"), JSON.stringify(result));
    
    // Mark processed
    await kv_delete("workflow", key);
  }
}
```

### Pattern: Checkpoint-Based Resumption

Large job can be resumed from last checkpoint.

```javascript
// Agent A: Scrape 1000 items with checkpoints
async function scrapeWithCheckpoints() {
  const lastCheckpoint = JSON.parse(await kv_get("pipeline", "checkpoint") || '{"index": 0}');
  const items = await fetchItemList();
  
  for (let i = lastCheckpoint.index; i < items.length; i++) {
    const item = items[i];
    const data = await scrapeItem(item);
    
    await kv_set("pipeline", `item_${i}`, JSON.stringify(data));
    
    // Save checkpoint every 100 items
    if (i % 100 === 0) {
      await kv_set("pipeline", "checkpoint", JSON.stringify({ index: i + 1 }));
    }
  }
  
  // Cleanup
  await kv_delete("pipeline", "checkpoint");
}
```

---

## Daemon Mode & Multi-Client Coordination

### Setup

```bash
# Start daemon once (persistent, always-on)
pagerunner daemon &

# All `pagerunner mcp` instances auto-detect daemon
# No manual coordination needed
```

### Verification

```bash
# See daemon process
ps aux | grep pagerunner

# Daemon logs
tail -f ~/.pagerunner/daemon.log

# Multiple MCP clients share state
pagerunner list-sessions  # Sees all active sessions across all clients
```

---

## Performance Optimization

### Batch Operations

Instead of sequential waits:

```javascript
// ❌ SLOW — wait after each action
for (const url of urls) {
  await navigate(sessionId, tabId, url);
  await wait_for(sessionId, tabId, selector: ".content", ms: 5000);
  const content = await get_content(sessionId, tabId);
}

// ✅ FAST — parallelize where possible
const promises = urls.map(async url => {
  const s = await open_session({ profile: "scraper" });
  const [tab] = await list_tabs(s);
  await navigate(s, tab.target_id, url);
  await wait_for(s, tab.target_id, selector: ".content");
  const content = await get_content(s, tab.target_id);
  await close_session(s);
  return content;
});

const results = await Promise.all(promises);
```

### Efficient Selectors

```javascript
// ❌ SLOW — traverses entire DOM
const count = document.querySelectorAll('div').filter(el => el.textContent.includes('Item')).length;

// ✅ FAST — targeted query
const count = document.querySelectorAll('[data-testid="item"]').length;
```

### Caching Results

```javascript
// ❌ SLOW — re-evaluate every time
const count1 = await evaluate(..., "document.querySelectorAll('.item').length");
const count2 = await evaluate(..., "document.querySelectorAll('.item').length");

// ✅ FAST — evaluate once, cache
const count = await evaluate(..., "document.querySelectorAll('.item').length");
// Use cached value multiple times
```

---

## Security Patterns (Beyond SECURITY.md)

### Role-Based Access

Different agent profiles for different roles:

```toml
[[profiles]]
name = "agent-readonly"
# Limited to read-only operations
user_data_dir = "/path/to/readonly/profile"

[[profiles]]
name = "agent-write"
# Can fill forms, submit
user_data_dir = "/path/to/write/profile"
```

### Token Rotation

Periodically refresh snapshots:

```javascript
// Every 24 hours
setInterval(async () => {
  const sessionId = await open_session({ profile: "agent-work" });
  const [tab] = await list_tabs(sessionId);
  
  await navigate(sessionId, tab.target_id, "https://app.example.com");
  // Force re-auth if session expired
  await evaluate(sessionId, tab.target_id, "document.querySelector('.logout-btn')?.click()");
  
  // Re-login
  await fill(sessionId, tab.target_id, "input[type='email']", agentEmail);
  // ... complete login ...
  
  // Update snapshot
  await save_snapshot(sessionId, tab.target_id, origin: "https://app.example.com");
  await close_session(sessionId);
}, 24 * 60 * 60 * 1000);
```

---

## Debugging Advanced Workflows

### Performance Profiling

```javascript
// Measure each operation
const start = Date.now();

await navigate(...);
console.log(`navigate: ${Date.now() - start}ms`);

await wait_for(...);
console.log(`wait_for: ${Date.now() - start}ms`);

await get_content(...);
console.log(`get_content: ${Date.now() - start}ms`);
```

### State Inspection

```javascript
// Inspect internal state at any point
const state = await evaluate(sessionId, tabId, `
  ({
    url: window.location.href,
    title: document.title,
    memoryUsage: performance.memory?.usedJSHeapSize || 'N/A',
    sessionState: window.__STORE__.getState()
  })
`);

console.log("Current state:", state);
```

---

## Integration with External Services

### API Calls with Auth

```javascript
// Use browser context to make authenticated API calls
const response = await evaluate(sessionId, tabId, `
  fetch('https://api.example.com/data', {
    headers: {
      'Authorization': 'Bearer ' + window.__AUTH_TOKEN__
    }
  }).then(r => r.json())
`);
```

### Database Operations

```javascript
// After scraping, insert into database
const scrapedData = await get_content(sessionId, tabId);

await fetch('https://internal-api.company.com/data', {
  method: 'POST',
  body: JSON.stringify({ data: scrapedData }),
  headers: { 'Authorization': `Bearer ${process.env.API_KEY}` }
});
```

---

## Troubleshooting Advanced Issues

### Memory Leaks

If daemon memory grows over time:

```bash
# Check daemon memory
ps aux | grep pagerunner | grep -v grep

# Restart daemon to free memory
pkill -f "pagerunner daemon"
pagerunner daemon &
```

### Stale Sessions

Sessions should be closed after use:

```javascript
// ❌ WRONG — session never closed
const sessionId = await open_session(...);
// ... do work ...
// forgot to close_session

// ✅ RIGHT — always close
try {
  const sessionId = await open_session(...);
  // ... do work ...
} finally {
  await close_session(sessionId);
}
```

---

## Best Practices Checklist

- [ ] Use daemon mode for multi-agent workflows
- [ ] Store credentials in KV, not code
- [ ] Close sessions when done (use try/finally)
- [ ] Cache results when possible
- [ ] Monitor daemon logs for errors
- [ ] Use specific selectors (not DOM traversal)
- [ ] Parallelize independent operations
- [ ] Checkpoint long-running jobs
- [ ] Test with sample data first
- [ ] Enable audit logging for security-sensitive workflows

---

## Next Steps

- Implement KV-based coordination for your multi-agent pipeline
- Profile performance bottlenecks
- Set up daemon for always-on automation
- Integrate with external APIs and databases

See EXAMPLES.md for multi-agent patterns in action.
