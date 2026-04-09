---
name: replication-api
description: Use the AEM 6.5 LTS Replication API for programmatic content activation, deactivation, and replication status management
---

# AEM 6.5 LTS Replication API

This skill provides comprehensive guidance on using the official Adobe Experience Manager 6.5 LTS Replication API for programmatic replication operations. The API enables custom code to activate, deactivate, and manage content distribution workflows.

## When to Use This Skill

Use this skill when you need to:
- Programmatically activate/deactivate content from custom code
- Build custom replication workflows in OSGi services or servlets
- Integrate replication with external systems
- Implement custom replication triggers and automation
- Check replication status in application logic
- Create custom content distribution tools
- Bulk replicate content via scripts
- Implement conditional replication logic

## Prerequisites

- AEM 6.5 LTS environment with configured replication agents
- Java development environment for AEM
- Maven project with AEM dependencies
- Understanding of OSGi services and Sling ResourceResolver
- Configured replication agents (see `configure-replication-agent` skill)
- Service user or user session with replication permissions

## Official API Documentation

All public replication APIs are documented in the official Adobe AEM 6.5 LTS JavaDoc:

- **Package Summary (Complete API Reference)**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/package-summary.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/package-summary.html)
- **Replicator Interface**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/Replicator.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/Replicator.html)
- **AgentManager Interface**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/AgentManager.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/AgentManager.html)
- **ReplicationQueue Interface**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationQueue.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationQueue.html)
- **ReplicationListener Interface**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationListener.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationListener.html)
- **AgentFilter Interface**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/AgentFilter.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/AgentFilter.html)
- **Agent Interface**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/Agent.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/Agent.html)
- **ReplicationAction Class**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationAction.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationAction.html)
- **ReplicationOptions Class**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationOptions.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationOptions.html)
- **ReplicationStatus Interface**: [https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationStatus.html](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationStatus.html)

## Core API Components

### 1. Replicator Interface

The primary service for managing replication operations.

**Service Type:** OSGi Service  
**Package:** `com.day.cq.replication`  
**Interface:** `com.day.cq.replication.Replicator`

### 2. ReplicationActionType Enum

Defines the type of replication operation:

| Type | Purpose | Effect |
|------|---------|--------|
| `ACTIVATE` | Publish content | Sends to Publish instances |
| `DEACTIVATE` | Unpublish content | Removes from Publish |
| `DELETE` | Delete from Publish | Permanent removal |
| `TEST` | Test replication | Verifies connectivity |
| `REVERSE` | Reverse replicate | Publish → Author |
| `INTERNAL_POLL` | Internal polling | System use |

### 3. ReplicationOptions Class

Encapsulates optional parameters for replication requests.

### 4. ReplicationStatus Interface

Provides status information about replicated content.

## Maven Dependencies

Add AEM API dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.adobe.aem</groupId>
    <artifactId>uber-jar</artifactId>
    <version>6.5.0</version>
    <classifier>apis</classifier>
    <scope>provided</scope>
</dependency>
```

Or for individual API:

```xml
<dependency>
    <groupId>com.day.cq</groupId>
    <artifactId>cq-replication</artifactId>
    <version>6.5.0</version>
    <scope>provided</scope>
</dependency>
```

## Replicator Interface Methods

### Method 1: replicate(Session, ReplicationActionType, String)

**Signature:**
```java
void replicate(Session session, 
               ReplicationActionType type, 
               String path) 
throws ReplicationException
```

**Parameters:**
- `session` - JCR session (user permissions determine access)
- `type` - ReplicationActionType (ACTIVATE, DEACTIVATE, DELETE, etc.)
- `path` - Content path to replicate (e.g., "/content/mysite/en/page")

**Throws:** `ReplicationException` if replication fails

**Description:** Triggers replication for a single path with default options.

**Example:**
```java
import com.day.cq.replication.Replicator;
import com.day.cq.replication.ReplicationActionType;
import com.day.cq.replication.ReplicationException;
import org.apache.sling.api.resource.ResourceResolver;
import org.osgi.service.component.annotations.Reference;

import javax.jcr.Session;

@Reference
private Replicator replicator;

public void activatePage(ResourceResolver resolver, String pagePath) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    replicator.replicate(session, ReplicationActionType.ACTIVATE, pagePath);
}
```

### Method 2: replicate(Session, ReplicationActionType, String, ReplicationOptions)

**Signature:**
```java
void replicate(Session session, 
               ReplicationActionType type, 
               String path,
               ReplicationOptions options) 
throws ReplicationException
```

**Parameters:**
- `session` - JCR session
- `type` - ReplicationActionType
- `path` - Content path
- `options` - ReplicationOptions for custom configuration

**Description:** Initiates replication with customizable options for one path.

**Example:**
```java
public void activatePageSync(ResourceResolver resolver, String pagePath) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    
    ReplicationOptions opts = new ReplicationOptions();
    opts.setSynchronous(true); // Wait for completion
    opts.setSuppressVersions(true); // Don't create versions
    
    replicator.replicate(session, ReplicationActionType.ACTIVATE, pagePath, opts);
}
```

### Method 3: replicate(Session, ReplicationActionType, String[], ReplicationOptions)

**Signature:**
```java
void replicate(Session session, 
               ReplicationActionType type, 
               String[] paths,
               ReplicationOptions options) 
throws ReplicationException
```

**Parameters:**
- `session` - JCR session
- `type` - ReplicationActionType
- `paths` - Array of content paths (String[])
- `options` - ReplicationOptions

**Description:** Handles replication across multiple paths with supplied settings.

**Example:**
```java
public void activateMultiplePages(ResourceResolver resolver, String[] pagePaths) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    
    ReplicationOptions opts = new ReplicationOptions();
    opts.setSynchronous(false); // Async for better performance
    
    replicator.replicate(session, ReplicationActionType.ACTIVATE, pagePaths, opts);
}
```

### Method 4: checkPermission(Session, ReplicationActionType, String)

**Signature:**
```java
void checkPermission(Session session, 
                     ReplicationActionType type, 
                     String path) 
throws ReplicationException
```

**Parameters:**
- `session` - JCR session
- `type` - ReplicationActionType
- `path` - Content path

**Throws:** `ReplicationException` if user lacks permissions

**Description:** Verifies whether a user has sufficient permissions for replication activities.

**Example:**
```java
public boolean canUserActivate(ResourceResolver resolver, String pagePath) {
    Session session = resolver.adaptTo(Session.class);
    try {
        replicator.checkPermission(session, ReplicationActionType.ACTIVATE, pagePath);
        return true; // User has permission
    } catch (ReplicationException e) {
        return false; // User lacks permission
    }
}
```

### Method 5: getReplicationStatus(Session, String)

**Signature:**
```java
ReplicationStatus getReplicationStatus(Session session, String path)
```

**Parameters:**
- `session` - JCR session
- `path` - Content path

**Returns:** `ReplicationStatus` object or `null` if unavailable

**Description:** Retrieves the replication status for a given path.

**Example:**
```java
import com.day.cq.replication.ReplicationStatus;
import java.util.Calendar;

public ReplicationInfo getPageReplicationInfo(ResourceResolver resolver, String pagePath) {
    Session session = resolver.adaptTo(Session.class);
    ReplicationStatus status = replicator.getReplicationStatus(session, pagePath);
    
    if (status != null) {
        boolean isActivated = status.isActivated();
        Calendar lastPublished = status.getLastPublished();
        String lastPublishedBy = status.getLastPublishedBy();
        
        return new ReplicationInfo(isActivated, lastPublished, lastPublishedBy);
    }
    return null;
}
```

### Method 6: getActivatedPaths(Session, String)

**Signature:**
```java
Iterator<String> getActivatedPaths(Session session, String path) 
throws ReplicationException
```

**Parameters:**
- `session` - JCR session
- `path` - Root path for subtree

**Returns:** Iterator of activated paths

**Throws:** `ReplicationException`

**Description:** Returns paths of all activated nodes for the given subtree path.

**Example:**
```java
import java.util.Iterator;
import java.util.ArrayList;
import java.util.List;

public List<String> getActivatedPages(ResourceResolver resolver, String rootPath) 
    throws ReplicationException {
    Session session = resolver.adaptTo(Session.class);
    Iterator<String> activatedPaths = replicator.getActivatedPaths(session, rootPath);
    
    List<String> result = new ArrayList<>();
    while (activatedPaths.hasNext()) {
        result.add(activatedPaths.next());
    }
    return result;
}
```

## ReplicationOptions Class

Encapsulates optional configuration parameters for replication requests.

**Official Documentation:** [ReplicationOptions JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationOptions.html)

### Key Methods:

#### setSynchronous(boolean)

**Description:** Controls whether replication executes synchronously (blocking) or asynchronously (default).

**Parameters:**
- `synchronous` - `true` for synchronous, `false` for asynchronous (default)

**Example:**
```java
ReplicationOptions opts = new ReplicationOptions();
opts.setSynchronous(true); // Wait for replication to complete
```

**Use cases:**
- Synchronous: When you need confirmation before proceeding (e.g., before redirecting user)
- Asynchronous: Better performance for bulk operations

#### setFilter(AgentFilter)

**Description:** Sets the filter for selecting specific agents for replication.

**Parameters:**
- `filter` - `AgentFilter` implementation

**Example:**
```java
import com.day.cq.replication.AgentFilter;
import com.day.cq.replication.Agent;

ReplicationOptions opts = new ReplicationOptions();

// Filter to specific agent
opts.setFilter(new AgentFilter() {
    public boolean isIncluded(Agent agent) {
        return agent.getId().equals("publish_instance_1");
    }
});

replicator.replicate(session, ReplicationActionType.ACTIVATE, pagePath, opts);
```

**Use cases:**
- Target specific publish instance
- Exclude certain agents
- Route content to specific environments

#### setSuppressVersions(boolean)

**Description:** Controls whether to create versions during replication.

**Parameters:**
- `suppress` - `true` to skip version creation, `false` otherwise

**Example:**
```java
ReplicationOptions opts = new ReplicationOptions();
opts.setSuppressVersions(true); // Don't create versions (performance)
```

#### setSuppressStatusUpdate(boolean)

**Description:** Controls whether to update replication status.

**Parameters:**
- `suppress` - `true` to skip status update, `false` otherwise

**Example:**
```java
ReplicationOptions opts = new ReplicationOptions();
opts.setSuppressStatusUpdate(true); // Don't update status metadata
```

#### setRevision(String)

**Description:** Sets the specific revision to replicate.

**Parameters:**
- `revision` - Revision identifier

**Example:**
```java
ReplicationOptions opts = new ReplicationOptions();
opts.setRevision("1.0");
replicator.replicate(session, ReplicationActionType.ACTIVATE, pagePath, opts);
```

## ReplicationStatus Interface

Provides status information about replicated content.

**Official Documentation:** [ReplicationStatus JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationStatus.html)

### Key Methods:

```java
// Check if content is activated
boolean isActivated();

// Get last published timestamp
Calendar getLastPublished();

// Get user who last published
String getLastPublishedBy();

// Get last replication action
String getLastReplicationAction();

// Get last replicated revision
String getLastReplicatedRevision();
```

**Example:**
```java
public Map<String, Object> getReplicationMetadata(ResourceResolver resolver, String path) {
    Session session = resolver.adaptTo(Session.class);
    ReplicationStatus status = replicator.getReplicationStatus(session, path);
    
    Map<String, Object> metadata = new HashMap<>();
    if (status != null) {
        metadata.put("activated", status.isActivated());
        metadata.put("lastPublished", status.getLastPublished());
        metadata.put("lastPublishedBy", status.getLastPublishedBy());
        metadata.put("lastAction", status.getLastReplicationAction());
    }
    return metadata;
}
```

## Complete Implementation Examples

### Example 1: OSGi Service with Replication

```java
package com.mycompany.aem.core.services.impl;

import com.day.cq.replication.Replicator;
import com.day.cq.replication.ReplicationActionType;
import com.day.cq.replication.ReplicationException;
import com.day.cq.replication.ReplicationOptions;
import org.apache.sling.api.resource.ResourceResolver;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.jcr.Session;

@Component(service = ContentPublisher.class, immediate = true)
public class ContentPublisherImpl implements ContentPublisher {
    
    private static final Logger LOG = LoggerFactory.getLogger(ContentPublisherImpl.class);
    
    @Reference
    private Replicator replicator;
    
    @Override
    public boolean publishPage(ResourceResolver resolver, String pagePath) {
        try {
            Session session = resolver.adaptTo(Session.class);
            
            // Create options for synchronous replication
            ReplicationOptions opts = new ReplicationOptions();
            opts.setSynchronous(true);
            opts.setSuppressVersions(false); // Keep version history
            
            // Activate the page
            replicator.replicate(session, ReplicationActionType.ACTIVATE, pagePath, opts);
            
            LOG.info("Successfully activated page: {}", pagePath);
            return true;
            
        } catch (ReplicationException e) {
            LOG.error("Failed to activate page: {}", pagePath, e);
            return false;
        }
    }
    
    @Override
    public boolean unpublishPage(ResourceResolver resolver, String pagePath) {
        try {
            Session session = resolver.adaptTo(Session.class);
            
            ReplicationOptions opts = new ReplicationOptions();
            opts.setSynchronous(true);
            
            // Deactivate the page
            replicator.replicate(session, ReplicationActionType.DEACTIVATE, pagePath, opts);
            
            LOG.info("Successfully deactivated page: {}", pagePath);
            return true;
            
        } catch (ReplicationException e) {
            LOG.error("Failed to deactivate page: {}", pagePath, e);
            return false;
        }
    }
}
```

### Example 2: Servlet with Replication

```java
package com.mycompany.aem.core.servlets;

import com.day.cq.replication.Replicator;
import com.day.cq.replication.ReplicationActionType;
import com.day.cq.replication.ReplicationException;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

import javax.jcr.Session;
import javax.servlet.Servlet;
import java.io.IOException;

@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.methods=POST",
        "sling.servlet.paths=/bin/myapp/publish"
    }
)
public class PublishServlet extends SlingAllMethodsServlet {
    
    @Reference
    private Replicator replicator;
    
    @Override
    protected void doPost(SlingHttpServletRequest request, 
                         SlingHttpServletResponse response) throws IOException {
        
        String path = request.getParameter("path");
        String action = request.getParameter("action"); // activate or deactivate
        
        if (path == null || action == null) {
            response.setStatus(400);
            response.getWriter().write("{\"error\": \"Missing parameters\"}");
            return;
        }
        
        try {
            Session session = request.getResourceResolver().adaptTo(Session.class);
            ReplicationActionType type = "activate".equals(action) 
                ? ReplicationActionType.ACTIVATE 
                : ReplicationActionType.DEACTIVATE;
            
            replicator.replicate(session, type, path);
            
            response.setContentType("application/json");
            response.getWriter().write("{\"success\": true, \"path\": \"" + path + "\"}");
            
        } catch (ReplicationException e) {
            response.setStatus(500);
            response.getWriter().write("{\"error\": \"" + e.getMessage() + "\"}");
        }
    }
}
```

### Example 3: Workflow Process Step with Replication

```java
package com.mycompany.aem.core.workflows;

import com.adobe.granite.workflow.WorkflowException;
import com.adobe.granite.workflow.WorkflowSession;
import com.adobe.granite.workflow.exec.WorkItem;
import com.adobe.granite.workflow.exec.WorkflowProcess;
import com.adobe.granite.workflow.metadata.MetaDataMap;
import com.day.cq.replication.Replicator;
import com.day.cq.replication.ReplicationActionType;
import com.day.cq.replication.ReplicationException;
import com.day.cq.replication.ReplicationOptions;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.jcr.Session;

@Component(
    service = WorkflowProcess.class,
    property = {
        "process.label=Custom Replication Process"
    }
)
public class CustomReplicationProcess implements WorkflowProcess {
    
    private static final Logger LOG = LoggerFactory.getLogger(CustomReplicationProcess.class);
    
    @Reference
    private Replicator replicator;
    
    @Override
    public void execute(WorkItem workItem, WorkflowSession workflowSession, 
                       MetaDataMap metaDataMap) throws WorkflowException {
        
        String payloadPath = workItem.getWorkflowData().getPayload().toString();
        Session session = workflowSession.adaptTo(Session.class);
        
        try {
            // Custom replication logic
            ReplicationOptions opts = new ReplicationOptions();
            opts.setSynchronous(true);
            
            // Get config from process arguments
            String processArgs = metaDataMap.get("PROCESS_ARGS", "");
            if ("activate".equals(processArgs)) {
                replicator.replicate(session, ReplicationActionType.ACTIVATE, payloadPath, opts);
                LOG.info("Workflow activated: {}", payloadPath);
            }
            
        } catch (ReplicationException e) {
            LOG.error("Replication failed in workflow", e);
            throw new WorkflowException("Replication failed", e);
        }
    }
}
```

### Example 4: Bulk Replication with AgentFilter

```java
import com.day.cq.replication.Agent;
import com.day.cq.replication.AgentFilter;
import com.day.cq.replication.ReplicationOptions;

public void bulkActivateToSpecificAgent(ResourceResolver resolver, 
                                        List<String> paths, 
                                        String targetAgentId) 
    throws ReplicationException {
    
    Session session = resolver.adaptTo(Session.class);
    
    // Configure options with agent filter
    ReplicationOptions opts = new ReplicationOptions();
    opts.setSynchronous(false); // Async for performance
    opts.setFilter(new AgentFilter() {
        @Override
        public boolean isIncluded(Agent agent) {
            return agent.getId().equals(targetAgentId);
        }
    });
    
    // Convert List to array
    String[] pathArray = paths.toArray(new String[0]);
    
    // Replicate all paths to specific agent
    replicator.replicate(session, ReplicationActionType.ACTIVATE, pathArray, opts);
    
    LOG.info("Bulk activated {} paths to agent: {}", paths.size(), targetAgentId);
}
```

## Additional Public APIs

Beyond the core Replicator interface, AEM 6.5 LTS provides additional public APIs for advanced replication management. These are all documented in the [official package summary](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/package-summary.html).

### AgentManager Interface

Manages replication agents programmatically.

**Official Documentation:** [AgentManager JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/AgentManager.html)

**Key Method:**

```java
Map<String, Agent> getAgents()
```

Returns a map of all available agents (agent ID → Agent instance).

**Example:**

```java
import com.day.cq.replication.AgentManager;
import com.day.cq.replication.Agent;
import org.osgi.service.component.annotations.Reference;

@Reference
private AgentManager agentManager;

public void listAllAgents() {
    Map<String, Agent> agents = agentManager.getAgents();
    
    for (Map.Entry<String, Agent> entry : agents.entrySet()) {
        String agentId = entry.getKey();
        Agent agent = entry.getValue();
        
        LOG.info("Agent ID: {}", agentId);
        LOG.info("  Enabled: {}", agent.isEnabled());
        LOG.info("  Transport URI: {}", agent.getConfiguration().getTransportURI());
        LOG.info("  Serialization Type: {}", agent.getConfiguration().getSerializationType());
    }
}

public Agent getAgentById(String agentId) {
    Map<String, Agent> agents = agentManager.getAgents();
    return agents.get(agentId);
}

public boolean isAgentEnabled(String agentId) {
    Agent agent = agentManager.getAgents().get(agentId);
    return agent != null && agent.isEnabled();
}
```

### ReplicationQueue Interface

Manages replication queue operations programmatically.

**Official Documentation:** [ReplicationQueue JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationQueue.html)

**Key Methods:**

```java
// Get queue name
String getName()

// Get all queue entries
List<ReplicationQueue.Entry> entries()

// Get entries for specific path
List<ReplicationQueue.Entry> entries(String path)

// Get specific entry
ReplicationQueue.Entry getEntry(String path, Calendar after)

// Clear entire queue
void clear()

// Clear specific entries by ID
void clear(Set<String> ids)

// Get queue status
ReplicationQueue.Status getStatus()

// Check if paused
boolean isPaused()

// Pause/resume queue
void setPaused(boolean paused)

// Force retry on blocked entry
void forceRetry()
```

**Example:**

```java
import com.day.cq.replication.Agent;
import com.day.cq.replication.ReplicationQueue;
import com.day.cq.replication.AgentManager;

@Reference
private AgentManager agentManager;

public void monitorQueue(String agentId) {
    Agent agent = agentManager.getAgents().get(agentId);
    if (agent == null) {
        LOG.error("Agent not found: {}", agentId);
        return;
    }
    
    ReplicationQueue queue = agent.getQueue();
    
    // Get queue status
    ReplicationQueue.Status status = queue.getStatus();
    LOG.info("Queue: {}", queue.getName());
    LOG.info("Status: {}", status);
    LOG.info("Is Paused: {}", queue.isPaused());
    
    // List all entries
    List<ReplicationQueue.Entry> entries = queue.entries();
    LOG.info("Queue size: {}", entries.size());
    
    for (ReplicationQueue.Entry entry : entries) {
        LOG.info("  Path: {}", entry.getPath());
        LOG.info("  Time: {}", entry.getTime());
        LOG.info("  ID: {}", entry.getId());
    }
}

public void clearQueueForPath(String agentId, String path) {
    Agent agent = agentManager.getAgents().get(agentId);
    if (agent == null) return;
    
    ReplicationQueue queue = agent.getQueue();
    
    // Get entries for specific path
    List<ReplicationQueue.Entry> entries = queue.entries(path);
    
    if (entries.isEmpty()) {
        LOG.info("No queue entries for path: {}", path);
        return;
    }
    
    // Collect entry IDs
    Set<String> ids = new HashSet<>();
    for (ReplicationQueue.Entry entry : entries) {
        ids.add(entry.getId());
    }
    
    // Clear specific entries
    queue.clear(ids);
    LOG.info("Cleared {} entries for path: {}", ids.size(), path);
}

public void pauseQueue(String agentId) {
    Agent agent = agentManager.getAgents().get(agentId);
    if (agent != null) {
        ReplicationQueue queue = agent.getQueue();
        queue.setPaused(true);
        LOG.info("Paused queue: {}", queue.getName());
    }
}

public void forceRetryBlockedQueue(String agentId) {
    Agent agent = agentManager.getAgents().get(agentId);
    if (agent != null) {
        ReplicationQueue queue = agent.getQueue();
        queue.forceRetry();
        LOG.info("Forced retry on queue: {}", queue.getName());
    }
}
```

### ReplicationListener Interface

Listens for replication events in real-time.

**Official Documentation:** [ReplicationListener JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationListener.html)

**Methods:**

```java
// Called before replication starts
void onStart(Agent agent, ReplicationAction action)

// Called when log message written
void onMessage(ReplicationLog.Level level, String message)

// Called when replication completes
void onEnd(Agent agent, ReplicationAction action, ReplicationResult result)

// Called on replication error
void onError(Agent agent, ReplicationAction action, Exception exception)
```

**Example:**

```java
import com.day.cq.replication.ReplicationListener;
import com.day.cq.replication.Agent;
import com.day.cq.replication.ReplicationAction;
import com.day.cq.replication.ReplicationResult;
import com.day.cq.replication.ReplicationLog;

public class CustomReplicationListener implements ReplicationListener {
    
    private static final Logger LOG = LoggerFactory.getLogger(CustomReplicationListener.class);
    
    @Override
    public void onStart(Agent agent, ReplicationAction action) {
        LOG.info("Replication starting - Agent: {}, Path: {}, Type: {}", 
            agent.getId(), 
            action.getPath(), 
            action.getType());
    }
    
    @Override
    public void onMessage(ReplicationLog.Level level, String message) {
        switch (level) {
            case DEBUG:
                LOG.debug("Replication: {}", message);
                break;
            case INFO:
                LOG.info("Replication: {}", message);
                break;
            case WARN:
                LOG.warn("Replication: {}", message);
                break;
            case ERROR:
                LOG.error("Replication: {}", message);
                break;
        }
    }
    
    @Override
    public void onEnd(Agent agent, ReplicationAction action, ReplicationResult result) {
        LOG.info("Replication completed - Agent: {}, Path: {}, Success: {}", 
            agent.getId(), 
            action.getPath(), 
            result.isSuccess());
            
        // Send notification, update metrics, etc.
        if (result.isSuccess()) {
            // Success handling
            notifySuccess(action.getPath());
        }
    }
    
    @Override
    public void onError(Agent agent, ReplicationAction action, Exception exception) {
        LOG.error("Replication failed - Agent: {}, Path: {}", 
            agent.getId(), 
            action.getPath(), 
            exception);
            
        // Error handling, alerts, retry logic
        alertReplicationFailure(action.getPath(), exception);
    }
    
    private void notifySuccess(String path) {
        // Custom notification logic
    }
    
    private void alertReplicationFailure(String path, Exception e) {
        // Custom alert logic
    }
}

// Using listener with Replicator
@Reference
private Replicator replicator;

public void replicateWithListener(ResourceResolver resolver, String path) 
    throws ReplicationException {
    
    Session session = resolver.adaptTo(Session.class);
    
    ReplicationOptions opts = new ReplicationOptions();
    opts.setSynchronous(true);
    
    // Attach custom listener
    CustomReplicationListener listener = new CustomReplicationListener();
    opts.setListener(listener);
    
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path, opts);
}
```

### AgentFilter Interface

Filters agents for selective replication (detailed earlier).

**Official Documentation:** [AgentFilter JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/AgentFilter.html)

**Method:**

```java
boolean isIncluded(Agent agent)
```

**Static Fields:**
- `AgentFilter.DEFAULT` - Filter for non-specific agents
- `AgentFilter.OUTBOX_AGENT_FILTER` - Filter for distribution operations

**Advanced Example:**

```java
import com.day.cq.replication.AgentFilter;
import com.day.cq.replication.Agent;

// Filter by multiple criteria
public class CustomAgentFilter implements AgentFilter {
    
    private final Set<String> allowedAgentIds;
    private final String requiredTransportProtocol;
    
    public CustomAgentFilter(Set<String> allowedAgentIds, String protocol) {
        this.allowedAgentIds = allowedAgentIds;
        this.requiredTransportProtocol = protocol;
    }
    
    @Override
    public boolean isIncluded(Agent agent) {
        // Filter by agent ID
        if (!allowedAgentIds.contains(agent.getId())) {
            return false;
        }
        
        // Filter by enabled status
        if (!agent.isEnabled()) {
            return false;
        }
        
        // Filter by transport protocol
        String transportURI = agent.getConfiguration().getTransportURI();
        if (transportURI != null && !transportURI.startsWith(requiredTransportProtocol)) {
            return false;
        }
        
        return true;
    }
}

// Usage
Set<String> allowedAgents = new HashSet<>(Arrays.asList("publish1", "publish2"));
AgentFilter filter = new CustomAgentFilter(allowedAgents, "https://");

ReplicationOptions opts = new ReplicationOptions();
opts.setFilter(filter);

replicator.replicate(session, ReplicationActionType.ACTIVATE, path, opts);
```

### Agent Interface

Represents a replication agent.

**Official Documentation:** [Agent JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/Agent.html)

**Key Methods:**

```java
String getId()                    // Get agent ID
boolean isEnabled()               // Check if enabled
AgentConfig getConfiguration()    // Get configuration
ReplicationQueue getQueue()       // Get agent's queue
boolean isConnected()             // Check connectivity status
```

**Example:**

```java
public void inspectAgent(String agentId) {
    Agent agent = agentManager.getAgents().get(agentId);
    
    if (agent == null) {
        LOG.error("Agent not found: {}", agentId);
        return;
    }
    
    LOG.info("Agent Details:");
    LOG.info("  ID: {}", agent.getId());
    LOG.info("  Enabled: {}", agent.isEnabled());
    LOG.info("  Connected: {}", agent.isConnected());
    
    AgentConfig config = agent.getConfiguration();
    LOG.info("  Transport URI: {}", config.getTransportURI());
    LOG.info("  Serialization: {}", config.getSerializationType());
    LOG.info("  User: {}", config.getTransportUser());
    
    ReplicationQueue queue = agent.getQueue();
    LOG.info("  Queue Name: {}", queue.getName());
    LOG.info("  Queue Size: {}", queue.entries().size());
    LOG.info("  Queue Status: {}", queue.getStatus());
}
```

## Exception Handling

### ReplicationException Hierarchy

```java
try {
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
    
} catch (ReplicationException e) {
    // Base exception for all replication errors
    LOG.error("Replication failed", e);
    
    // Common causes:
    // - Agent not found
    // - Agent disabled or blocked
    // - Network connectivity issues
    // - Permission denied
    // - Target instance unavailable
}
```

### Best Practices for Exception Handling:

```java
public ReplicationResult safeReplicate(ResourceResolver resolver, String path) {
    try {
        // Check permissions first
        Session session = resolver.adaptTo(Session.class);
        replicator.checkPermission(session, ReplicationActionType.ACTIVATE, path);
        
        // Perform replication
        ReplicationOptions opts = new ReplicationOptions();
        opts.setSynchronous(true);
        replicator.replicate(session, ReplicationActionType.ACTIVATE, path, opts);
        
        return ReplicationResult.success(path);
        
    } catch (ReplicationException e) {
        LOG.error("Replication failed for path: {}", path, e);
        return ReplicationResult.failure(path, e.getMessage());
    }
}
```

## Security Considerations

### 1. Use Service Users

Never use admin sessions. Create dedicated service users:

```xml
<!-- Service user mapping in ui.apps/src/main/content/META-INF/sling/nodetypes.cnd -->
<serviceUserMapping>
    <entry>
        bundle="com.mycompany.aem.core"
        subServiceName="replication-service"
        user="replication-service"
    </entry>
</serviceUserMapping>
```

```java
import org.apache.sling.api.resource.ResourceResolverFactory;
import org.apache.sling.api.resource.ResourceResolver;

@Reference
private ResourceResolverFactory resolverFactory;

public void replicateWithServiceUser(String path) throws ReplicationException {
    Map<String, Object> authInfo = new HashMap<>();
    authInfo.put(ResourceResolverFactory.SUBSERVICE, "replication-service");
    
    try (ResourceResolver resolver = resolverFactory.getServiceResourceResolver(authInfo)) {
        Session session = resolver.adaptTo(Session.class);
        replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
    } catch (LoginException e) {
        LOG.error("Service user login failed", e);
    }
}
```

### 2. Permission Checks

Always verify permissions before replication:

```java
public boolean activateIfAllowed(ResourceResolver resolver, String path) {
    Session session = resolver.adaptTo(Session.class);
    
    try {
        // Check permission first
        replicator.checkPermission(session, ReplicationActionType.ACTIVATE, path);
        
        // Permission granted, proceed
        replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
        return true;
        
    } catch (ReplicationException e) {
        LOG.warn("User lacks replication permission for: {}", path);
        return false;
    }
}
```

### 3. Input Validation

Validate paths and parameters:

```java
public void safeActivate(ResourceResolver resolver, String path) 
    throws IllegalArgumentException, ReplicationException {
    
    // Validate path
    if (path == null || path.trim().isEmpty()) {
        throw new IllegalArgumentException("Path cannot be null or empty");
    }
    
    // Ensure path is under /content
    if (!path.startsWith("/content/")) {
        throw new IllegalArgumentException("Invalid path: must be under /content");
    }
    
    // Check resource exists
    Resource resource = resolver.getResource(path);
    if (resource == null) {
        throw new IllegalArgumentException("Resource not found: " + path);
    }
    
    // Safe to replicate
    Session session = resolver.adaptTo(Session.class);
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
}
```

## Performance Optimization

### 1. Use Asynchronous Replication for Bulk Operations

```java
public void bulkActivate(ResourceResolver resolver, List<String> paths) 
    throws ReplicationException {
    
    Session session = resolver.adaptTo(Session.class);
    
    ReplicationOptions opts = new ReplicationOptions();
    opts.setSynchronous(false); // Non-blocking for better performance
    opts.setSuppressVersions(true); // Skip versioning to reduce overhead
    
    String[] pathArray = paths.toArray(new String[0]);
    replicator.replicate(session, ReplicationActionType.ACTIVATE, pathArray, opts);
}
```

### 2. Batch Multiple Paths

Use array variants instead of looping:

```java
// BAD: Individual calls in loop
for (String path : paths) {
    replicator.replicate(session, ReplicationActionType.ACTIVATE, path);
}

// GOOD: Single batch call
String[] pathArray = paths.toArray(new String[0]);
replicator.replicate(session, ReplicationActionType.ACTIVATE, pathArray, opts);
```

### 3. Suppress Unnecessary Operations

```java
ReplicationOptions opts = new ReplicationOptions();
opts.setSuppressVersions(true); // Don't create versions
opts.setSuppressStatusUpdate(false); // Keep status tracking
```

## Testing and Validation

### Unit Testing with Mocks

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class ContentPublisherTest {
    
    @Mock
    private Replicator replicator;
    
    @Mock
    private ResourceResolver resolver;
    
    @Mock
    private Session session;
    
    @InjectMocks
    private ContentPublisherImpl contentPublisher;
    
    @Test
    void testPublishPage() throws Exception {
        when(resolver.adaptTo(Session.class)).thenReturn(session);
        
        String path = "/content/mysite/en/page";
        boolean result = contentPublisher.publishPage(resolver, path);
        
        assertTrue(result);
        verify(replicator).replicate(
            eq(session),
            eq(ReplicationActionType.ACTIVATE),
            eq(path),
            any(ReplicationOptions.class)
        );
    }
}
```

## Troubleshooting

### Common Issues:

**Issue: ReplicationException - Agent not found**
```
Solution: Verify replication agents are configured and enabled
Check: /etc/replication/agents.author
```

**Issue: Permission denied**
```
Solution: Grant replication privileges to service user
Check: User permissions on content path and crx:replicate privilege
```

**Issue: Synchronous replication times out**
```
Solution: 
1. Use asynchronous replication for large operations
2. Increase timeout in replication agent configuration
3. Check Publish instance performance
```

**Issue: Content not appearing on Publish**
```
Solution:
1. Check replication queue status
2. Verify Dispatcher Flush agent is working
3. Check Publish instance logs
4. Use getReplicationStatus() to verify activation
```

## Related Skills

- `configure-replication-agent`: Set up replication agents
- `replicate-content`: UI-based content activation methods
- `troubleshoot-replication`: Diagnose and fix replication issues

## Success Criteria

- ✓ Successfully integrated Replicator service via OSGi reference
- ✓ Programmatic activation/deactivation working
- ✓ ReplicationOptions configured correctly
- ✓ Permission checks implemented
- ✓ Exception handling in place
- ✓ Service user configured with proper permissions
- ✓ Unit tests passing with mocked Replicator
- ✓ Replication status can be queried
- ✓ Content appears on Publish after API call
- ✓ Logs show successful replication entries

## Additional Resources

- [Replication API JavaDoc (Complete Package Summary)](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/package-summary.html)
- [Replicator Interface JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/Replicator.html)
- [ReplicationOptions JavaDoc](https://developer.adobe.com/experience-manager/reference-materials/6-5-lts/javadoc/com/day/cq/replication/ReplicationOptions.html)
- [Official AEM 6.5 LTS Replication Documentation](https://experienceleague.adobe.com/en/docs/experience-manager-65-lts/content/implementing/deploying/configuring/replication)
- [AEM 6.5 LTS Replication Troubleshooting Guide](https://experienceleague.adobe.com/en/docs/experience-manager-65-lts/content/implementing/deploying/configuring/troubleshoot-rep)
- [AEM 6.5 LTS Documentation Hub](https://experienceleague.adobe.com/en/docs/experience-manager-65-lts)
- AEM Developer Documentation for OSGi services
- ACS AEM Commons Replication Examples: [GitHub](https://github.com/Adobe-Consulting-Services/acs-aem-commons)
