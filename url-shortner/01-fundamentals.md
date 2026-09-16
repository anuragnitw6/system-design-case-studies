# URL Shortener — Step 1: Fundamentals

## 1. What is a URL Shortener?

A URL shortener converts a long URL into a short, easy-to-share URL.

Example:


Long URL:
https://example.com/products/category/mobile/iphone-17?campaign=instagram

Short URL:
https://sho.rt/aB72x


When a user opens the short URL, the system looks up the original URL and redirects the user to it.



## 2. Basic System Flow


User
 |
 | Create short URL
 v
API Server
 |
 v
Database
 |
 v
Short URL returned to user


For a redirect:


User
 |
 | GET /aB72x
 v
API Server
 |
 v
Database
 |
 v
Original URL
 |
 v
HTTP Redirect



## 3. Core Components

### Client

The client can be a browser, mobile application, or another service.

It sends requests such as:

http
POST /api/v1/urls

and:

http
GET /aB72x


### API Server

The API server contains the application logic.

It:

- receives requests
- validates URLs
- generates short codes
- stores URL mappings
- retrieves original URLs
- returns redirects

### Database

The database provides durable storage for the mapping:


short_code -> original_url


Example:

aB72x -> https://example.com/products/iphone

## 4. Stateless API Servers

API servers should ideally be stateless.

That means an API server does not depend on important persistent information stored only in its own memory.

Instead, shared state is kept in systems such as:

- Database
- Redis
- Object storage
- Message queues

This allows multiple API servers to handle requests interchangeably.


                Load Balancer
                 /    |                   
             API-1 API-2 API-3
                \     |     /
                 \    |    /
                   Redis
                     |
                  Database

If API-1 handles one request and API-3 handles the next request, API-3 can still process it because the required state is stored outside API-1.


## 5. Load Balancer

A load balancer receives incoming traffic and distributes requests across available API servers.

Example:


                 Users
                   |
                   v
             Load Balancer
              /     |                  
           API-1  API-2  API-3


If API-2 becomes unhealthy, the load balancer can stop sending traffic to it.


API-1 -> healthy
API-2 -> unhealthy
API-3 -> healthy


This provides better availability and allows horizontal scaling.

## 6. Redis / Cache

Redis can be used as a fast cache in front of the database.

For a popular short URL:

aB72x -> https://example.com/products/iphone

the mapping can be stored in Redis.

Redirect flow:


User
 |
 v
API Server
 |
 v
Redis
 |
 +-- Cache HIT --> Original URL --> Redirect
 |
 +-- Cache MISS --> Database
                       |
                       v
                    Redis
                       |
                       v
                    Redirect


This is especially important because URL shorteners are generally read-heavy.



## 7. Important Mental Model

Each component has a different responsibility:


Load Balancer
    |
    +--> "Which API server should handle this request?"

API Server
    |
    +--> "What should I do with this request?"

Redis
    |
    +--> "Do I already have this frequently accessed data?"

Database
    |
    +--> "What is the durable source of truth?"




## 8. Viral / Hot URL Example

Suppose one short URL receives 10,000 requests.


https://sho.rt/aB72x
          |
          +---- 10,000 requests
                    |
                    v
              Load Balancer
                    |
             API Servers
                    |
                    v
                  Redis
                    |
                    v
          aB72x -> original URL


If the URL is cached, the system does not need to query the database for every request.

This is called a **hot key** when one cache key receives a disproportionately large amount of traffic.



## 9. Key Concepts Learned

- URL shortener
- API server
- Stateless architecture
- Load balancer
- Horizontal scaling
- Redis
- Cache hit / cache miss
- Database as source of truth
- Read-heavy workloads
- Hot keys



## Next Step

**Step 2 — Requirements**

We will define:

- Functional requirements
- Non-functional requirements
- API requirements
- Scale assumptions
- What is inside and outside the system
