# BasicWebsiteSetup
FastAPI backend, React vite frontend, using docker compose. 


## First 

We need to have a `backend` and `frontend` folder in the root directory.

### Backend

For the backend, we just need to have:

1. a `requirements.txt` file with the dependencies for the FastAPI backend.

    ```
    fastapi
    uvicorn[standard]
    sqlalchemy
    alembic
    psycopg2-binary
    pydantic-settings
    python-dotenv
    ```
    
2. a `app` folder with the FastAPI application code, and inside there should be a `main.py` file with the FastAPI application code. (We will also add a `database.py` file for the database connection).

    ```python
    from fastapi import FastAPI

    app = FastAPI(title="Cookbook API")

    @app.get("/")
    def root():
        return {"message": "Cookbook API is running"}
    ```

    ```python
    from sqlalchemy import create_engine
    from sqlalchemy.orm import sessionmaker, DeclarativeBase
    import os

    DATABASE_URL = os.getenv("DATABASE_URL")

    engine = create_engine(DATABASE_URL)
    SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

    class Base(DeclarativeBase):
        pass

    def get_db():
        db = SessionLocal()
        try:
            yield db
        finally:
            db.close()
    ```
    
    
    
3. a `Dockerfile` to build the backend image.

    ```dockerfile
    FROM python:3.12-slim

    WORKDIR /app

    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt

    COPY . .

    CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
    ```
    
### Frontend

For the frontend, we need to have install Vite and React, and then create a `Dockerfile` to build the frontend image. (I am going to add tailwindcss)

1. Install Vite, React and tailwindcss.

    ```bash
    npm create vite@latest frontend --template react
    cd frontend
    npm install
    npm install -D tailwindcss @tailwindcss/vite
    ```
    
    Add this to `vite.config.js`:

    ```javascript
    import { defineConfig } from 'vite'
    import react from '@vitejs/plugin-react'
    import tailwindcss from '@tailwindcss/vite'

    // https://vite.dev/config/
    export default defineConfig({
    plugins: [
        react(),
        tailwindcss(),
    ],
    })
    ```

2. Create a `Dockerfile` to build the frontend image.

    ```dockerfile
    FROM node:20-slim

    WORKDIR /app

    COPY package*.json ./
    RUN npm install

    COPY . .

    CMD ["npm", "run", "dev", "--", "--host"]
    ```


### Docker compose

Finally, we need to create a `docker-compose.yml` file in the root directory to run both the backend and frontend services together.

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: firstwebpage
      POSTGRES_PASSWORD: firstwebpage
      POSTGRES_DB: firstwebpage
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
    environment:
      DATABASE_URL: postgresql://firstwebpage:firstwebpage@db/firstwebpage
    depends_on:
      - db
      
  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    volumes:
      - ./frontend:/app
      - /app/node_modules
      
volumes:
  pgdata:
```


## Running the application

To run the application, we just need to run the following command in the root directory:

```bash
docker-compose up --build
```

To turn it down, we can run:

```bash
docker-compose down
```


