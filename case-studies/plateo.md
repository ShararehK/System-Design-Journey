# Case Study: Plateo - Restaurant Inventory Management

*A real-world system design case study from a project I'm building*

## 📋 Overview

Plateo is a SaaS platform for restaurant inventory management that I'm building as technical co-founder.

## 🎯 The Problem

- Restaurants lose 4-10% revenue to food waste
- Manual inventory tracking is time-consuming
- Multi-location chains lack centralized visibility
- Stockouts lead to lost sales

## 🏗️ System Architecture
┌─────────────┐
│   Clients   │ (React Web App)
└──────┬──────┘
│
↓
┌─────────────┐
│  API Layer  │ (Node.js + Express)
└──────┬──────┘
│
↓
┌─────────────┐
│  Database   │ (PostgreSQL)
└─────────────┘

## 💡 Key Design Decisions

### 1. Why PostgreSQL over NoSQL?

**Decision**: PostgreSQL  
**Reasoning**:
- Need ACID compliance (inventory counts must be accurate)
- Complex relationships (items, suppliers, orders, locations)
- Strong consistency requirements
- SQL querying for analytics/reporting

**Trade-off**: Less horizontal scalability than NoSQL, but consistency matters more here.

### 2. Real-Time Inventory Updates

**Challenge**: Multiple users updating same inventory simultaneously

**Solution**: 
- Optimistic locking with version numbers
- WebSocket connections for live updates
- Row-level locking for critical operations
```sql
UPDATE inventory 
SET quantity = quantity - 5, version = version + 1
WHERE item_id = 123 AND version = current_version;
```

### 3. Multi-Location Support

**Challenge**: Restaurant chains need centralized view + location-specific data

**Architecture**:
- Tenant-based data isolation (tenant_id in every table)
- Location hierarchy (chain → region → restaurant)
- Aggregated reporting across locations

## 📊 Scalability Considerations

### Current Scale
- Target: 100-500 restaurants
- ~1,000 inventory items per restaurant
- ~100 updates per restaurant per day

### Future Scale (if successful)
- 10,000+ restaurants
- Need to implement:
  - Database read replicas
  - Caching layer (Redis)
  - CDN for static assets
  - Horizontal API scaling

## 🚧 Challenges & Solutions

### Challenge 1: Concurrent Inventory Updates
**Problem**: Two chefs reducing same item quantity at once  
**Solution**: Database row-level locking + optimistic concurrency control

### Challenge 2: Accurate Low-Stock Alerts  
**Problem**: Don't want false alerts from temporary usage  
**Solution**: Threshold-based alerts + configurable buffer periods

### Challenge 3: Multi-Currency Support
**Problem**: International chains use different currencies  
**Solution**: Store amounts as integers (cents), currency as separate field

## 💰 Cost Considerations

- PostgreSQL on managed service: ~$50-200/month
- Node.js hosting: ~$25-100/month  
- Total: <$300/month for MVP

Scales with revenue - SaaS pricing covers infrastructure costs.

## 📈 Lessons Learned

1. **Start simple**: Didn't over-engineer early. No Redis/microservices yet.
2. **Consistency > Availability**: For inventory, accuracy matters more than speed
3. **Database design matters**: Spent time on schema design - saved headaches later

## 🔮 Future Improvements

- [ ] Add Redis caching layer
- [ ] Implement database read replicas
- [ ] Add full-text search (Elasticsearch)
- [ ] Build mobile app (React Native)

---

**Status**: Active development  
**Last Updated**: December 2024  
**Tech Stack**: React, Node.js, PostgreSQL, AWS
