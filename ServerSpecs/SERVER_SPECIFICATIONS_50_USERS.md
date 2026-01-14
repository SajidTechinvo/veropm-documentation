# Server Specifications for 50 Concurrent Users

## Overview

This document provides server specification recommendations for hosting VeroPM with DocEngine (Document Generation Engine) features for **50 concurrent users**.

### Architecture Overview
- **Backend**: .NET Core API with DocEngine pipeline
- **Frontend**: React (static files)
- **Databases**: SQL Server, Neo4j
- **Cache**: Redis
- **AI**: Azure OpenAI (external API calls) / OpenRouter (we recommend OpenRouter to use better models)
- **Background Services**: AI chunk processing

---

## Recommended Server Configuration

### Option 1: Single Server (Small Scale)

For 50 users with moderate AI usage:

**Server Specs:**
- **CPU**: 8 cores (16 vCPU)
- **RAM**: 32 GB
- **Storage**: 500 GB SSD (OS + Apps + Logs)
- **Network**: 1 Gbps
- **OS**: Windows Server 2022

**Deployment:**
- Backend API: IIS
- Frontend: IIS static hosting
- SQL Server: Express/Standard (on same server)
- Neo4j: Community Edition (on same server)
- Redis: Standalone (on same server)

**Pros:**
- Simpler deployment
- Easier management

**Cons:**
- Resource contention during peak loads
- Limited scalability
- Single point of failure

---

### Option 2: Separated Services (Recommended)

Better performance and scalability:

#### 1. Application Server (Backend + Frontend)
- **CPU**: 4 cores (8 vCPU)
- **RAM**: 16 GB
- **Storage**: 200 GB SSD
- **Network**: 1 Gbps
- **Services**:
  - .NET Core API (Kestrel)
  - React static files (IIS)
  - Background services (DocEngine orchestrator)

#### 2. Database Server (SQL Server)
- **CPU**: 4 cores (8 vCPU)
- **RAM**: 16 GB
- **Storage**: 500 GB SSD (with auto-grow)
- **Network**: 1 Gbps
- **SQL Server**: Standard Edition (recommended) or Express

#### 3. Graph Database Server (Neo4j)
- **CPU**: 2 cores (4 vCPU)
- **RAM**: 8 GB
- **Storage**: 200 GB SSD
- **Network**: 1 Gbps
- **Neo4j**: Community Edition

#### 4. Cache Server (Redis)
- **CPU**: 2 cores (4 vCPU)
- **RAM**: 4 GB
- **Storage**: 50 GB SSD
- **Network**: 1 Gbps
- **Redis**: Standalone or Azure Cache for Redis

**Pros:**
- Better performance isolation
- Easier to scale individual components
- Better resource utilization
- Improved reliability

**Cons:**
- More complex deployment
- Requires network configuration

---

## Resource Allocation Breakdown

### For 50 Concurrent Users

**Backend API Server:**
- **CPU**: 4-8 cores
- **RAM**: 16 GB (8 GB for API, 4 GB for background services, 4 GB buffer)
- **Concurrent Requests**: ~100-200/sec
- **AI Processing**: 10 concurrent generations (from config)

**Database (SQL Server):**
- **CPU**: 4 cores
- **RAM**: 16 GB (8 GB for SQL Server, 8 GB for OS)
- **Storage**: 500 GB (with growth)
- **Connection Pool**: 50-100 connections

**Neo4j:**
- **CPU**: 2 cores
- **RAM**: 8 GB
- **Storage**: 200 GB
- **Purpose**: GraphRAG and traceability

**Redis:**
- **RAM**: 4 GB
- **Purpose**: Session cache, rate limiting, query caching

**Frontend:**
- Static hosting (IIS/CDN)
- Minimal resources needed

---

## Performance Considerations

### AI Workload (DocEngine)

From your configuration:
- **Max Concurrent Generations**: 10
- **Max Concurrent Chunks Per Session**: 4
- **Generation Timeout**: 120 minutes
- **Polling Interval**: 1 second

**Impact:**
- Each AI generation can take 2-5 minutes
- 10 concurrent = ~50-100 AI API calls/hour
- Azure OpenAI rate limits: 30 req/min, 80K tokens/min

**Recommendation:** Monitor Azure OpenAI usage and adjust `MaxConcurrentGenerations` based on your tier.

### Database Considerations

**SQL Server:**
- Connection pooling: Configure `Min Pool Size=5;Max Pool Size=100` (from appsettings.json)
- Index optimization for frequent queries
- Regular maintenance jobs (backup, index rebuild)

**Neo4j:**
- Graph queries can be memory-intensive
- Monitor query performance
- Consider indexing for frequently accessed nodes

**Redis:**
- Use for session storage and caching
- Configure appropriate TTL for cache entries
- Monitor memory usage

---

## Minimum Viable Specs

**Single Server:**
- **CPU**: 4 cores
- **RAM**: 16 GB
- **Storage**: 500 GB SSD

**Limitations:**
- All services on one server
- Limited scalability
- Potential performance bottlenecks during peak AI processing
- May need to reduce `MaxConcurrentGenerations` to 5-7

---

## Monitoring and Scaling Recommendations

### 1. Key Metrics to Monitor

- **CPU Usage**: Target <70% average
- **RAM Usage**: Target <80% average
- **Database Connections**: Monitor active connections
- **AI API Call Rates**: Track Azure OpenAI usage
- **Response Times**: API response times should be <2 seconds
- **Disk I/O**: Monitor read/write operations
- **Network**: Bandwidth utilization

### 2. Auto-Scaling Triggers

Consider scaling when:
- CPU > 75% for 5 minutes
- RAM > 85% for 5 minutes
- Response time > 2 seconds average
- Database connection pool > 80% utilized

### 3. Performance Tuning

**Backend API:**
- Configure Kestrel thread pool settings
- Optimize async/await patterns
- Implement response compression
- Use connection pooling for databases

**SQL Server:**
- Regular index maintenance
- Query optimization
- Configure appropriate max memory
- Enable query store for performance insights

**Neo4j:**
- Tune heap size based on available RAM
- Configure page cache appropriately
- Monitor query performance

**Redis:**
- Configure max memory policy (eviction strategy)
- Monitor memory usage
- Use appropriate data structures

---

## Required Software and Versions

### Application Server

#### Web Server
- **Internet Information Services (IIS)**: Version 10.0 or later
  - Required Windows features: IIS, ASP.NET Core Module, URL Rewrite Module

#### .NET Runtime
- **.NET Runtime**: Version 8.0 or later
  - Required for running .NET Core API
  - Download from: https://dotnet.microsoft.com/download

#### Additional Components
- **Visual C++ Redistributable**: Latest version (required by .NET Core)
- **Windows PowerShell**: Version 5.1 or later (included with Windows Server)

---

### Database Server

#### SQL Server
- **Microsoft SQL Server Express Edition**: 2022 or later
  - Alternative: SQL Server Standard Edition (recommended for production)
  - SQL Server Management Studio (SSMS): Version 19.0 or later
  - Download from: https://www.microsoft.com/sql-server/sql-server-downloads

#### SQL Server Tools
- **SQL Server Command Line Utilities**: Latest version
- **SQL Server Configuration Manager**: Included with SQL Server installation

---

### Graph Database Server

#### Neo4j
- **Neo4j Community Edition**: Version 5.x or later
  - Download from: https://neo4j.com/download/
  - Java Runtime Environment (JRE): Version 17 or later (required by Neo4j)
  - Download JRE from: https://www.oracle.com/java/technologies/downloads/

#### Neo4j Tools
- **Neo4j Browser**: Included with Neo4j installation
- **Neo4j Desktop** (optional): For easier management

---

### Cache Server

#### Redis
- **Redis for Windows**: Version 7.0 or later
  - Download from: https://github.com/microsoftarchive/redis/releases
  - Alternative: Memurai (Redis-compatible for Windows)
  - Download Memurai from: https://www.memurai.com/

---

### All Servers (Common Software)

#### Operating System
- **Windows Server**: 2022 Standard or Datacenter Edition
  - Latest service packs and security updates

#### Security & Monitoring
- **Windows Defender** or compatible antivirus solution
- **Windows Firewall**: Configured with appropriate rules
- **Windows Update**: Configured for automatic security updates

#### Utilities
- **7-Zip** or similar archive utility: Latest version
- **Notepad++** or similar text editor: Latest version (optional, for configuration editing)

---

### Software Installation Order

1. **Operating System**
   - Install Windows Server 2022
   - Apply all Windows Updates
   - Configure Windows Firewall

2. **Prerequisites**
   - Install Visual C++ Redistributable
   - Install Java Runtime Environment (JRE) 17+ (for Neo4j)

3. **Web Server**
   - Enable IIS and required features
   - Install .NET Runtime 8.0+

4. **Database Software**
   - Install SQL Server Express/Standard Edition
   - Install SQL Server Management Studio (SSMS)
   - Configure SQL Server authentication and ports

5. **Graph Database**
   - Install Neo4j Community Edition
   - Configure Neo4j settings and ports
   - Set up Neo4j service

6. **Cache Server**
   - Install Redis for Windows or Memurai
   - Configure Redis settings and ports
   - Set up Redis service

7. **Application Deployment**
   - Deploy .NET Core API to IIS
   - Deploy React frontend static files
   - Configure application settings

---

### Version Compatibility Notes

- **.NET 8.0** requires Windows Server 2016 or later
- **SQL Server 2022** requires Windows Server 2019 or later
- **Neo4j 5.x** requires Java 17 or later
- **Redis 7.0** for Windows is compatible with Windows Server 2016+

### Download Links Summary

- **.NET Runtime**: https://dotnet.microsoft.com/download
- **SQL Server**: https://www.microsoft.com/sql-server/sql-server-downloads
- **Neo4j**: https://neo4j.com/download/
- **Java JRE**: https://www.oracle.com/java/technologies/downloads/
- **Redis for Windows**: https://github.com/microsoftarchive/redis/releases
- **Memurai**: https://www.memurai.com/

---

## Final Recommendation

For **50 concurrent users** with DocEngine features, we recommend **Option 2 (Separated Services)**:

1. **Application Server**: 4 cores, 16 GB RAM
2. **SQL Server**: 4 cores, 16 GB RAM
3. **Neo4j**: 2 cores, 8 GB RAM
4. **Redis**: 2 cores, 4 GB RAM

This setup:
- ✅ Handles 50 concurrent users comfortably
- ✅ Supports DocEngine background processing
- ✅ Allows room for growth to 100-150 users
- ✅ Provides good performance isolation
- ✅ Better reliability and maintainability

### Alternative: Option 1 (Single Server)

Option 1 can work but:
- Monitor performance closely
- Be prepared to scale up quickly
- Consider reducing `MaxConcurrentGenerations` to 7-8
- Plan for migration to Option 2 when user base grows

---

## Deployment Checklist

### Pre-Deployment
- [ ] Choose server configuration (Option 1 or 2)
- [ ] Provision servers with required specs
- [ ] Configure network security (firewalls, ports)
- [ ] Set up SSL certificates
- [ ] Configure domain/DNS

### Application Deployment
- [ ] Deploy .NET Core API
- [ ] Deploy React frontend (static files)
- [ ] Configure appsettings.json for production
- [ ] Set up background services (DocEngine orchestrator)
- [ ] Configure logging and monitoring

### Database Setup
- [ ] Install and configure SQL Server
- [ ] Run database migrations
- [ ] Configure connection strings
- [ ] Set up backup strategy
- [ ] Install and configure Neo4j
- [ ] Configure Neo4j connection
- [ ] Install and configure Redis
- [ ] Test all database connections

### Monitoring Setup
- [ ] Configure application monitoring (Application Insights, etc.)
- [ ] Set up server monitoring (CPU, RAM, Disk)
- [ ] Configure alerting for critical metrics
- [ ] Set up log aggregation
- [ ] Configure Azure OpenAI usage monitoring

### Security
- [ ] Configure authentication/authorization
- [ ] Set up rate limiting
- [ ] Configure CORS policies
- [ ] Enable HTTPS only
- [ ] Set up firewall rules
- [ ] Configure database access controls

---

## Notes

- **Azure OpenAI**: External service. Monitor usage closely.
- **Growth Planning**: Plan for 2x growth (100 users) within 6-12 months.
- **Backup Strategy**: Implement daily backups for databases.
- **Disaster Recovery**: Consider backup server or cloud backup solution.
- **Testing**: Load test with 50 concurrent users before going live.

---

## Support

For questions or issues related to server configuration, refer to:
- Application logs: `logs/veropm-.log`
- Database connection issues: Check connection strings in `appsettings.json`
- AI processing issues: Check `AIBrain` configuration section
- Performance issues: Review monitoring metrics and adjust accordingly
