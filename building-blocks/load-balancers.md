# Load Balancers

*Last Updated: December 2024*

## 📋 What is Load Balancing?

Load balancing distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed.

## 🎯 Why It Matters

- **Scalability**: Handle more users by adding servers
- **High Availability**: If one server fails, others continue
- **Performance**: Distribute load = faster response times

## 🏗️ Types of Load Balancers

### 1. Hardware Load Balancers
- Physical devices (expensive)
- Used by large enterprises
- Examples: F5, Citrix

### 2. Software Load Balancers  
- More flexible and cost-effective
- Examples: Nginx, HAProxy, AWS ELB

### 3. DNS Load Balancing
- Distribute traffic at DNS level
- Simple but less flexible

## ⚖️ Load Balancing Algorithms

### Round Robin
Server 1 → Server 2 → Server 3 → Server 1 → ...
- Simplest approach
- Equal distribution
- Doesn't consider server load

### Least Connections
Route to server with fewest active connections
- Better for varying request times
- More intelligent than round robin

### Weighted Round Robin
Server 1 (weight: 5) gets 5x more requests than Server 2 (weight: 1)
- Useful when servers have different capacities

## 💻 Simple Implementation
```python
class LoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.current = 0
    
    def get_next_server(self):
        """Round-robin algorithm"""
        server = self.servers[self.current]
        self.current = (self.current + 1) % len(self.servers)
        return server

# Usage
lb = LoadBalancer(['server1.com', 'server2.com', 'server3.com'])
print(lb.get_next_server())  # server1.com
print(lb.get_next_server())  # server2.com
```

## 🏢 Real-World Examples

**Netflix**: Uses AWS Elastic Load Balancing to distribute millions of requests across thousands of servers.

**Uber**: Multiple load balancing layers to handle ride requests globally.

## 🤔 Interview Questions

1. **Q: What happens if a server fails during load balancing?**
   A: Health checks detect failed servers and stop routing traffic to them.

2. **Q: How do you handle sticky sessions?**
   A: Use consistent hashing or session affinity to route same user to same server.

3. **Q: What's the difference between Layer 4 vs Layer 7 load balancing?**
   A: Layer 4 (transport) routes based on IP/port. Layer 7 (application) routes based on HTTP headers/content.

## 📚 Further Reading

- [AWS ELB Documentation](https://aws.amazon.com/elasticloadbalancing/)
- [Nginx Load Balancing Guide](https://nginx.org/en/docs/http/load_balancing.html)
