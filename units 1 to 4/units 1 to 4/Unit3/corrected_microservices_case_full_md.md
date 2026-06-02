# FULL DETAILED SOLUTION (CORRECTED): Migrating from Monolithic to Microservices

---

## 1. Understanding the Case (Simple Explanation)

A startup has a **single large application (monolith)** where:
- Frontend, backend, and database are on the same server

### Problem:
- Slow during high traffic
- Hard to scale
- Risky updates
- Poor fault isolation

### Goal:
Convert into **microservices using Docker Compose**:
- Frontend → separate container
- Backend → separate container
- Database → separate container

---

## 2. Monolithic vs Microservices

### Monolithic
- Single deployment
- Tight coupling
- Difficult scaling

### Microservices
- Independent services
- Scalable
- Easy maintenance

---

## 3. Architecture

Flow:

User → Frontend → Backend → Database

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

---

### frontend/Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

---

## 6. Backend Implementation

### package.json

```json
{
 "name": "backend",
 "version": "1.0.0",
 "main": "server.js",
 "scripts": {
  "start": "node server.js"
 },
 "dependencies": {
  "express": "^4.18.2",
  "mysql2": "^3.11.0"
 }
}
```

---

### server.js

```javascript
const express = require('express');
const mysql = require('mysql2');

const app = express();

const db = mysql.createConnection({
 host: 'database',
 user: 'root',
 password: 'rootpassword',
 database: 'ecommerce_db'
});

db.connect((err)=>{
 if(err){
  console.log("DB connection failed", err.message);
 } else {
  console.log("Connected to DB");
 }
});

app.get('/', (req,res)=>{
 res.send("Hello from Backend");
});

app.listen(5000, ()=>{
 console.log("Server running on port 5000");
});
```

---

### backend/Dockerfile (CORRECTED)

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 5000
CMD ["npm", "start"]
```

---

## 7. docker-compose.yml (CORRECTED)

```yaml
services:
 frontend:
  build: ./frontend
  platform: linux/amd64
  ports:
   - "80:80"
  depends_on:
   - backend

 backend:
  build: ./backend
  platform: linux/amd64
  ports:
   - "5000:5000"
  depends_on:
   - database

 database:
  image: mysql:8.0
  platform: linux/amd64
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

## 8. Commands to Run

```bash
docker compose down
docker system prune -a -f
docker compose build --no-cache
docker compose up
```

---

## 9. Testing

Open browser:
- http://localhost
- http://localhost:5000

---

## 10. Communication Logic

- Frontend → Backend using localhost:5000
- Backend → Database using service name "database"

---

## 11. Common Errors & Fix

### ERROR: exec format error

✔ Cause:
- Architecture mismatch

✔ Fix:
- Add `platform: linux/amd64`
- Use stable base image (`node:18`)

---

## 12. Conclusion

This setup successfully converts a monolithic system into a scalable microservices architecture using Docker Compose with proper platform compatibility.

