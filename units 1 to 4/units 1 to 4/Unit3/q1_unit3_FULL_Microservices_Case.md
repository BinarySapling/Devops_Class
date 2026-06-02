# FULL DETAILED SOLUTION: Migrating from Monolithic to Microservices

## 1. Understanding the case in simple words

### Given situation
A startup has one big e-commerce application running on a single server. That means:
- frontend is on the same machine
- backend is on the same machine
- database is on the same machine

This is called a **monolithic setup**.

### Problem in this setup
When sales traffic increases:
- the whole application becomes slow
- one overloaded part affects all other parts
- scaling is difficult
- maintenance becomes harder
- updates are risky because changing one part can affect the whole system

### What students are asked to do
Students must redesign this as a **microservices-based containerized application** where:
- frontend runs in its own container
- backend runs in its own container
- database runs in its own container
- all services are connected using Docker Compose

---

## 2. Monolithic vs Microservices

### Monolithic architecture
- tightly coupled system
- single deployment unit
- difficult scaling and debugging

### Microservices architecture
- independent services
- better scalability
- easier maintenance
- isolated failures

---

## 3. Architecture Design

Flow:
User → Frontend → Backend → Database

### Explanation
- User interacts with frontend
- Frontend sends API request to backend
- Backend processes logic
- Backend interacts with database
- Response flows back to user

---

## 4. Project Structure

```
ecommerce-microservices/
│
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   └── index.html
└── backend/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

---

## 5. Frontend Implementation

### index.html
```html
<!DOCTYPE html>
<html>
<body>
<h1>Frontend</h1>
<button onclick="load()">Call Backend</button>
<p id="res"></p>
<script>
async function load(){
 let r = await fetch('http://localhost:5000/');
 let t = await r.text();
 document.getElementById('res').innerText=t;
}
</script>
</body>
</html>
```

### Explanation
- Button triggers API call
- Fetch connects to backend
- Displays response

---

### Dockerfile
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

### Explanation
- Uses nginx server
- Serves static HTML
- Exposes port 80

---

## 6. Backend Implementation

### package.json
```json
{
 "name":"backend",
 "dependencies":{
  "express":"^4.18.2",
  "mysql2":"^3.11.0"
 }
}
```

---

### server.js
```javascript
const express=require('express');
const mysql=require('mysql2');

const app=express();

const db=mysql.createConnection({
 host:'database',
 user:'root',
 password:'rootpassword',
 database:'ecommerce_db'
});

db.connect((err)=>{
 if(err){
  console.log("DB error");
 }
});

app.get('/',(req,res)=>res.send("Hello Backend"));

app.listen(5000);
```

### Explanation
- Express handles HTTP
- MySQL connection uses service name
- Backend exposes API

---

### Dockerfile
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY server.js .
EXPOSE 5000
CMD ["node","server.js"]
```

---

## 7. Docker Compose File

```yaml
services:
 frontend:
  build: ./frontend
  ports:
   - "80:80"

 backend:
  build: ./backend
  ports:
   - "5000:5000"
  depends_on:
   - database

 database:
  image: mysql:8.0
  environment:
   MYSQL_ROOT_PASSWORD: rootpassword
   MYSQL_DATABASE: ecommerce_db
  ports:
   - "3306:3306"
  volumes:
   - mysql_data:/var/lib/mysql

volumes:
 mysql_data:
```

---

## 8. Explanation of Compose

- services define components
- build creates images
- ports expose services
- depends_on controls startup order
- volumes ensure persistence

---

## 9. Deployment

```bash
docker compose up --build
```

---

## 10. Testing

Open browser:
- http://localhost
- http://localhost:5000/

---

## 11. Communication Logic

- frontend → backend via localhost
- backend → database via service name

---

## 12. Why this is Microservices

- separation of concerns
- independent scaling
- fault isolation

---

## 13. Improvements over Monolith

- scalability
- maintainability
- modularity

---

## 14. Common Mistakes

### Wrong DB host
Use 'database', not localhost

### No volume
Data loss risk

### Port conflict
Change host ports

---

## 15. Conclusion

This solution transforms a monolithic system into a scalable microservices architecture using Docker Compose.

