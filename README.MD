# Handmade CDN

> Your goal is to create your own cdn for delivering millions of images across the globe.

## Set up 7 containers - bind server, load balancer 1, load balancer 2, node1, node2, node3, node 4. 

```bash
services:
  bind-server:
    image: internetsystemsconsortium/bind9:9.18
    container_name: bind-server
    ports:
      - "1053:53/tcp"
      - "1053:53/udp"
    volumes:
      - ./bind/named.conf:/etc/bind/named.conf
      - ./bind/db.cdn.test:/etc/bind/db.cdn.test
      - ./bind/db.127:/etc/bind/db.127
    environment:
      - BIND9_LOG_LEVEL=info
    networks:
      - cdn-network
    entrypoint: [ "/usr/sbin/named", "-f", "-c", "/etc/bind/named.conf" ]

  load-balancer-1:
    image: nginx:latest
    container_name: load-balancer-1
    ports:
      - "8081:80" 
    volumes:
      - ./lb1/default.conf:/etc/nginx/nginx.conf 
    depends_on:
      - bind-server
      - node1
      - node2
    networks:
      - cdn-network

  load-balancer-2:
    image: nginx:latest
    container_name: load-balancer-2
    ports:
      - "8082:80"  
    volumes:
      - ./lb2/default.conf:/etc/nginx/nginx.conf  
    depends_on:
      - bind-server
      - node3
      - node4
    networks:
      - cdn-network

  node1:
    build:
      context: ./nodes/node1
    container_name: node1
    ports:
      - "8083:80"
    networks:
      - cdn-network

  node2:
    build:
      context: ./nodes/node2
    container_name: node2
    ports:
      - "8084:80"
    networks:
      - cdn-network

  node3:
    build:
      context: ./nodes/node3
    container_name: node3
    ports:
      - "8085:80"
    networks:
      - cdn-network

  node4:
    build:
      context: ./nodes/node4
    container_name: node4
    ports:
      - "8086:80"
    networks:
      - cdn-network

networks:
  cdn-network:
    driver: bridge
    
```

---

## Try to implement different balancing approaches. Implement efficient caching. 

### Approach 1: Round-Robin

```bash
upstream cdn_nodes {
    server node1:80;
    server node2:80;
}
```

### Approach 2: Weighted Round Robin

```bash
upstream cdn_nodes {
    server node3:80 weight=3;
    server node4:80 weight=1;
}
```

### Approach 3: Least Connections

More balanced, reduces overload. More overhead, complex to manage.

```bash
upstream cdn_nodes {
    least_conn;
    server node1:80;
    server node2:80;
}
```

### Approach 4: IP Hash

Ensures session persistence, useful for caching. Uneven load distribution, failure management.

```bash
upstream cdn_nodes {
    ip_hash;
    server node1:80;
    server node2:80;
}
```

---

## Write down pros and cons of each approach.

| **Load Balancing Method** | **Pros**                                                            | **Cons**                                                                |
|---------------------------|---------------------------------------------------------------------|-------------------------------------------------------------------------|
| **Round-Robin**           | - Simple to implement                                               | - Doesn't consider server load                                          |
|                           | - Distributes traffic equally in most cases                         | - Can overload slower or less capable servers                           |
|                           |                                                                     | - Not ideal for long-lived connections                                  |
| **Weighted Round-Robin**  | - Can prioritize more powerful servers                              | - Requires manual configuration and tuning                              |
|                           | - Still simple and predictable                                      | - Does not adapt to real-time server load changes                       |
|                           |                                                                     | - Less effective when traffic patterns are unpredictable                |
| **Least Connections**     | - Dynamically balances based on server load                         | - Requires tracking active connections on each server                   |
|                           | - Prevents overloading busy servers                                 | - Slightly higher overhead for maintaining the connection count         |
|                           | - Ideal for long-lived connections                                  | - Not suitable for low-latency, high-throughput applications            |
| **IP Hash**               | - Ensures session persistence (same client goes to the same server) | - Can lead to uneven load distribution if clients are not evenly spread |
|                           | - Useful for caching (client gets same cache server)                | - If a server fails, clients hashed to that server may encounter issues |
|                           | - Simple and efficient for session persistence                      | - Does not consider real-time server performance or capacity            |


