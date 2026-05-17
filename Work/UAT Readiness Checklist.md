### **1️⃣ Code & Feature Readiness**

☐ Ensure all core features are implemented and tested in **local development**  
☐ Verify all **API endpoints** work as expected  
☐ Ensure **database migrations** are up to date  
☐ Implement **logging** for debugging in UAT (but no sensitive data in logs)  
☐ Test batch jobs and ensure they work correctly

---

### **2️⃣ Docker Setup for UAT**

☐ **Keep separate Dockerfiles** for **development** and **production**  
☐ Modify `docker-compose.override.yml` for development (**hot reload**)  
☐ Create a **production-ready Dockerfile** (optimized, no hot reload)  
☐ Optimize images by using **multi-stage builds** to reduce image size  
☐ Configure environment variables properly (`.env` for UAT, secrets for production)

---

### **3️⃣ Database & Data Preparation**

☐ Set up a **UAT MySQL database** (could be a copy of production with masked data)  
☐ Migrate schema and ensure **UAT database is seeded** correctly  
☐ Back up the existing database before making any changes  
☐ Ensure test users and roles are created for UAT testing

---

### **4️⃣ UAT Server Setup**

☐ Ensure **UAT Linux server** (Ubuntu) is ready  
☐ Install **Docker & Docker Compose** on the UAT machine  
☐ Transfer project files and environment variables securely  
☐ Set up **certificates & domain** if UAT needs HTTPS

---

### **5️⃣ Running UAT Containers**

☐ Use **docker-compose.prod.yml** for UAT deployment  
☐ Start the UAT environment with:

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

☐ Verify that all **containers are running properly**:

```bash
docker ps
```

☐ Ensure logs are clean and no **crashes/errors** occur

---

### **6️⃣ Testing & Debugging**

☐ Test **Next.js frontend**, **.NET API**, and **database connectivity**  
☐ Validate batch jobs & **cron jobs** if applicable  
☐ Test for **performance issues** (slow endpoints, memory leaks)  
☐ Have test users **run workflows** to simulate real usage  
☐ Collect **feedback and bugs from UAT testers**

---

# **📌 Setting Up Docker for Production**

### **1️⃣ Development Docker Setup**

Your current **Docker setup for development** likely includes:  
✅ **Hot reload** for Next.js  
✅ **Automatic rebuild** for .NET backend  
✅ **Mounted volumes** for live file updates

Here’s an example **development Dockerfile for Next.js**:

```dockerfile
FROM node:18 AS dev
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

This setup supports **hot reloading** via `npm run dev` inside a mounted volume.

---

### **2️⃣ Creating a Production-Ready Dockerfile**

For **production**, we need:  
✅ A **smaller image** (remove unnecessary dev dependencies)  
✅ **Optimized builds** (precompiled Next.js, compiled .NET, MySQL config)  
✅ **No unnecessary rebuilds** (use multi-stage builds)

#### **Next.js Production Dockerfile**

```dockerfile
# Build Stage
FROM node:18 AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

# Production Stage
FROM node:18-alpine AS runner
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./

CMD ["npm", "run", "start"]
```

✅ **Why?**

- **Multi-stage build**: The first stage **builds** Next.js, the second **runs** it with only needed files.
- **Alpine base image**: Reduces image size.

---

#### **.NET Production Dockerfile**

```dockerfile
# Build Stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o /out

# Production Stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /out ./
CMD ["dotnet", "YourProject.dll"]
```

✅ **Why?**

- First stage **compiles** the .NET app.
- Second stage **runs only necessary files**, reducing size.

---

#### **Optimized `docker-compose.prod.yml`**

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: qo-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: hr_db
      MYSQL_USER: hr_user
      MYSQL_PASSWORD: hr_password
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"

  backend:
    image: qo-hr-backend
    container_name: qo-backend
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    environment:
      - DATABASE_URL=mysql://hr_user:hr_password@mysql/hr_db
    depends_on:
      - mysql
    ports:
      - "5000:5000"

  frontend:
    image: qo-hr-frontend
    container_name: qo-frontend
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    depends_on:
      - backend
    ports:
      - "80:3000"

volumes:
  mysql_data:
```

✅ **Why?**

- Uses **production-ready builds** (no dev tools, no hot reload).
- Maps **port 80** to serve Next.js in production mode.
- Uses environment variables for **secure database connections**.

---

# **Final Steps for UAT Deployment**

### **1️⃣ Build & Push Images to Docker Hub (or Private Registry)**

Instead of building locally every time, push images to **Docker Hub**:

```bash
docker build -t yourusername/qo-hr-frontend -f frontend/Dockerfile.prod .
docker build -t yourusername/qo-hr-backend -f backend/Dockerfile.prod .
docker push yourusername/qo-hr-frontend
docker push yourusername/qo-hr-backend
```

### **2️⃣ Deploy to UAT Server**

On the **UAT server**, pull images and run:

```bash
docker pull yourusername/qo-hr-frontend
docker pull yourusername/qo-hr-backend
docker-compose -f docker-compose.prod.yml up -d
```

### **3️⃣ Monitor & Debug**

- **Check logs**:
    
    ```bash
    docker logs -f qo-backend
    ```
    
- **Restart a container**:
    
    ```bash
    docker restart qo-backend
    ```
    

### **4️⃣ Perform UAT Testing**

✅ Run through the **UAT checklist**  
✅ Fix bugs before **production deployment**

---

# **🎯 Final Thoughts**

- **Keep dev & prod Dockerfiles separate**
- **Use multi-stage builds** to keep images small
- **Deploy using a private Docker registry or Docker Hub**
- **Monitor logs and ensure smooth deployment before full rollout**