# rix-cn-crisis-nexus

# RIX.GG Crisis Nexus (RIX-CN) Backend Implementation

import os
from fastapi import FastAPI, HTTPException, Depends
from sqlalchemy import create_engine, Column, Integer, String, Float, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel
from datetime import datetime
from typing import List
import uvicorn

# Database setup
SQLALCHEMY_DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./rix_cn.db")
engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Models
class Incident(Base):
    __tablename__ = "incidents"
    id = Column(Integer, primary_key=True, index=True)
    type = Column(String, index=True)
    location = Column(String)
    severity = Column(Integer)
    timestamp = Column(DateTime, default=datetime.utcnow)
    description = Column(String)

class Resource(Base):
    __tablename__ = "resources"
    id = Column(Integer, primary_key=True, index=True)
    type = Column(String, index=True)
    quantity = Column(Integer)
    location = Column(String)
    status = Column(String)

# Pydantic models for request/response
class IncidentCreate(BaseModel):
    type: str
    location: str
    severity: int
    description: str

class IncidentResponse(BaseModel):
    id: int
    type: str
    location: str
    severity: int
    timestamp: datetime
    description: str

class ResourceCreate(BaseModel):
    type: str
    quantity: int
    location: str
    status: str

class ResourceResponse(BaseModel):
    id: int
    type: str
    quantity: int
    location: str
    status: str

# Create tables
Base.metadata.create_all(bind=engine)

# FastAPI app
app = FastAPI(title="RIX.GG Crisis Nexus (RIX-CN) API")

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Routes
@app.post("/incidents/", response_model=IncidentResponse)
def create_incident(incident: IncidentCreate, db: Session = Depends(get_db)):
    db_incident = Incident(**incident.dict())
    db.add(db_incident)
    db.commit()
    db.refresh(db_incident)
    return db_incident

@app.get("/incidents/", response_model=List[IncidentResponse])
def list_incidents(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    incidents = db.query(Incident).offset(skip).limit(limit).all()
    return incidents

@app.post("/resources/", response_model=ResourceResponse)
def create_resource(resource: ResourceCreate, db: Session = Depends(get_db)):
    db_resource = Resource(**resource.dict())
    db.add(db_resource)
    db.commit()
    db.refresh(db_resource)
    return db_resource

@app.get("/resources/", response_model=List[ResourceResponse])
def list_resources(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    resources = db.query(Resource).offset(skip).limit(limit).all()
    return resources

# Main entry point
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)

"""
README.md

# RIX.GG Crisis Nexus (RIX-CN)

## Project Description
RIX.GG Crisis Nexus (RIX-CN) is a centralized, efficient, and scalable disaster response system. It combines real-time data collection, AI-driven resource allocation, and advanced communication tools to streamline crisis management and improve response times during disasters.

## Installation and Setup

### Prerequisites
- Python 3.8+
- pip (Python package manager)
- Git

### Steps
1. Clone the repository:
   ```
   git clone https://github.com/your-username/rix-cn.git
   cd rix-cn
   ```

2. Create a virtual environment:
   ```
   python -m venv venv
   source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
   ```

3. Install dependencies:
   ```
   pip install -r requirements.txt
   ```

4. Set up environment variables:
   ```
   export DATABASE_URL="sqlite:///./rix_cn.db"  # Or your preferred database URL
   ```

## Running the Project
1. Start the FastAPI server:
   ```
   uvicorn main:app --reload
   ```

2. Access the API documentation at `http://localhost:8000/docs`

## Testing
To run the tests, execute:
```
pytest
```

## Credits
This project was developed by the RIX.GG team:
- Abdullah Haroon
- Aisha Shakil
- Abdullah Toheed
- Ahmed Faisal

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

"""

"""
requirements.txt

fastapi==0.68.0
uvicorn==0.15.0
sqlalchemy==1.4.23
pydantic==1.8.2
pytest==6.2.5
"""

"""
LICENSE

MIT License

Copyright (c) 2025 RIX.GG Team

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
"""
