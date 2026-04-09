# Replication API Quick Reference

Quick reference for AEM 6.5 LTS Replication API methods and interfaces.

## Core Interfaces

### Replicator

**Package:** `com.day.cq.replication.Replicator`

**Purpose:** Core interface for replication operations

| Method | Parameters | Returns | Use Case |
|--------|-----------|---------|----------|
| `replicate()` | Session, Type, String | void | Simple activation/deactivation |
| `replicate()` | Session, Type, String, Options | void | Activation with custom options |
| `replicate()` | Session, Type, String[], Options | void | Bulk activation of multiple paths |
| `checkPermission()` | Session, Type, String | void | Verify user can replicate (throws if denied) |
| `getReplicationStatus()` | Session, String | ReplicationStatus | Query replication state of resource |
| `getActivatedPaths()` | Session, String | Iterator<String> | List all activated paths under root |

### ReplicationOptions

**Package:** `com.day.cq.replication.ReplicationOptions`

**Purpose:** Configure replication behavior

| Method | Parameter | Default | Use Case |
|--------|-----------|---------|----------|
| `setSynchronous()` | boolean | false | Wait for replication to complete |
| `setSuppressVersions()` | boolean | false | Don't create version on activation |
| `setSuppressStatusUpdate()` | boolean | false | Don't update replication metadata |
| `setFilter()` | AgentFilter | null | Target specific agents only |
| `setRevision()` | String | null | Replicate specific version |

### ReplicationStatus

**Package:** `com.day.cq.replication.ReplicationStatus`

**Purpose:** Query replication state

| Method | Returns | Description |
|--------|---------|-------------|
| `isActivated()` | boolean | True if replicated to at least one agent |
| `isDeactivated()` | boolean | True if deactivated |
| `getLastPublished()` | Calendar | Last activation timestamp |
| `getLastPublishedBy()` | String | User who last activated |
| `getLastReplicationAction()` | String | Last action (ACTIVATE, DEACTIVATE, etc.) |
| `getLastReplicatedRevision()` | String | Version ID last replicated |

### AgentManager

**Package:** `com.day.cq.replication.AgentManager`

**Purpose:** Inspect and manage agents

| Method | Parameters | Returns | Use Case |
|--------|-----------|---------|----------|
| `getAgents()` | - | Map<String, Agent> | Get all agents |
| `getAgents()` | AgentFilter | Map<String, Agent> | Get filtered agents |
| `getAgent()` | String agentId | Agent | Get specific agent |

### ReplicationQueue

**Package:** `com.day.cq.replication.ReplicationQueue`

**Purpose:** Inspect queue state (read-only via JMX)

| Method | Returns | Description |
|--------|---------|-------------|
| `getEntries()` | Collection<ReplicationQueueItem> | All queued items |
| `getSize()` | int | Queue depth |
| `getState()` | State | IDLE, RUNNING, BLOCKED, PAUSED |

### ReplicationListener

**Package:** `com.day.cq.replication.ReplicationListener`

**Purpose:** Monitor replication events

| Method | Parameters | Use Case |
|--------|-----------|----------|
| `onStart()` | Agent, ReplicationAction | Replication initiated |
| `onEnd()` | Agent, ReplicationAction, ReplicationResult | Replication completed |
| `onError()` | Agent, ReplicationAction, Exception | Replication failed |

## ReplicationActionType Enum

**Package:** `com.day.cq.replication.ReplicationActionType`

| Constant | Purpose |
|----------|---------|
| `ACTIVATE` | Publish content to target instances |
| `DEACTIVATE` | Remove content from target instances |
| `DELETE` | Delete content on target (use with caution) |
| `TEST` | Test agent connectivity without replicating |
| `INTERNAL_POLL` | Internal use (reverse replication) |

## Exception Handling

### ReplicationException

**Package:** `com.day.cq.replication.ReplicationException`

**Common Causes:**
- Network connectivity failure
- Authentication error
- Target instance unavailable
- Invalid resource path
- Insufficient permissions

**Handling Pattern:**
```java
try {
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
} catch (ReplicationException e) {
    LOG.error("Replication failed for path: {}", path, e);
    // Handle error (retry, alert, log)
}
```

## Common Patterns

### Simple Activation

```java
@Reference
private Replicator replicator;

public void activate(ResourceResolver resolver, String path) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    if (session == null) {
        throw new IllegalStateException("Unable to adapt to Session");
    }
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
}
```

### Synchronous Activation

```java
public void activateSync(ResourceResolver resolver, String path) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    if (session == null) {
        throw new IllegalStateException("Unable to adapt to Session");
    }
    
    ReplicationOptions opts = new ReplicationOptions();
    opts.setSynchronous(true);
    
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path, opts);
}
```

### Bulk Activation

```java
public void activateBulk(ResourceResolver resolver, List<String> paths) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    if (session == null) {
        throw new IllegalStateException("Unable to adapt to Session");
    }
    
    ReplicationOptions opts = new ReplicationOptions();
    opts.setSynchronous(false); // Async for performance
    
    String[] pathArray = paths.toArray(new String[0]);
    replicator.replicate(session, ReplicationActionType.ACTIVATE, pathArray, opts);
}
```

### Agent Filtering

```java
public void activateToSpecificAgent(ResourceResolver resolver, String path, 
                                     String targetAgentId) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    if (session == null) {
        throw new IllegalStateException("Unable to adapt to Session");
    }
    
    ReplicationOptions opts = new ReplicationOptions();
    opts.setFilter(agent -> agent.getId().equals(targetAgentId));
    
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path, opts);
}
```

### Check Replication Status

```java
public boolean isActivated(ResourceResolver resolver, String path) {
    Session session = resolver.adaptTo(Session.class);
    if (session == null) {
        return false;
    }
    
    ReplicationStatus status = replicator.getReplicationStatus(session, path);
    return status != null && status.isActivated();
}
```

### Permission Check

```java
public boolean canActivate(ResourceResolver resolver, String path) {
    Session session = resolver.adaptTo(Session.class);
    if (session == null) {
        return false;
    }
    
    try {
        replicator.checkPermission(session, ReplicationActionType.ACTIVATE, path);
        return true;
    } catch (ReplicationException e) {
        return false; // User lacks permission
    }
}
```

## Service User Pattern

```java
@Reference
private ResourceResolverFactory resolverFactory;

@Reference
private Replicator replicator;

public void activateWithServiceUser(String path) throws ReplicationException {
    Map<String, Object> authInfo = new HashMap<>();
    authInfo.put(ResourceResolverFactory.SUBSERVICE, "replication-service");
    
    try (ResourceResolver resolver = resolverFactory.getServiceResourceResolver(authInfo)) {
        Session session = resolver.adaptTo(Session.class);
        if (session == null) {
            throw new IllegalStateException("Unable to adapt to Session");
        }
        replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
    } catch (LoginException e) {
        LOG.error("Service user login failed", e);
        throw new ReplicationException("Authentication failed", e);
    }
}
```

## Official Documentation

- **JavaDoc:** https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/package-summary.html
- **User Guide:** https://experienceleague.adobe.com/en/docs/experience-manager-65-lts/content/implementing/deploying/configuring/replication
- **Troubleshooting:** https://experienceleague.adobe.com/en/docs/experience-manager-65-lts/content/implementing/deploying/configuring/troubleshoot-rep
