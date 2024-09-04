# Handmade CDN

> Your goal is to create your own cdn for delivering millions of images across the globe.

## Set up 7 containers - bind server, load balancer 1, load balancer 2, node1, node2, node3, node 4. 

```bash
version: '3.8'

services:
  bind:
    image: internetsystemsconsortium/bind9:9.16
    container_name: bind
    volumes:
      - ./bind/named.conf.local:/etc/bind/named.conf.local
      - ./bind/db.cdn.example.com:/etc/bind/db.cdn.example.com
    ports:
      - "53:53/udp"
      - "53:53/tcp"
    networks:
      - cdn_network

  lb1:
    image: nginx:latest
    container_name: lb1
    volumes:
      - ./lb1/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "8081:80"
    depends_on:
      - node1
      - node2
      - node3
      - node4
    networks:
      - cdn_network

  lb2:
    image: nginx:latest
    container_name: lb2
    volumes:
      - ./lb2/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "8082:80"
    depends_on:
      - node1
      - node2
      - node3
      - node4
    networks:
      - cdn_network

  node1:
    image: nginx:latest
    container_name: node1
    volumes:
      - ./nodes/node1/default.conf:/etc/nginx/conf.d/default.conf
      - ./nodes/node1/static:/usr/share/nginx/html
    networks:
      - cdn_network

  node2:
    image: nginx:latest
    container_name: node2
    volumes:
      - ./nodes/node2/default.conf:/etc/nginx/conf.d/default.conf
      - ./nodes/node2/static:/usr/share/nginx/html
    networks:
      - cdn_network

  node3:
    image: nginx:latest
    container_name: node3
    volumes:
      - ./nodes/node3/default.conf:/etc/nginx/conf.d/default.conf
      - ./nodes/node3/static:/usr/share/nginx/html
    networks:
      - cdn_network

  node4:
    image: nginx:latest
    container_name: node4
    volumes:
      - ./nodes/node4/default.conf:/etc/nginx/conf.d/default.conf
      - ./nodes/node4/static:/usr/share/nginx/html
    networks:
      - cdn_network

networks:
  cdn_network:
    driver: bridge

```

---

## Try to implement different balancing approaches. Implement efficient caching. 

### Approach 1: Round-Robin

### Approach 2: Least Connections

### Approach 3: IP Hash

### Approach 4: Geo-DNS + Load Balancer


---

## Write down pros and cons of each approach.

| Approach              | Pros                                            | Cons                                           | 
|-----------------------|-------------------------------------------------|------------------------------------------------| 
| **Round-Robin**       | Simple, easy to implement                       | Ignores server load, may overload some servers | 
| **Least Connections** | More balanced, reduces overload                 | More overhead, complex to manage               | 
| **IP Hashing**        | Ensures session persistence, useful for caching | Uneven load distribution, failure management   | 
| **Geo-DNS**           | Lower latency, optimized for global traffic     | More complex DNS setup                         |




