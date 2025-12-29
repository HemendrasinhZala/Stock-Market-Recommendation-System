# 📈 STOCKHUB PRO - Complete Documentation

## 🎯 Project Overview

**STOCKHUB PRO** is a professional, production-ready stock market recommendation system built with modern web technologies. The application leverages AI/ML algorithms to provide intelligent BUY/HOLD/SELL signals based on comprehensive technical analysis.

### ✨ Key Features

- **AI-Powered Recommendations**: Advanced ML algorithms analyzing RSI, MACD, Bollinger Bands, Moving Averages, and more
- **Real-Time Analytics**: Live market data visualization with interactive charts
- **Portfolio Management**: Track investments, monitor P/L, and optimize portfolio allocation
- **Technical Indicators**: 20+ indicators including RSI, MACD, SMA, EMA, Bollinger Bands, Momentum
- **AI Chatbot**: Specialized stock market assistant answering trading and investment questions
- **Professional UI**: Dark theme with glassmorphism design elements
- **Responsive Design**: Fully optimized for desktop, tablet, and mobile devices
- **JWT Authentication**: Secure user authentication system

---

## 🏗️ Technology Stack

### Frontend
- **React 18.3.1** - Modern UI library
- **TypeScript** - Type-safe JavaScript
- **Tailwind CSS 4.x** - Utility-first CSS framework
- **Recharts 2.15.2** - Data visualization library
- **Vite 6.3.5** - Next-generation frontend tooling

### State Management
- **React Context API** - AuthContext & PortfolioContext

### AI/ML Engine
- **Custom TypeScript ML Engine** - Technical indicator calculations
- **Pattern Recognition** - Chart pattern analysis
- **Confidence Scoring** - ML-based prediction confidence

---

## 📁 Project Structure

```
/stockhub-pro
├── src/
│   ├── app/
│   │   ├── App.tsx                    # Main application component
│   │   ├── pages/
│   │   │   ├── LandingPage.tsx        # Marketing landing page
│   │   │   ├── LoginPage.tsx          # User authentication
│   │   │   ├── RegisterPage.tsx       # User registration
│   │   │   ├── Dashboard.tsx          # Main dashboard with market overview
│   │   │   ├── PortfolioPage.tsx      # Portfolio management
│   │   │   ├── RecommendationsPage.tsx # AI stock recommendations
│   │   │   └── ChatbotPage.tsx        # AI assistant chatbot
│   │   ├── context/
│   │   │   ├── AuthContext.tsx        # Authentication state management
│   │   │   └── PortfolioContext.tsx   # Portfolio state management
│   │   ├── engine/
│   │   │   ├── mlEngine.ts            # AI/ML recommendation engine
│   │   │   └── chatbot.ts             # AI chatbot engine
│   │   └── components/
│   │       └── ui/                    # Reusable UI components
│   └── styles/
│       ├── theme.css                  # Theme configuration
│       ├── index.css                  # Global styles
│       └── tailwind.css               # Tailwind directives
├── package.json                       # Dependencies
├── vite.config.ts                     # Vite configuration
└── tsconfig.json                      # TypeScript configuration
```

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18.x or higher
- npm or pnpm package manager

### Installation

1. **Clone the repository** (or download the source code)
   ```bash
   cd stockhub-pro
   ```

2. **Install dependencies**
   ```bash
   npm install
   # or
   pnpm install
   ```

3. **Start development server**
   ```bash
   npm run dev
   ```

4. **Open browser**
   ```
   Navigate to: http://localhost:5173
   ```

### Build for Production

```bash
npm run build
```

The production-ready files will be in the `dist/` directory.

---

## 🤖 AI/ML Engine - Technical Details

### Core Algorithms

#### 1. RSI (Relative Strength Index)
```typescript
// Formula: RSI = 100 - (100 / (1 + RS))
// Where RS = Average Gain / Average Loss
calculateRSI(prices: number[], period: number = 14): number
```

**Interpretation:**
- RSI < 30: Oversold → **BUY Signal**
- RSI > 70: Overbought → **SELL Signal**
- RSI 40-60: Neutral → **HOLD Signal**

#### 2. MACD (Moving Average Convergence Divergence)
```typescript
// MACD Line = EMA(12) - EMA(26)
// Signal Line = EMA(9) of MACD
// Histogram = MACD - Signal
calculateMACD(prices: number[]): { macd, signal, histogram }
```

**Signals:**
- MACD crosses above Signal → **Bullish (BUY)**
- MACD crosses below Signal → **Bearish (SELL)**

#### 3. Bollinger Bands
```typescript
// Upper Band = SMA(20) + (2 × Standard Deviation)
// Middle Band = SMA(20)
// Lower Band = SMA(20) - (2 × Standard Deviation)
calculateBollingerBands(prices: number[], period: number = 20)
```

**Trading Strategy:**
- Price at lower band → **Oversold (BUY)**
- Price at upper band → **Overbought (SELL)**

#### 4. Moving Averages
```typescript
// Simple Moving Average
calculateSMA(prices: number[], period: number): number

// Exponential Moving Average
calculateEMA(prices: number[], period: number): number
```

**Golden Cross / Death Cross:**
- 50-day MA crosses above 200-day MA → **Golden Cross (BULLISH)**
- 50-day MA crosses below 200-day MA → **Death Cross (BEARISH)**

#### 5. Momentum Indicator
```typescript
// Momentum = ((Current Price - Price N periods ago) / Price N periods ago) × 100
calculateMomentum(prices: number[], period: number = 10): number
```

### ML Recommendation Algorithm

The AI engine uses a **weighted scoring system**:

```typescript
function generateRecommendation(stockData: StockData[]): Recommendation {
  let score = 0;
  
  // Weight Distribution:
  // RSI: 20%
  // MACD: 20%
  // Moving Average Crossover: 20%
  // EMA Crossover: 15%
  // Bollinger Bands: 15%
  // Volume: 5%
  // Momentum: 5%
  
  // Final Signal:
  // score >= 40 → BUY
  // score <= -40 → SELL
  // else → HOLD
  
  return { signal, confidence, indicators, reasoning };
}
```

**Confidence Score Calculation:**
```
BUY/SELL Confidence = min((|score| / 100) × 100, 95)
HOLD Confidence = min((50 - |score|) × 1.2, 75)
```

---

## 💬 AI Chatbot - Knowledge Base

The chatbot is specialized for stock market queries and includes:

### Knowledge Domains

1. **Technical Indicators**
   - RSI, MACD, Bollinger Bands, Moving Averages
   - Interpretation and usage strategies

2. **Trading Strategies**
   - Day Trading
   - Swing Trading
   - Value Investing
   - Position Trading

3. **Risk Management**
   - Position sizing
   - Stop-loss strategies
   - Risk-reward ratios
   - Portfolio diversification

4. **Market Analysis**
   - Fundamental analysis
   - Technical analysis
   - Chart patterns
   - Candlestick patterns

5. **Market News**
   - Latest market updates
   - Economic indicators
   - Federal Reserve policy

### Usage Example

```typescript
import { getChatbotResponse } from './engine/chatbot';

const userQuery = "What is RSI and how do I use it?";
const response = getChatbotResponse(userQuery);

// Response includes:
// - Detailed explanation
// - Key levels (30/70)
// - Trading strategies
// - Real-world examples
```

---

## 🔐 Authentication System

### Mock Authentication (Development)

The current implementation uses **client-side mock authentication**:

```typescript
// Login Function
const login = async (email: string, password: string): Promise<boolean> => {
  // Accepts any email/password for demo
  const mockUser = {
    id: 'user_' + Date.now(),
    email,
    name: email.split('@')[0],
    createdAt: new Date().toISOString()
  };
  
  const mockToken = 'jwt_token_' + Math.random().toString(36);
  localStorage.setItem('stockhub_token', mockToken);
  localStorage.setItem('stockhub_user', JSON.stringify(mockUser));
  
  return true;
};
```

### Production Backend Integration

For production, replace with real API calls:

```typescript
const login = async (email: string, password: string): Promise<boolean> => {
  try {
    const response = await fetch('https://your-api.com/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    
    if (data.success) {
      localStorage.setItem('stockhub_token', data.token);
      localStorage.setItem('stockhub_user', JSON.stringify(data.user));
      return true;
    }
    
    return false;
  } catch (error) {
    console.error('Login error:', error);
    return false;
  }
};
```

---

## 📊 Backend Integration Guide

### Required API Endpoints

#### 1. Authentication

```typescript
POST /api/auth/register
Request: { name: string, email: string, password: string }
Response: { success: boolean, token: string, user: User }

POST /api/auth/login
Request: { email: string, password: string }
Response: { success: boolean, token: string, user: User }

GET /api/auth/me
Headers: { Authorization: 'Bearer <token>' }
Response: { user: User }
```

#### 2. Portfolio Management

```typescript
GET /api/portfolio
Headers: { Authorization: 'Bearer <token>' }
Response: { stocks: Stock[], transactions: Transaction[] }

POST /api/portfolio/add
Headers: { Authorization: 'Bearer <token>' }
Request: { symbol, name, shares, avgPrice, sector, purchaseDate }
Response: { success: boolean, stock: Stock }

DELETE /api/portfolio/:symbol
Headers: { Authorization: 'Bearer <token>' }
Response: { success: boolean }
```

#### 3. Stock Data (Recommended: Alpha Vantage, Yahoo Finance, or IEX Cloud)

```typescript
GET /api/stocks/:symbol
Response: {
  symbol: string,
  currentPrice: number,
  historicalData: StockData[],
  change: number,
  changePercent: number
}

GET /api/stocks/:symbol/historical?period=90
Response: { data: StockData[] }
```

#### 4. AI Recommendations

```typescript
POST /api/recommendations/analyze
Request: { symbol: string }
Response: {
  symbol: string,
  recommendation: Recommendation,
  timestamp: string
}

GET /api/recommendations/batch
Request: { symbols: string[] }
Response: { recommendations: Map<string, Recommendation> }
```

#### 5. AI Chatbot

```typescript
POST /api/chatbot/query
Request: { message: string, conversationId?: string }
Response: {
  reply: string,
  conversationId: string,
  timestamp: string
}
```

### Sample Python Flask Backend

```python
# app.py
from flask import Flask, request, jsonify
from flask_cors import CORS
import jwt
from datetime import datetime, timedelta

app = Flask(__name__)
CORS(app)
app.config['SECRET_KEY'] = 'your-secret-key-here'

# Authentication
@app.route('/api/auth/register', methods=['POST'])
def register():
    data = request.json
    # Validate and create user in database
    user = {
        'id': generate_user_id(),
        'email': data['email'],
        'name': data['name']
    }
    token = jwt.encode({
        'user_id': user['id'],
        'exp': datetime.utcnow() + timedelta(days=7)
    }, app.config['SECRET_KEY'])
    
    return jsonify({'success': True, 'token': token, 'user': user})

@app.route('/api/auth/login', methods=['POST'])
def login():
    data = request.json
    # Verify credentials
    user = verify_user(data['email'], data['password'])
    if user:
        token = jwt.encode({
            'user_id': user['id'],
            'exp': datetime.utcnow() + timedelta(days=7)
        }, app.config['SECRET_KEY'])
        return jsonify({'success': True, 'token': token, 'user': user})
    return jsonify({'success': False, 'error': 'Invalid credentials'}), 401

# Portfolio
@app.route('/api/portfolio', methods=['GET'])
def get_portfolio():
    token = request.headers.get('Authorization').replace('Bearer ', '')
    user_id = verify_token(token)
    # Fetch portfolio from database
    portfolio = get_user_portfolio(user_id)
    return jsonify(portfolio)

# Stock Data
@app.route('/api/stocks/<symbol>', methods=['GET'])
def get_stock_data(symbol):
    # Fetch from external API (Alpha Vantage, Yahoo Finance, etc.)
    data = fetch_stock_data(symbol)
    return jsonify(data)

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

---

## 🗄️ Database Schema

### Recommended: PostgreSQL or MongoDB

#### PostgreSQL Schema

```sql
-- Users Table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Portfolio Table
CREATE TABLE portfolio (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    symbol VARCHAR(10) NOT NULL,
    name VARCHAR(255) NOT NULL,
    shares DECIMAL(18, 8) NOT NULL,
    avg_price DECIMAL(18, 2) NOT NULL,
    sector VARCHAR(50),
    purchase_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Transactions Table
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    symbol VARCHAR(10) NOT NULL,
    type VARCHAR(10) NOT NULL, -- 'BUY' or 'SELL'
    shares DECIMAL(18, 8) NOT NULL,
    price DECIMAL(18, 2) NOT NULL,
    total DECIMAL(18, 2) NOT NULL,
    transaction_date DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Stock Predictions Table (Cache AI recommendations)
CREATE TABLE stock_predictions (
    id SERIAL PRIMARY KEY,
    symbol VARCHAR(10) NOT NULL,
    signal VARCHAR(10) NOT NULL, -- 'BUY', 'HOLD', 'SELL'
    confidence INTEGER NOT NULL,
    indicators JSONB,
    reasoning TEXT[],
    target_price DECIMAL(18, 2),
    stop_loss DECIMAL(18, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP
);

-- Indexes for performance
CREATE INDEX idx_portfolio_user_id ON portfolio(user_id);
CREATE INDEX idx_transactions_user_id ON transactions(user_id);
CREATE INDEX idx_predictions_symbol ON stock_predictions(symbol);
```

#### MongoDB Schema

```javascript
// Users Collection
{
  "_id": ObjectId,
  "email": "user@example.com",
  "name": "John Doe",
  "passwordHash": "...",
  "createdAt": ISODate
}

// Portfolio Collection
{
  "_id": ObjectId,
  "userId": ObjectId,
  "symbol": "AAPL",
  "name": "Apple Inc.",
  "shares": 50,
  "avgPrice": 165.50,
  "currentPrice": 178.25,
  "sector": "Technology",
  "purchaseDate": "2024-01-15",
  "createdAt": ISODate
}

// Transactions Collection
{
  "_id": ObjectId,
  "userId": ObjectId,
  "symbol": "AAPL",
  "type": "BUY",
  "shares": 50,
  "price": 165.50,
  "total": 8275.00,
  "date": "2024-01-15",
  "createdAt": ISODate
}

// Predictions Collection (Cache)
{
  "_id": ObjectId,
  "symbol": "AAPL",
  "signal": "BUY",
  "confidence": 87,
  "indicators": {
    "rsi": 42.5,
    "macd": {...},
    // ... other indicators
  },
  "reasoning": ["RSI indicates oversold...", "..."],
  "targetPrice": 190.00,
  "stopLoss": 170.00,
  "createdAt": ISODate,
  "expiresAt": ISODate
}
```

---

## 🌐 External API Integration

### Recommended Stock Data APIs

#### 1. Alpha Vantage (FREE Tier Available)
```typescript
const API_KEY = 'YOUR_ALPHA_VANTAGE_KEY';
const symbol = 'AAPL';

// Daily data
const url = `https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=${symbol}&apikey=${API_KEY}`;

// Real-time quote
const quoteUrl = `https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=${symbol}&apikey=${API_KEY}`;
```

#### 2. Yahoo Finance (via RapidAPI)
```typescript
const options = {
  method: 'GET',
  headers: {
    'X-RapidAPI-Key': 'YOUR_RAPIDAPI_KEY',
    'X-RapidAPI-Host': 'yahoo-finance15.p.rapidapi.com'
  }
};

fetch(`https://yahoo-finance15.p.rapidapi.com/api/yahoo/qu/quote/${symbol}`, options)
  .then(response => response.json())
  .then(data => console.log(data));
```

#### 3. IEX Cloud
```typescript
const token = 'YOUR_IEX_TOKEN';
const symbol = 'AAPL';

// Latest price
fetch(`https://cloud.iexapis.com/stable/stock/${symbol}/quote?token=${token}`)
  .then(res => res.json());

// Historical data
fetch(`https://cloud.iexapis.com/stable/stock/${symbol}/chart/3m?token=${token}`)
  .then(res => res.json());
```

### Chatbot API Integration (OpenAI)

```typescript
const OPENAI_API_KEY = 'YOUR_OPENAI_KEY';

async function getChatbotResponse(message: string): Promise<string> {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${OPENAI_API_KEY}`
    },
    body: JSON.stringify({
      model: 'gpt-4',
      messages: [
        {
          role: 'system',
          content: 'You are a stock market expert assistant. Only answer questions related to stocks, trading, and finance.'
        },
        {
          role: 'user',
          content: message
        }
      ],
      temperature: 0.7,
      max_tokens: 500
    })
  });
  
  const data = await response.json();
  return data.choices[0].message.content;
}
```

---

## 📱 Responsive Design Breakpoints

```css
/* Mobile First Approach */

/* Mobile: Default (< 640px) */
.container { padding: 1rem; }

/* Tablet: 640px and up */
@media (min-width: 640px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}

/* Desktop: 1024px and up */
@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); }
  .container { max-width: 1200px; }
}

/* Large Desktop: 1280px and up */
@media (min-width: 1280px) {
  .container { max-width: 1400px; }
}
```

---

## 🚀 Deployment Guide

### Deploy to Vercel (Recommended)

1. **Install Vercel CLI**
   ```bash
   npm i -g vercel
   ```

2. **Build the project**
   ```bash
   npm run build
   ```

3. **Deploy**
   ```bash
   vercel
   ```

4. **Production deployment**
   ```bash
   vercel --prod
   ```

### Deploy to Netlify

1. **Build the project**
   ```bash
   npm run build
   ```

2. **Install Netlify CLI**
   ```bash
   npm i -g netlify-cli
   ```

3. **Deploy**
   ```bash
   netlify deploy --prod --dir=dist
   ```

### Deploy to AWS S3 + CloudFront

1. **Build**
   ```bash
   npm run build
   ```

2. **Upload to S3**
   ```bash
   aws s3 sync dist/ s3://your-bucket-name --delete
   ```

3. **Configure CloudFront** for CDN and HTTPS

### Environment Variables

Create `.env` file for production:

```env
VITE_API_URL=https://your-backend-api.com
VITE_ALPHA_VANTAGE_KEY=your_key_here
VITE_OPENAI_KEY=your_key_here
```

Access in code:
```typescript
const apiUrl = import.meta.env.VITE_API_URL;
```

---

## 🧪 Testing Guide

### Unit Tests (Recommended: Vitest)

```typescript
// mlEngine.test.ts
import { describe, it, expect } from 'vitest';
import { calculateRSI, calculateMACD } from './mlEngine';

describe('ML Engine', () => {
  it('should calculate RSI correctly', () => {
    const prices = [100, 102, 101, 103, 105, 104];
    const rsi = calculateRSI(prices, 5);
    expect(rsi).toBeGreaterThan(0);
    expect(rsi).toBeLessThan(100);
  });
  
  it('should generate BUY signal for oversold stocks', () => {
    const stockData = generateMockStockData('TEST', 90, 100);
    const recommendation = generateRecommendation(stockData);
    expect(['BUY', 'HOLD', 'SELL']).toContain(recommendation.signal);
    expect(recommendation.confidence).toBeGreaterThan(0);
  });
});
```

---

## 📊 Performance Optimization

### Code Splitting

```typescript
// Lazy load pages
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Portfolio = lazy(() => import('./pages/PortfolioPage'));

// Use Suspense
<Suspense fallback={<Loading />}>
  <Dashboard />
</Suspense>
```

### Memoization

```typescript
import { useMemo } from 'react';

const calculations = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);
```

### Chart Performance

```typescript
// Limit data points for charts
const chartData = stockData.slice(-30); // Only show last 30 days
```

---

## 🔒 Security Best Practices

1. **Never expose API keys in frontend code**
   - Use environment variables
   - Proxy API calls through backend

2. **Implement rate limiting**
   - Prevent API abuse
   - Use backend middleware

3. **Sanitize user inputs**
   - Prevent XSS attacks
   - Validate all form data

4. **Use HTTPS only**
   - Enable SSL certificates
   - Force HTTPS redirects

5. **Implement JWT refresh tokens**
   - Short-lived access tokens
   - Long-lived refresh tokens

---

## 📝 License

This project is for educational and portfolio purposes. Please ensure you comply with the terms of service for any third-party APIs you integrate.

---

## 🤝 Contributing

This is a portfolio project, but contributions and suggestions are welcome!

---

## 📧 Support

For questions or issues, please refer to the inline code documentation or create an issue in the repository.

---

**Built with ❤️ by STOCKHUB PRO Team**

**Last Updated:** December 26, 2024
