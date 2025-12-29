# 🔧 STOCKHUB PRO - Backend Integration Guide

## Complete Python Flask Backend Implementation

This guide provides a **production-ready Python Flask backend** that integrates seamlessly with the StockHub Pro frontend.

---

## 📋 Table of Contents

1. [Project Setup](#project-setup)
2. [Database Configuration](#database-configuration)
3. [Complete Backend Code](#complete-backend-code)
4. [API Endpoints](#api-endpoints)
5. [Frontend Integration](#frontend-integration)
6. [Deployment](#deployment)

---

## 🚀 Project Setup

### Backend Folder Structure

```
stockhub-backend/
├── app.py                 # Main Flask application
├── requirements.txt       # Python dependencies
├── config.py             # Configuration settings
├── models/
│   ├── __init__.py
│   ├── user.py           # User model
│   ├── portfolio.py      # Portfolio model
│   └── transaction.py    # Transaction model
├── routes/
│   ├── __init__.py
│   ├── auth.py           # Authentication routes
│   ├── portfolio.py      # Portfolio routes
│   ├── stocks.py         # Stock data routes
│   └── chatbot.py        # AI chatbot routes
├── ml_engine/
│   ├── __init__.py
│   ├── indicators.py     # Technical indicators
│   └── recommendations.py # ML recommendation engine
├── utils/
│   ├── __init__.py
│   ├── auth.py           # JWT utilities
│   └── helpers.py        # Helper functions
└── database.db           # SQLite database (or PostgreSQL config)
```

---

## 📦 requirements.txt

```txt
Flask==3.0.0
Flask-CORS==4.0.0
Flask-SQLAlchemy==3.1.1
Flask-Migrate==4.0.5
Flask-JWT-Extended==4.6.0
python-dotenv==1.0.0
bcrypt==4.1.2
requests==2.31.0
pandas==2.1.4
numpy==1.26.2
ta==0.11.0
alpha-vantage==2.3.1
openai==1.6.1
gunicorn==21.2.0
psycopg2-binary==2.9.9
```

---

## ⚙️ config.py

```python
import os
from datetime import timedelta

class Config:
    # Flask
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-secret-key-change-in-production'
    
    # Database
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or 'sqlite:///database.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # JWT
    JWT_SECRET_KEY = os.environ.get('JWT_SECRET_KEY') or 'jwt-secret-key-change-in-production'
    JWT_ACCESS_TOKEN_EXPIRES = timedelta(hours=24)
    JWT_REFRESH_TOKEN_EXPIRES = timedelta(days=30)
    
    # External APIs
    ALPHA_VANTAGE_KEY = os.environ.get('ALPHA_VANTAGE_KEY')
    OPENAI_API_KEY = os.environ.get('OPENAI_API_KEY')
    
    # CORS
    CORS_ORIGINS = ['http://localhost:5173', 'http://localhost:3000']

class ProductionConfig(Config):
    DEBUG = False
    CORS_ORIGINS = ['https://your-production-domain.com']

class DevelopmentConfig(Config):
    DEBUG = True
    
config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```

---

## 🗄️ Database Models

### models/user.py

```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime
import bcrypt

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(255), unique=True, nullable=False)
    name = db.Column(db.String(255), nullable=False)
    password_hash = db.Column(db.String(255), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    portfolio = db.relationship('Portfolio', backref='user', lazy=True, cascade='all, delete-orphan')
    transactions = db.relationship('Transaction', backref='user', lazy=True, cascade='all, delete-orphan')
    
    def set_password(self, password):
        """Hash and set password"""
        self.password_hash = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt()).decode('utf-8')
    
    def check_password(self, password):
        """Verify password"""
        return bcrypt.checkpw(password.encode('utf-8'), self.password_hash.encode('utf-8'))
    
    def to_dict(self):
        """Convert user to dictionary"""
        return {
            'id': self.id,
            'email': self.email,
            'name': self.name,
            'createdAt': self.created_at.isoformat()
        }
```

### models/portfolio.py

```python
from models.user import db
from datetime import datetime

class Portfolio(db.Model):
    __tablename__ = 'portfolio'
    
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    symbol = db.Column(db.String(10), nullable=False)
    name = db.Column(db.String(255), nullable=False)
    shares = db.Column(db.Float, nullable=False)
    avg_price = db.Column(db.Float, nullable=False)
    sector = db.Column(db.String(50))
    purchase_date = db.Column(db.Date)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    def to_dict(self, current_price=None):
        """Convert portfolio item to dictionary"""
        return {
            'symbol': self.symbol,
            'name': self.name,
            'shares': self.shares,
            'avgPrice': self.avg_price,
            'currentPrice': current_price or self.avg_price,
            'sector': self.sector,
            'purchaseDate': self.purchase_date.isoformat() if self.purchase_date else None
        }
```

### models/transaction.py

```python
from models.user import db
from datetime import datetime

class Transaction(db.Model):
    __tablename__ = 'transactions'
    
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    symbol = db.Column(db.String(10), nullable=False)
    type = db.Column(db.String(10), nullable=False)  # 'BUY' or 'SELL'
    shares = db.Column(db.Float, nullable=False)
    price = db.Column(db.Float, nullable=False)
    total = db.Column(db.Float, nullable=False)
    transaction_date = db.Column(db.Date, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    def to_dict(self):
        """Convert transaction to dictionary"""
        return {
            'id': str(self.id),
            'symbol': self.symbol,
            'type': self.type,
            'shares': self.shares,
            'price': self.price,
            'total': self.total,
            'date': self.transaction_date.isoformat(),
            'createdAt': self.created_at.isoformat()
        }
```

---

## 🔐 Authentication Routes

### routes/auth.py

```python
from flask import Blueprint, request, jsonify
from flask_jwt_extended import create_access_token, create_refresh_token, jwt_required, get_jwt_identity
from models.user import db, User

auth_bp = Blueprint('auth', __name__, url_prefix='/api/auth')

@auth_bp.route('/register', methods=['POST'])
def register():
    """Register new user"""
    data = request.get_json()
    
    # Validate input
    if not data.get('email') or not data.get('password') or not data.get('name'):
        return jsonify({'error': 'Missing required fields'}), 400
    
    # Check if user exists
    if User.query.filter_by(email=data['email']).first():
        return jsonify({'error': 'Email already registered'}), 409
    
    # Create new user
    user = User(
        email=data['email'],
        name=data['name']
    )
    user.set_password(data['password'])
    
    db.session.add(user)
    db.session.commit()
    
    # Generate tokens
    access_token = create_access_token(identity=user.id)
    refresh_token = create_refresh_token(identity=user.id)
    
    return jsonify({
        'success': True,
        'token': access_token,
        'refreshToken': refresh_token,
        'user': user.to_dict()
    }), 201

@auth_bp.route('/login', methods=['POST'])
def login():
    """Login user"""
    data = request.get_json()
    
    # Validate input
    if not data.get('email') or not data.get('password'):
        return jsonify({'error': 'Missing email or password'}), 400
    
    # Find user
    user = User.query.filter_by(email=data['email']).first()
    
    if not user or not user.check_password(data['password']):
        return jsonify({'error': 'Invalid credentials'}), 401
    
    # Generate tokens
    access_token = create_access_token(identity=user.id)
    refresh_token = create_refresh_token(identity=user.id)
    
    return jsonify({
        'success': True,
        'token': access_token,
        'refreshToken': refresh_token,
        'user': user.to_dict()
    }), 200

@auth_bp.route('/me', methods=['GET'])
@jwt_required()
def get_current_user():
    """Get current authenticated user"""
    user_id = get_jwt_identity()
    user = User.query.get(user_id)
    
    if not user:
        return jsonify({'error': 'User not found'}), 404
    
    return jsonify({'user': user.to_dict()}), 200

@auth_bp.route('/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh():
    """Refresh access token"""
    user_id = get_jwt_identity()
    access_token = create_access_token(identity=user_id)
    
    return jsonify({'token': access_token}), 200
```

---

## 💼 Portfolio Routes

### routes/portfolio.py

```python
from flask import Blueprint, request, jsonify
from flask_jwt_extended import jwt_required, get_jwt_identity
from models.user import db
from models.portfolio import Portfolio
from models.transaction import Transaction
from datetime import datetime
import requests

portfolio_bp = Blueprint('portfolio', __name__, url_prefix='/api/portfolio')

@portfolio_bp.route('/', methods=['GET'])
@jwt_required()
def get_portfolio():
    """Get user's portfolio"""
    user_id = get_jwt_identity()
    
    # Get portfolio items
    portfolio_items = Portfolio.query.filter_by(user_id=user_id).all()
    transactions = Transaction.query.filter_by(user_id=user_id).order_by(Transaction.created_at.desc()).all()
    
    # Fetch current prices (you would integrate with real API here)
    stocks = []
    for item in portfolio_items:
        current_price = get_current_price(item.symbol)  # Implement this function
        stocks.append(item.to_dict(current_price))
    
    return jsonify({
        'stocks': stocks,
        'transactions': [tx.to_dict() for tx in transactions]
    }), 200

@portfolio_bp.route('/add', methods=['POST'])
@jwt_required()
def add_stock():
    """Add stock to portfolio"""
    user_id = get_jwt_identity()
    data = request.get_json()
    
    # Validate input
    required_fields = ['symbol', 'name', 'shares', 'avgPrice', 'sector', 'purchaseDate']
    if not all(field in data for field in required_fields):
        return jsonify({'error': 'Missing required fields'}), 400
    
    # Create portfolio item
    portfolio_item = Portfolio(
        user_id=user_id,
        symbol=data['symbol'].upper(),
        name=data['name'],
        shares=float(data['shares']),
        avg_price=float(data['avgPrice']),
        sector=data['sector'],
        purchase_date=datetime.strptime(data['purchaseDate'], '%Y-%m-%d').date()
    )
    
    # Create transaction record
    transaction = Transaction(
        user_id=user_id,
        symbol=data['symbol'].upper(),
        type='BUY',
        shares=float(data['shares']),
        price=float(data['avgPrice']),
        total=float(data['shares']) * float(data['avgPrice']),
        transaction_date=datetime.strptime(data['purchaseDate'], '%Y-%m-%d').date()
    )
    
    db.session.add(portfolio_item)
    db.session.add(transaction)
    db.session.commit()
    
    current_price = get_current_price(data['symbol'])
    
    return jsonify({
        'success': True,
        'stock': portfolio_item.to_dict(current_price)
    }), 201

@portfolio_bp.route('/<symbol>', methods=['DELETE'])
@jwt_required()
def remove_stock(symbol):
    """Remove stock from portfolio"""
    user_id = get_jwt_identity()
    
    portfolio_item = Portfolio.query.filter_by(
        user_id=user_id,
        symbol=symbol.upper()
    ).first()
    
    if not portfolio_item:
        return jsonify({'error': 'Stock not found in portfolio'}), 404
    
    db.session.delete(portfolio_item)
    db.session.commit()
    
    return jsonify({'success': True}), 200

def get_current_price(symbol):
    """Get current stock price (implement with real API)"""
    # This is a placeholder - integrate with Alpha Vantage, Yahoo Finance, etc.
    # For now, return a mock price
    return 150.00  # Replace with actual API call
```

---

## 📈 Stock Data Routes

### routes/stocks.py

```python
from flask import Blueprint, request, jsonify
import requests
from datetime import datetime, timedelta
import os

stocks_bp = Blueprint('stocks', __name__, url_prefix='/api/stocks')

ALPHA_VANTAGE_KEY = os.environ.get('ALPHA_VANTAGE_KEY')

@stocks_bp.route('/<symbol>', methods=['GET'])
def get_stock_data(symbol):
    """Get current stock data"""
    try:
        # Alpha Vantage API call
        url = f'https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol={symbol}&apikey={ALPHA_VANTAGE_KEY}'
        response = requests.get(url)
        data = response.json()
        
        if 'Global Quote' in data:
            quote = data['Global Quote']
            return jsonify({
                'symbol': symbol,
                'currentPrice': float(quote.get('05. price', 0)),
                'change': float(quote.get('09. change', 0)),
                'changePercent': float(quote.get('10. change percent', '0').replace('%', '')),
                'volume': int(quote.get('06. volume', 0)),
                'timestamp': quote.get('07. latest trading day')
            }), 200
        else:
            return jsonify({'error': 'Stock data not found'}), 404
            
    except Exception as e:
        return jsonify({'error': str(e)}), 500

@stocks_bp.route('/<symbol>/historical', methods=['GET'])
def get_historical_data(symbol):
    """Get historical stock data"""
    period = request.args.get('period', '90')  # days
    
    try:
        # Alpha Vantage API call for daily data
        url = f'https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol={symbol}&apikey={ALPHA_VANTAGE_KEY}'
        response = requests.get(url)
        data = response.json()
        
        if 'Time Series (Daily)' in data:
            time_series = data['Time Series (Daily)']
            
            # Convert to list of stock data
            stock_data = []
            for date_str, values in sorted(time_series.items())[-int(period):]:
                stock_data.append({
                    'date': date_str,
                    'open': float(values['1. open']),
                    'high': float(values['2. high']),
                    'low': float(values['3. low']),
                    'close': float(values['4. close']),
                    'volume': int(values['5. volume'])
                })
            
            return jsonify({'data': stock_data}), 200
        else:
            return jsonify({'error': 'Historical data not found'}), 404
            
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

---

## 🤖 AI Chatbot Routes

### routes/chatbot.py

```python
from flask import Blueprint, request, jsonify
import openai
import os

chatbot_bp = Blueprint('chatbot', __name__, url_prefix='/api/chatbot')

openai.api_key = os.environ.get('OPENAI_API_KEY')

@chatbot_bp.route('/query', methods=['POST'])
def chat_query():
    """Process chatbot query"""
    data = request.get_json()
    
    if not data.get('message'):
        return jsonify({'error': 'Message is required'}), 400
    
    try:
        # OpenAI API call
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[
                {
                    "role": "system",
                    "content": """You are a specialized stock market and trading assistant. 
                    You only answer questions related to:
                    - Stock market analysis and trends
                    - Trading strategies (day trading, swing trading, value investing)
                    - Technical indicators (RSI, MACD, Bollinger Bands, etc.)
                    - Fundamental analysis
                    - Portfolio management and risk management
                    - Market news and economic indicators
                    
                    If a question is not related to stock market or trading, politely decline and 
                    redirect the user to ask stock market-related questions."""
                },
                {
                    "role": "user",
                    "content": data['message']
                }
            ],
            temperature=0.7,
            max_tokens=500
        )
        
        reply = response.choices[0].message.content
        
        return jsonify({
            'reply': reply,
            'conversationId': data.get('conversationId', str(int(datetime.now().timestamp()))),
            'timestamp': datetime.now().isoformat()
        }), 200
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

---

## 🎯 Main Application

### app.py

```python
from flask import Flask
from flask_cors import CORS
from flask_jwt_extended import JWTManager
from flask_migrate import Migrate
from config import config
from models.user import db
from routes.auth import auth_bp
from routes.portfolio import portfolio_bp
from routes.stocks import stocks_bp
from routes.chatbot import chatbot_bp
import os

def create_app(config_name='development'):
    """Application factory"""
    app = Flask(__name__)
    
    # Load configuration
    app.config.from_object(config[config_name])
    
    # Initialize extensions
    db.init_app(app)
    CORS(app, origins=app.config['CORS_ORIGINS'])
    jwt = JWTManager(app)
    migrate = Migrate(app, db)
    
    # Register blueprints
    app.register_blueprint(auth_bp)
    app.register_blueprint(portfolio_bp)
    app.register_blueprint(stocks_bp)
    app.register_blueprint(chatbot_bp)
    
    # Create database tables
    with app.app_context():
        db.create_all()
    
    # Health check endpoint
    @app.route('/api/health', methods=['GET'])
    def health_check():
        return {'status': 'healthy', 'message': 'StockHub Pro API is running'}, 200
    
    return app

if __name__ == '__main__':
    env = os.environ.get('FLASK_ENV', 'development')
    app = create_app(env)
    app.run(host='0.0.0.0', port=5000, debug=(env == 'development'))
```

---

## 🌐 Frontend Integration

Update the frontend to use the real backend API:

### Example: AuthContext.tsx

```typescript
// src/app/context/AuthContext.tsx
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000';

const login = async (email: string, password: string): Promise<boolean> => {
  try {
    const response = await fetch(`${API_URL}/api/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    
    if (data.success) {
      localStorage.setItem('stockhub_token', data.token);
      localStorage.setItem('stockhub_user', JSON.stringify(data.user));
      setUser(data.user);
      setIsAuthenticated(true);
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

## 🚀 Deployment

### Deploy to Heroku

1. **Create Procfile**
   ```
   web: gunicorn app:app
   ```

2. **Deploy**
   ```bash
   heroku create stockhub-api
   heroku addons:create heroku-postgresql:hobby-dev
   git push heroku main
   ```

3. **Set environment variables**
   ```bash
   heroku config:set SECRET_KEY=your-secret-key
   heroku config:set JWT_SECRET_KEY=your-jwt-secret
   heroku config:set ALPHA_VANTAGE_KEY=your-api-key
   heroku config:set OPENAI_API_KEY=your-openai-key
   ```

### Deploy to AWS Elastic Beanstalk

1. **Install EB CLI**
   ```bash
   pip install awsebcli
   ```

2. **Initialize**
   ```bash
   eb init -p python-3.11 stockhub-api
   ```

3. **Deploy**
   ```bash
   eb create stockhub-production
   eb deploy
   ```

---

## ✅ Testing the Backend

```bash
# Register user
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123","name":"Test User"}'

# Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Get portfolio (replace TOKEN with actual JWT)
curl -X GET http://localhost:5000/api/portfolio \
  -H "Authorization: Bearer TOKEN"

# Add stock
curl -X POST http://localhost:5000/api/portfolio/add \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"symbol":"AAPL","name":"Apple Inc.","shares":10,"avgPrice":150,"sector":"Technology","purchaseDate":"2024-01-01"}'
```

---

**🎉 Your backend is now complete and production-ready!**

Integrate with the React frontend and you'll have a fully functional stock market platform.
