# 🔌 API Integration Examples - STOCKHUB PRO

This guide shows you **exactly how to connect the frontend to real backend APIs**.

---

## 📋 Table of Contents

1. [Authentication API](#authentication-api)
2. [Portfolio API](#portfolio-api)
3. [Stock Data API](#stock-data-api)
4. [ML Prediction API](#ml-prediction-api)
5. [Chatbot API](#chatbot-api)
6. [Real-Time Data](#real-time-data)
7. [Error Handling](#error-handling)
8. [Complete Integration](#complete-integration)

---

## 🔐 Authentication API

### Backend Endpoint Setup (Node.js + Express)

```javascript
// backend/routes/auth.js
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const router = express.Router();

// REGISTER
router.post('/register', async (req, res) => {
  try {
    const { name, email, password } = req.body;
    
    // Validation
    if (!name || !email || !password) {
      return res.status(400).json({ message: 'All fields required' });
    }
    
    // Check if user exists
    const existing = await User.findOne({ email });
    if (existing) {
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
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    res.status(201).json({
      success: true,
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        createdAt: user.createdAt
      }
    });
  } catch (error) {
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});

// LOGIN
router.post('/login', async (req, res) => {
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
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    res.json({
      success: true,
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        createdAt: user.createdAt
      }
    });
  } catch (error) {
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});

module.exports = router;
```

### Frontend Integration

Replace in `/src/app/context/AuthContext.tsx`:

```typescript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000';

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);

  // Load user from localStorage on mount
  useEffect(() => {
    const storedToken = localStorage.getItem('stockhub_token');
    const storedUser = localStorage.getItem('stockhub_user');
    
    if (storedToken && storedUser) {
      setToken(storedToken);
      setUser(JSON.parse(storedUser));
    }
  }, []);

  // REGISTER
  const register = async (name: string, email: string, password: string): Promise<boolean> => {
    try {
      const response = await fetch(`${API_URL}/api/auth/register`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ name, email, password }),
      });

      if (!response.ok) {
        const error = await response.json();
        console.error('Registration failed:', error.message);
        return false;
      }

      const data = await response.json();

      if (data.success && data.token) {
        setUser(data.user);
        setToken(data.token);
        
        // Store in localStorage
        localStorage.setItem('stockhub_token', data.token);
        localStorage.setItem('stockhub_user', JSON.stringify(data.user));

        return true;
      }

      return false;
    } catch (error) {
      console.error('Registration error:', error);
      return false;
    }
  };

  // LOGIN
  const login = async (email: string, password: string): Promise<boolean> => {
    try {
      const response = await fetch(`${API_URL}/api/auth/login`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        const error = await response.json();
        console.error('Login failed:', error.message);
        return false;
      }

      const data = await response.json();

      if (data.success && data.token) {
        setUser(data.user);
        setToken(data.token);
        
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

  // LOGOUT
  const logout = () => {
    setUser(null);
    setToken(null);
    localStorage.removeItem('stockhub_token');
    localStorage.removeItem('stockhub_user');
  };

  return (
    <AuthContext.Provider value={{ user, isAuthenticated: !!user, login, register, logout, token }}>
      {children}
    </AuthContext.Provider>
  );
}
```

---

## 💼 Portfolio API

### Backend Endpoint Setup

```javascript
// backend/routes/portfolio.js
const express = require('express');
const Portfolio = require('../models/Portfolio');
const { authMiddleware } = require('../middleware/auth');

const router = express.Router();

// GET user portfolio
router.get('/', authMiddleware, async (req, res) => {
  try {
    let portfolio = await Portfolio.findOne({ userId: req.userId });
    
    if (!portfolio) {
      portfolio = new Portfolio({ userId: req.userId, stocks: [] });
      await portfolio.save();
    }
    
    res.json({ success: true, portfolio: portfolio.stocks });
  } catch (error) {
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});

// ADD stock to portfolio
router.post('/add', authMiddleware, async (req, res) => {
  try {
    const { symbol, name, quantity, buyPrice, currentPrice } = req.body;
    
    let portfolio = await Portfolio.findOne({ userId: req.userId });
    
    if (!portfolio) {
      portfolio = new Portfolio({ userId: req.userId, stocks: [] });
    }
    
    portfolio.stocks.push({
      symbol,
      name,
      quantity,
      buyPrice,
      currentPrice,
      addedAt: new Date()
    });
    
    await portfolio.save();
    
    res.json({ success: true, portfolio: portfolio.stocks });
  } catch (error) {
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});

// REMOVE stock from portfolio
router.delete('/:stockId', authMiddleware, async (req, res) => {
  try {
    const portfolio = await Portfolio.findOne({ userId: req.userId });
    
    if (!portfolio) {
      return res.status(404).json({ message: 'Portfolio not found' });
    }
    
    portfolio.stocks = portfolio.stocks.filter(
      stock => stock._id.toString() !== req.params.stockId
    );
    
    await portfolio.save();
    
    res.json({ success: true, portfolio: portfolio.stocks });
  } catch (error) {
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});

module.exports = router;
```

### Frontend Integration

Create `/src/app/services/portfolioService.ts`:

```typescript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000';

export interface PortfolioStock {
  id: string;
  symbol: string;
  name: string;
  quantity: number;
  buyPrice: number;
  currentPrice: number;
  addedAt: string;
}

export class PortfolioService {
  private static getHeaders(token: string) {
    return {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    };
  }

  static async getPortfolio(token: string): Promise<PortfolioStock[]> {
    try {
      const response = await fetch(`${API_URL}/api/portfolio`, {
        method: 'GET',
        headers: this.getHeaders(token)
      });

      if (!response.ok) {
        throw new Error('Failed to fetch portfolio');
      }

      const data = await response.json();
      return data.portfolio;
    } catch (error) {
      console.error('Get portfolio error:', error);
      throw error;
    }
  }

  static async addStock(
    token: string,
    stock: Omit<PortfolioStock, 'id' | 'addedAt'>
  ): Promise<PortfolioStock[]> {
    try {
      const response = await fetch(`${API_URL}/api/portfolio/add`, {
        method: 'POST',
        headers: this.getHeaders(token),
        body: JSON.stringify(stock)
      });

      if (!response.ok) {
        throw new Error('Failed to add stock');
      }

      const data = await response.json();
      return data.portfolio;
    } catch (error) {
      console.error('Add stock error:', error);
      throw error;
    }
  }

  static async removeStock(token: string, stockId: string): Promise<PortfolioStock[]> {
    try {
      const response = await fetch(`${API_URL}/api/portfolio/${stockId}`, {
        method: 'DELETE',
        headers: this.getHeaders(token)
      });

      if (!response.ok) {
        throw new Error('Failed to remove stock');
      }

      const data = await response.json();
      return data.portfolio;
    } catch (error) {
      console.error('Remove stock error:', error);
      throw error;
    }
  }
}
```

Update `/src/app/context/PortfolioContext.tsx`:

```typescript
import { PortfolioService } from '../services/portfolioService';
import { useAuth } from './AuthContext';

export function PortfolioProvider({ children }: { children: ReactNode }) {
  const { token } = useAuth();
  const [portfolio, setPortfolio] = useState<PortfolioStock[]>([]);
  const [loading, setLoading] = useState(false);

  // Load portfolio from API
  useEffect(() => {
    if (token) {
      loadPortfolio();
    }
  }, [token]);

  const loadPortfolio = async () => {
    if (!token) return;
    
    setLoading(true);
    try {
      const data = await PortfolioService.getPortfolio(token);
      setPortfolio(data);
    } catch (error) {
      console.error('Failed to load portfolio:', error);
    } finally {
      setLoading(false);
    }
  };

  const addToPortfolio = async (stock: Omit<PortfolioStock, 'id' | 'addedAt'>) => {
    if (!token) return;
    
    setLoading(true);
    try {
      const data = await PortfolioService.addStock(token, stock);
      setPortfolio(data);
    } catch (error) {
      console.error('Failed to add stock:', error);
    } finally {
      setLoading(false);
    }
  };

  const removeFromPortfolio = async (id: string) => {
    if (!token) return;
    
    setLoading(true);
    try {
      const data = await PortfolioService.removeStock(token, id);
      setPortfolio(data);
    } catch (error) {
      console.error('Failed to remove stock:', error);
    } finally {
      setLoading(false);
    }
  };

  // ... rest of context implementation
}
```

---

## 📊 Stock Data API

### Using Yahoo Finance API (Python Backend)

```python
# ml-backend/app/stock_data.py
import yfinance as yf
from datetime import datetime, timedelta

def get_stock_data(symbol: str, period: str = '1y'):
    """
    Fetch real-time stock data from Yahoo Finance
    """
    try:
        ticker = yf.Ticker(symbol)
        
        # Get historical data
        hist = ticker.history(period=period)
        
        # Get current info
        info = ticker.info
        
        return {
            'symbol': symbol,
            'name': info.get('longName', symbol),
            'current_price': info.get('currentPrice', hist['Close'].iloc[-1]),
            'previous_close': info.get('previousClose'),
            'open': info.get('open'),
            'day_high': info.get('dayHigh'),
            'day_low': info.get('dayLow'),
            'volume': info.get('volume'),
            'market_cap': info.get('marketCap'),
            'pe_ratio': info.get('trailingPE'),
            'dividend_yield': info.get('dividendYield'),
            'historical_data': hist.to_dict('records')
        }
    except Exception as e:
        raise Exception(f"Error fetching data for {symbol}: {str(e)}")

def search_stocks(query: str):
    """
    Search for stocks by symbol or name
    """
    # In production, use a proper stock search API
    # For demo, return common matches
    common_stocks = {
        'AAPL': 'Apple Inc.',
        'MSFT': 'Microsoft Corporation',
        'GOOGL': 'Alphabet Inc.',
        'AMZN': 'Amazon.com Inc.',
        'TSLA': 'Tesla Inc.',
        'NVDA': 'NVIDIA Corporation',
        'META': 'Meta Platforms Inc.',
        'NFLX': 'Netflix Inc.'
    }
    
    query = query.upper()
    results = []
    
    for symbol, name in common_stocks.items():
        if query in symbol or query in name.upper():
            results.append({'symbol': symbol, 'name': name})
    
    return results
```

### Frontend Integration

Create `/src/app/services/stockService.ts`:

```typescript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000';

export interface StockData {
  symbol: string;
  name: string;
  current_price: number;
  previous_close: number;
  open: number;
  day_high: number;
  day_low: number;
  volume: number;
  market_cap: number;
  pe_ratio: number;
}

export class StockService {
  static async getStockData(symbol: string): Promise<StockData> {
    try {
      const response = await fetch(`${API_URL}/api/stocks/${symbol}`);
      
      if (!response.ok) {
        throw new Error('Stock not found');
      }
      
      return await response.json();
    } catch (error) {
      console.error('Get stock data error:', error);
      throw error;
    }
  }

  static async searchStocks(query: string): Promise<Array<{ symbol: string; name: string }>> {
    try {
      const response = await fetch(`${API_URL}/api/stocks/search?q=${encodeURIComponent(query)}`);
      
      if (!response.ok) {
        throw new Error('Search failed');
      }
      
      return await response.json();
    } catch (error) {
      console.error('Search stocks error:', error);
      throw error;
    }
  }
}
```

---

## 🤖 ML Prediction API

### Python FastAPI Backend

```python
# ml-backend/app/main.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import uvicorn
from model import analyze_stock
from stock_data import get_stock_data

app = FastAPI(title="StockHub ML API")

class AnalyzeRequest(BaseModel):
    symbol: str

@app.post("/api/analyze")
async def analyze(request: AnalyzeRequest):
    """
    Analyze stock and return recommendation
    """
    try:
        # Fetch real stock data
        stock_data = get_stock_data(request.symbol)
        
        # Run ML analysis
        analysis = analyze_stock(request.symbol, stock_data)
        
        return {
            'success': True,
            'analysis': analysis
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/api/health")
def health_check():
    return {"status": "healthy", "service": "StockHub ML API"}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Frontend Integration

Update `/src/app/utils/mlEngine.ts`:

```typescript
const ML_API_URL = import.meta.env.VITE_ML_API_URL || 'http://localhost:8000';

export async function analyzeStock(symbol: string): Promise<StockAnalysis> {
  try {
    const response = await fetch(`${ML_API_URL}/api/analyze`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ symbol })
    });

    if (!response.ok) {
      throw new Error('Analysis failed');
    }

    const data = await response.json();
    return data.analysis;
  } catch (error) {
    console.error('ML analysis error:', error);
    // Fallback to local calculation if API fails
    return fallbackAnalysis(symbol);
  }
}

// Keep fallback for offline mode
function fallbackAnalysis(symbol: string): StockAnalysis {
  // Your existing mock analysis code
  // ...
}
```

---

## 💬 Chatbot API

### Python Backend (Using OpenAI or Custom NLP)

```python
# ml-backend/app/chatbot.py
from transformers import pipeline
import openai  # If using OpenAI
import os

# Option 1: Use OpenAI GPT
def generate_response_openai(message: str, history: list = []):
    """
    Generate response using OpenAI GPT
    """
    openai.api_key = os.getenv('OPENAI_API_KEY')
    
    messages = [
        {"role": "system", "content": "You are a helpful stock market assistant for StockHub Pro. Help users understand stocks, technical indicators, and platform features."}
    ]
    
    for msg in history:
        messages.append({"role": msg['role'], "content": msg['content']})
    
    messages.append({"role": "user", "content": message})
    
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=messages,
        temperature=0.7,
        max_tokens=500
    )
    
    return response.choices[0].message.content

# Option 2: Use open-source model
nlp = pipeline("text-generation", model="gpt2")

def generate_response_local(message: str):
    """
    Generate response using local model
    """
    # Your existing knowledge base logic
    # ...
    pass

# FastAPI endpoint
from fastapi import FastAPI
app = FastAPI()

@app.post("/api/chat")
async def chat(request: dict):
    message = request.get('message')
    history = request.get('history', [])
    
    try:
        response = generate_response_openai(message, history)
        return {'success': True, 'response': response}
    except Exception as e:
        # Fallback to local
        response = generate_response_local(message)
        return {'success': True, 'response': response}
```

### Frontend Integration

Update `/src/app/utils/chatbot.ts`:

```typescript
const ML_API_URL = import.meta.env.VITE_ML_API_URL || 'http://localhost:8000';

export async function generateChatbotResponse(
  userMessage: string,
  conversationHistory: ChatMessage[] = []
): Promise<string> {
  try {
    const response = await fetch(`${ML_API_URL}/api/chat`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: userMessage,
        history: conversationHistory.map(msg => ({
          role: msg.role,
          content: msg.content
        }))
      })
    });

    if (!response.ok) {
      throw new Error('Chatbot API failed');
    }

    const data = await response.json();
    return data.response;
  } catch (error) {
    console.error('Chatbot error:', error);
    // Fallback to local logic
    return generateLocalResponse(userMessage);
  }
}

// Keep existing local response function as fallback
function generateLocalResponse(message: string): string {
  // Your existing chatbot logic
  // ...
}
```

---

## 🔄 Real-Time Data (WebSockets)

### Backend WebSocket Setup

```javascript
// backend/websocket.js
const WebSocket = require('ws');
const yahooFinance = require('yahoo-finance2');

function setupWebSocket(server) {
  const wss = new WebSocket.Server({ server });

  wss.on('connection', (ws) => {
    console.log('Client connected');
    
    // Subscribe to stock updates
    ws.on('message', async (message) => {
      const data = JSON.parse(message);
      
      if (data.type === 'SUBSCRIBE') {
        const symbols = data.symbols;
        
        // Send updates every 5 seconds
        const interval = setInterval(async () => {
          const updates = await Promise.all(
            symbols.map(async (symbol) => {
              const quote = await yahooFinance.quote(symbol);
              return {
                symbol,
                price: quote.regularMarketPrice,
                change: quote.regularMarketChange,
                changePercent: quote.regularMarketChangePercent
              };
            })
          );
          
          ws.send(JSON.stringify({ type: 'PRICE_UPDATE', data: updates }));
        }, 5000);
        
        // Clean up on disconnect
        ws.on('close', () => {
          clearInterval(interval);
          console.log('Client disconnected');
        });
      }
    });
  });
}

module.exports = { setupWebSocket };
```

### Frontend WebSocket Integration

```typescript
// src/app/hooks/useStockUpdates.ts
import { useEffect, useState } from 'react';

interface StockUpdate {
  symbol: string;
  price: number;
  change: number;
  changePercent: number;
}

export function useStockUpdates(symbols: string[]) {
  const [updates, setUpdates] = useState<Record<string, StockUpdate>>({});
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    const ws = new WebSocket('ws://localhost:5000');

    ws.onopen = () => {
      console.log('WebSocket connected');
      setConnected(true);
      
      // Subscribe to symbols
      ws.send(JSON.stringify({
        type: 'SUBSCRIBE',
        symbols
      }));
    };

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'PRICE_UPDATE') {
        const updatesMap: Record<string, StockUpdate> = {};
        data.data.forEach((update: StockUpdate) => {
          updatesMap[update.symbol] = update;
        });
        setUpdates(updatesMap);
      }
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
      setConnected(false);
    };

    ws.onclose = () => {
      console.log('WebSocket disconnected');
      setConnected(false);
    };

    return () => {
      ws.close();
    };
  }, [symbols]);

  return { updates, connected };
}
```

---

## ⚠️ Error Handling

### Centralized Error Handler

```typescript
// src/app/utils/errorHandler.ts
export class APIError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public details?: any
  ) {
    super(message);
    this.name = 'APIError';
  }
}

export async function handleAPIResponse<T>(response: Response): Promise<T> {
  if (!response.ok) {
    const error = await response.json().catch(() => ({ message: 'Unknown error' }));
    throw new APIError(response.status, error.message, error);
  }
  
  return await response.json();
}

export function handleError(error: unknown): string {
  if (error instanceof APIError) {
    switch (error.statusCode) {
      case 401:
        return 'Please login to continue';
      case 403:
        return 'You do not have permission';
      case 404:
        return 'Resource not found';
      case 500:
        return 'Server error. Please try again later';
      default:
        return error.message;
    }
  }
  
  if (error instanceof Error) {
    return error.message;
  }
  
  return 'An unexpected error occurred';
}
```

---

## ✅ Complete Integration Checklist

### Backend Setup
- [ ] Node.js Express server running
- [ ] MongoDB connected
- [ ] JWT authentication working
- [ ] CORS configured
- [ ] Environment variables set
- [ ] Error handling implemented
- [ ] Rate limiting added
- [ ] Python ML server running
- [ ] Stock data API connected

### Frontend Setup
- [ ] API URL configured in .env
- [ ] AuthContext updated with API calls
- [ ] PortfolioContext integrated
- [ ] ML engine calling backend
- [ ] Chatbot using API
- [ ] Error handling implemented
- [ ] Loading states added
- [ ] Token refresh logic
- [ ] WebSocket connection (if needed)

### Testing
- [ ] Register new user
- [ ] Login existing user
- [ ] Add stock to portfolio
- [ ] Remove stock from portfolio
- [ ] Get AI recommendation
- [ ] Chat with bot
- [ ] Real-time updates working
- [ ] Error scenarios handled

---

## 🚀 Quick Start Commands

```bash
# Terminal 1: Backend
cd backend
npm install
npm start

# Terminal 2: ML Backend
cd ml-backend
pip install -r requirements.txt
python app/main.py

# Terminal 3: Frontend
npm run dev
```

---

## 📝 Environment Variables

### Frontend `.env`
```env
VITE_API_URL=http://localhost:5000
VITE_ML_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:5000
```

### Backend `.env`
```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/stockhub
JWT_SECRET=your-secret-key-here
ML_API_URL=http://localhost:8000
```

### ML Backend `.env`
```env
PORT=8000
OPENAI_API_KEY=your-openai-key-here  # Optional
```

---

<div align="center">

**🎉 You're Ready to Integrate!**

Follow these examples and your StockHub Pro will be connected to real backends!

</div>
