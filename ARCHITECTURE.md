# STOCKHUB PRO - Complete Architecture Documentation

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Frontend Architecture](#frontend-architecture)
4. [Backend Integration Guide](#backend-integration-guide)
5. [ML Model Integration](#ml-model-integration)
6. [Database Schema](#database-schema)
7. [API Endpoints](#api-endpoints)
8. [Deployment Guide](#deployment-guide)
9. [Security Best Practices](#security-best-practices)

---

## 🎯 Project Overview

**STOCKHUB PRO** is an AI-powered stock recommendation platform that provides:
- Real-time stock analysis using ML models
- Technical indicator calculations (RSI, MACD, SMA, EMA)
- Portfolio tracking and management
- AI chatbot for stock market queries
- User authentication with JWT

### Current Implementation
This is a **production-ready React frontend** with mock backend functionality. It demonstrates the complete user experience and can be easily connected to real backend services.

---

## 🛠 Technology Stack

### Frontend (Current Implementation)
- **React 18** - UI framework
- **TypeScript** - Type safety
- **Tailwind CSS** - Styling
- **Recharts** - Data visualization
- **Context API** - State management

### Backend (Integration Required)
- **Node.js + Express** - REST API server
- **MongoDB** - Database
- **JWT** - Authentication
- **bcrypt** - Password hashing

### ML/AI (Integration Required)
- **Python 3.9+** - ML backend
- **FastAPI** - ML API server
- **scikit-learn** - Machine learning
- **pandas** - Data processing
- **numpy** - Numerical computing

---

## 🏗 Frontend Architecture

### File Structure
```
src/
├── app/
│   ├── App.tsx                 # Main application component
│   ├── context/
│   │   ├── AuthContext.tsx     # Authentication state management
│   │   └── PortfolioContext.tsx # Portfolio state management
│   ├── components/
│   │   ├── pages/
│   │   │   ├── LandingPage.tsx
│   │   │   ├── LoginPage.tsx
│   │   │   ├── RegisterPage.tsx
│   │   │   ├── Dashboard.tsx
│   │   │   ├── PortfolioPage.tsx
│   │   │   ├── RecommendationsPage.tsx
│   │   │   └── ChatbotPage.tsx
│   │   ├── shared/
│   │   │   ├── Navigation.tsx
│   │   │   └── AnimatedBackground.tsx
│   │   └── ui/              # Reusable UI components
│   ├── utils/
│   │   ├── mlEngine.ts      # ML simulation (replace with API calls)
│   │   └── chatbot.ts       # Chatbot logic (replace with API calls)
│   └── styles/
│       └── custom.css       # Custom styles
```

### Key Components

#### 1. AuthContext (`/src/app/context/AuthContext.tsx`)
Manages user authentication state.

**Current:** Mock authentication with localStorage  
**Production:** Replace with API calls

```typescript
// Current (Mock)
const login = async (email: string, password: string) => {
  // Mock validation
  return true;
};

// Production (Replace with)
const login = async (email: string, password: string) => {
  const response = await fetch('http://your-api.com/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });
  
  const data = await response.json();
  if (data.token) {
    setToken(data.token);
    setUser(data.user);
    return true;
  }
  return false;
};
```

#### 2. ML Engine (`/src/app/utils/mlEngine.ts`)
Simulates stock analysis and predictions.

**Current:** Client-side calculations  
**Production:** Call Python ML API

```typescript
// Current (Mock)
export function analyzeStock(symbol: string, currentPrice: number) {
  // Local calculations
  const rsi = calculateRSI(prices);
  return analysis;
}

// Production (Replace with)
export async function analyzeStock(symbol: string) {
  const response = await fetch('http://ml-api.com/api/analyze', {
    method: 'POST',
    headers: { 
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify({ symbol })
  });
  
  return await response.json();
}
```

---

## 🔌 Backend Integration Guide

### 1. Node.js + Express Backend

#### Setup
```bash
mkdir backend
cd backend
npm init -y
npm install express mongoose jsonwebtoken bcryptjs cors dotenv
npm install -D typescript @types/express @types/node ts-node nodemon
```

#### Project Structure
```
backend/
├── src/
│   ├── server.ts           # Entry point
│   ├── config/
│   │   └── database.ts     # MongoDB connection
│   ├── models/
│   │   ├── User.ts
│   │   ├── Stock.ts
│   │   ├── Portfolio.ts
│   │   └── Prediction.ts
│   ├── routes/
│   │   ├── auth.ts
│   │   ├── portfolio.ts
│   │   ├── stocks.ts
│   │   └── predictions.ts
│   ├── middleware/
│   │   └── auth.ts
│   └── controllers/
│       ├── authController.ts
│       ├── portfolioController.ts
│       └── stockController.ts
├── .env
└── package.json
```

#### Sample Code: Server Setup (`src/server.ts`)
```typescript
import express from 'express';
import cors from 'cors';
import mongoose from 'mongoose';
import dotenv from 'dotenv';

dotenv.config();

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI!)
  .then(() => console.log('✅ MongoDB Connected'))
  .catch(err => console.error('❌ MongoDB Error:', err));

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/portfolio', portfolioRoutes);
app.use('/api/stocks', stockRoutes);
app.use('/api/predictions', predictionRoutes);

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
});
```

#### Sample Code: User Model (`src/models/User.ts`)
```typescript
import mongoose, { Schema, Document } from 'mongoose';

export interface IUser extends Document {
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

const UserSchema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  createdAt: { type: Date, default: Date.now }
});

export default mongoose.model<IUser>('User', UserSchema);
```

#### Sample Code: Auth Controller (`src/controllers/authController.ts`)
```typescript
import { Request, Response } from 'express';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import User from '../models/User';

export const register = async (req: Request, res: Response) => {
  try {
    const { name, email, password } = req.body;

    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ message: 'User already exists' });
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 12);

    // Create user
    const user = new User({
      name,
      email,
      password: hashedPassword
    });

    await user.save();

    // Generate JWT
    const token = jwt.sign(
      { userId: user._id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: '7d' }
    );

    res.status(201).json({
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email
      }
    });
  } catch (error) {
    res.status(500).json({ message: 'Server error' });
  }
};

export const login = async (req: Request, res: Response) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }

    // Check password
    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }

    // Generate JWT
    const token = jwt.sign(
      { userId: user._id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: '7d' }
    );

    res.json({
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email
      }
    });
  } catch (error) {
    res.status(500).json({ message: 'Server error' });
  }
};
```

#### Sample Code: Auth Middleware (`src/middleware/auth.ts`)
```typescript
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

export interface AuthRequest extends Request {
  userId?: string;
}

export const authMiddleware = (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return res.status(401).json({ message: 'No token provided' });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
    req.userId = decoded.userId;
    next();
  } catch (error) {
    res.status(401).json({ message: 'Invalid token' });
  }
};
```

#### Environment Variables (`.env`)
```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/stockhub
JWT_SECRET=your-super-secret-jwt-key-change-this
ML_API_URL=http://localhost:8000
```

---

## 🤖 ML Model Integration

### Python ML Backend Setup

#### Project Structure
```
ml-backend/
├── app/
│   ├── main.py             # FastAPI server
│   ├── model.py            # ML model logic
│   ├── indicators.py       # Technical indicators
│   ├── train.py            # Model training
│   └── chatbot.py          # Chatbot AI
├── models/
│   └── stock_model.pkl     # Trained model
├── data/
│   └── historical/         # Historical stock data
├── requirements.txt
└── .env
```

#### Sample Code: FastAPI Server (`app/main.py`)
```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
import uvicorn
from model import analyze_stock
from chatbot import generate_response

app = FastAPI(title="StockHub ML API")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

class StockRequest(BaseModel):
    symbol: str

class ChatRequest(BaseModel):
    message: str
    conversation_history: list = []

@app.get("/")
def root():
    return {"message": "StockHub ML API - Running"}

@app.post("/api/analyze")
async def analyze(request: StockRequest):
    """
    Analyze a stock and return BUY/HOLD/SELL recommendation
    """
    try:
        analysis = analyze_stock(request.symbol)
        return analysis
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/api/chat")
async def chat(request: ChatRequest):
    """
    AI chatbot endpoint
    """
    try:
        response = generate_response(
            request.message,
            request.conversation_history
        )
        return {"response": response}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

#### Sample Code: Technical Indicators (`app/indicators.py`)
```python
import pandas as pd
import numpy as np

def calculate_rsi(prices, period=14):
    """
    Calculate Relative Strength Index
    """
    delta = prices.diff()
    gain = (delta.where(delta > 0, 0)).rolling(window=period).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=period).mean()
    
    rs = gain / loss
    rsi = 100 - (100 / (1 + rs))
    return rsi.iloc[-1]

def calculate_macd(prices, fast=12, slow=26, signal=9):
    """
    Calculate MACD (Moving Average Convergence Divergence)
    """
    ema_fast = prices.ewm(span=fast).mean()
    ema_slow = prices.ewm(span=slow).mean()
    
    macd = ema_fast - ema_slow
    signal_line = macd.ewm(span=signal).mean()
    
    return {
        'macd': macd.iloc[-1],
        'signal': signal_line.iloc[-1],
        'histogram': (macd - signal_line).iloc[-1]
    }

def calculate_sma(prices, period=20):
    """
    Calculate Simple Moving Average
    """
    return prices.rolling(window=period).mean().iloc[-1]

def calculate_ema(prices, period=50):
    """
    Calculate Exponential Moving Average
    """
    return prices.ewm(span=period).mean().iloc[-1]

def calculate_bollinger_bands(prices, period=20, std_dev=2):
    """
    Calculate Bollinger Bands
    """
    sma = prices.rolling(window=period).mean()
    std = prices.rolling(window=period).std()
    
    upper_band = sma + (std * std_dev)
    lower_band = sma - (std * std_dev)
    
    return {
        'upper': upper_band.iloc[-1],
        'middle': sma.iloc[-1],
        'lower': lower_band.iloc[-1]
    }
```

#### Sample Code: ML Model (`app/model.py`)
```python
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestClassifier
import yfinance as yf
from indicators import *
import joblib

def fetch_stock_data(symbol, period='1y'):
    """
    Fetch historical stock data
    """
    ticker = yf.Ticker(symbol)
    df = ticker.history(period=period)
    return df

def create_features(df):
    """
    Create features for ML model
    """
    df['RSI'] = calculate_rsi(df['Close'])
    
    macd_data = calculate_macd(df['Close'])
    df['MACD'] = macd_data['macd']
    df['MACD_Signal'] = macd_data['signal']
    
    df['SMA_20'] = calculate_sma(df['Close'], 20)
    df['SMA_50'] = calculate_sma(df['Close'], 50)
    df['EMA_50'] = calculate_ema(df['Close'], 50)
    
    # Price momentum
    df['Price_Change'] = df['Close'].pct_change()
    df['Volume_Change'] = df['Volume'].pct_change()
    
    # Trend
    df['Trend'] = np.where(df['Close'] > df['SMA_50'], 1, -1)
    
    return df

def analyze_stock(symbol):
    """
    Main analysis function
    Returns BUY/HOLD/SELL recommendation
    """
    try:
        # Fetch data
        df = fetch_stock_data(symbol)
        
        # Calculate indicators
        df = create_features(df)
        
        # Get latest values
        latest = df.iloc[-1]
        current_price = latest['Close']
        
        rsi = calculate_rsi(df['Close'])
        macd_data = calculate_macd(df['Close'])
        sma_20 = calculate_sma(df['Close'], 20)
        ema_50 = calculate_ema(df['Close'], 50)
        
        # Decision logic
        signals = []
        
        # RSI signals
        if rsi < 30:
            signals.append('BUY')
            reasoning = f"RSI at {rsi:.2f} indicates oversold conditions"
        elif rsi > 70:
            signals.append('SELL')
            reasoning = f"RSI at {rsi:.2f} indicates overbought conditions"
        else:
            signals.append('HOLD')
            reasoning = f"RSI at {rsi:.2f} shows neutral conditions"
        
        # MACD signals
        if macd_data['macd'] > macd_data['signal']:
            signals.append('BUY')
        else:
            signals.append('SELL')
        
        # Moving average signals
        if current_price > sma_20 and current_price > ema_50:
            signals.append('BUY')
        elif current_price < sma_20 and current_price < ema_50:
            signals.append('SELL')
        else:
            signals.append('HOLD')
        
        # Final recommendation (majority vote)
        from collections import Counter
        vote = Counter(signals).most_common(1)[0][0]
        
        # Calculate confidence
        confidence = (signals.count(vote) / len(signals)) * 100
        
        # Target price (simple projection)
        if vote == 'BUY':
            target_price = current_price * 1.10
        elif vote == 'SELL':
            target_price = current_price * 0.90
        else:
            target_price = current_price
        
        return {
            'symbol': symbol,
            'recommendation': vote,
            'confidence': confidence,
            'current_price': float(current_price),
            'target_price': float(target_price),
            'indicators': {
                'rsi': float(rsi),
                'macd': float(macd_data['macd']),
                'sma20': float(sma_20),
                'ema50': float(ema_50),
                'volume': int(latest['Volume'])
            },
            'reasoning': reasoning,
            'signals': {
                'technical': 'bullish' if vote == 'BUY' else 'bearish' if vote == 'SELL' else 'neutral',
                'momentum': 'strong' if confidence > 80 else 'moderate' if confidence > 60 else 'weak',
                'trend': 'uptrend' if current_price > ema_50 else 'downtrend' if current_price < ema_50 else 'sideways'
            }
        }
    
    except Exception as e:
        raise Exception(f"Error analyzing {symbol}: {str(e)}")
```

#### Requirements (`requirements.txt`)
```
fastapi==0.104.1
uvicorn==0.24.0
pandas==2.1.3
numpy==1.26.2
scikit-learn==1.3.2
yfinance==0.2.32
python-dotenv==1.0.0
joblib==1.3.2
```

#### Installation
```bash
cd ml-backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python app/main.py
```

---

## 🗄 Database Schema

### MongoDB Collections

#### 1. Users Collection
```javascript
{
  _id: ObjectId,
  name: String,
  email: String (unique, indexed),
  password: String (hashed with bcrypt),
  createdAt: Date,
  updatedAt: Date
}
```

#### 2. Portfolio Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: Users),
  stocks: [
    {
      symbol: String,
      name: String,
      quantity: Number,
      buyPrice: Number,
      currentPrice: Number,
      addedAt: Date
    }
  ],
  createdAt: Date,
  updatedAt: Date
}
```

#### 3. Predictions Collection
```javascript
{
  _id: ObjectId,
  symbol: String,
  recommendation: String (BUY/HOLD/SELL),
  confidence: Number,
  targetPrice: Number,
  currentPrice: Number,
  indicators: {
    rsi: Number,
    macd: Number,
    sma20: Number,
    ema50: Number,
    volume: Number
  },
  createdAt: Date
}
```

#### 4. Conversations Collection (Chatbot)
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: Users),
  messages: [
    {
      role: String (user/assistant),
      content: String,
      timestamp: Date
    }
  ],
  createdAt: Date,
  updatedAt: Date
}
```

---

## 🔗 API Endpoints

### Authentication
```
POST   /api/auth/register     # Register new user
POST   /api/auth/login        # Login user
GET    /api/auth/me           # Get current user (protected)
```

### Portfolio
```
GET    /api/portfolio         # Get user portfolio (protected)
POST   /api/portfolio/add     # Add stock to portfolio (protected)
DELETE /api/portfolio/:id     # Remove stock from portfolio (protected)
PUT    /api/portfolio/:id     # Update stock in portfolio (protected)
```

### Stocks
```
GET    /api/stocks/search/:symbol    # Search stock
GET    /api/stocks/popular           # Get popular stocks
GET    /api/stocks/:symbol/history   # Get historical data
```

### Predictions (ML)
```
POST   /api/predictions/analyze      # Analyze stock (calls ML API)
GET    /api/predictions/recent       # Get recent predictions
```

### Chatbot
```
POST   /api/chat/message             # Send message to chatbot
GET    /api/chat/history             # Get conversation history
```

---

## 🚀 Deployment Guide

### Frontend Deployment (Vercel/Netlify)

#### Vercel
```bash
npm install -g vercel
vercel login
vercel
```

#### Netlify
```bash
npm run build
# Upload dist/ folder to Netlify
```

### Backend Deployment (Heroku/Railway)

#### Heroku
```bash
heroku create stockhub-api
heroku addons:create mongolab
git push heroku main
```

#### Railway
1. Connect GitHub repo
2. Add MongoDB plugin
3. Set environment variables
4. Deploy

### ML Backend Deployment (AWS/GCP)

#### Docker Setup
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
docker build -t stockhub-ml .
docker run -p 8000:8000 stockhub-ml
```

---

## 🔒 Security Best Practices

### 1. Authentication
- ✅ Use JWT with short expiration (7 days max)
- ✅ Store tokens in HTTP-only cookies (not localStorage in production)
- ✅ Implement refresh tokens
- ✅ Hash passwords with bcrypt (salt rounds: 12)

### 2. API Security
- ✅ Rate limiting (express-rate-limit)
- ✅ CORS configuration
- ✅ Input validation (express-validator)
- ✅ SQL/NoSQL injection prevention
- ✅ XSS protection (helmet.js)

### 3. Environment Variables
```env
# NEVER commit these to Git
JWT_SECRET=use-a-strong-random-secret-here
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/db
ML_API_KEY=your-ml-api-key
```

### 4. HTTPS
- ✅ Always use HTTPS in production
- ✅ Redirect HTTP to HTTPS
- ✅ Use SSL certificates (Let's Encrypt)

---

## 📊 Performance Optimization

### Frontend
- ✅ Code splitting (React.lazy)
- ✅ Memoization (React.memo, useMemo)
- ✅ Virtual scrolling for large lists
- ✅ Image optimization

### Backend
- ✅ Database indexing
- ✅ Caching (Redis)
- ✅ Connection pooling
- ✅ Gzip compression

### ML API
- ✅ Model caching
- ✅ Batch predictions
- ✅ Async processing (Celery)
- ✅ Result caching

---

## 📝 Testing

### Frontend Testing
```bash
npm install -D @testing-library/react jest
npm test
```

### Backend Testing
```bash
npm install -D jest supertest
npm test
```

### ML Testing
```bash
pip install pytest
pytest
```

---

## 🎓 Learning Resources

- **React**: https://react.dev
- **Express**: https://expressjs.com
- **MongoDB**: https://www.mongodb.com/docs
- **FastAPI**: https://fastapi.tiangolo.com
- **scikit-learn**: https://scikit-learn.org

---

## 📞 Support

For questions or issues, refer to the inline code comments or create a GitHub issue.

**Built with ❤️ for StockHub Pro**
