# 🚀 STOCKHUB PRO - Deployment Checklist

Use this checklist to deploy your StockHub Pro platform to production.

---

## 📋 Pre-Deployment Checklist

### ✅ Code Review
- [ ] All TypeScript errors resolved
- [ ] No console.log statements in production code
- [ ] All TODO comments addressed
- [ ] Code formatted and linted
- [ ] All components tested manually
- [ ] Error handling implemented
- [ ] Loading states added

### ✅ Environment Setup
- [ ] Production environment variables configured
- [ ] API URLs updated for production
- [ ] Database connection strings secured
- [ ] JWT secrets generated (strong, random)
- [ ] API keys secured (not in code)
- [ ] CORS settings configured

### ✅ Security
- [ ] HTTPS enabled
- [ ] HTTP redirects to HTTPS
- [ ] Secure headers configured (helmet.js)
- [ ] Rate limiting implemented
- [ ] Input validation on backend
- [ ] SQL/NoSQL injection prevention
- [ ] XSS protection enabled
- [ ] CSRF tokens implemented
- [ ] Password hashing verified (bcrypt)
- [ ] JWT expiration set appropriately

### ✅ Performance
- [ ] Code splitting implemented
- [ ] Images optimized
- [ ] Lazy loading enabled
- [ ] Bundle size optimized
- [ ] Database queries optimized
- [ ] Caching implemented (Redis)
- [ ] CDN configured for static assets
- [ ] Compression enabled (gzip)

### ✅ Testing
- [ ] All features tested
- [ ] Cross-browser testing (Chrome, Firefox, Safari, Edge)
- [ ] Mobile responsive testing
- [ ] API endpoints tested
- [ ] Error scenarios handled
- [ ] Load testing performed
- [ ] Security testing done

---

## 🌐 Frontend Deployment

### Option 1: Vercel (Recommended)

#### Setup
```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Deploy
vercel
```

#### Configuration
Create `vercel.json`:
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "vite",
  "env": {
    "VITE_API_URL": "@api-url",
    "VITE_ML_API_URL": "@ml-api-url"
  }
}
```

#### Environment Variables (Vercel Dashboard)
```
VITE_API_URL=https://your-api.com
VITE_ML_API_URL=https://your-ml-api.com
```

#### Custom Domain
1. Go to Vercel dashboard
2. Settings → Domains
3. Add your domain
4. Update DNS records

---

### Option 2: Netlify

#### Setup
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Deploy
netlify deploy --prod
```

#### Configuration
Create `netlify.toml`:
```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[build.environment]
  VITE_API_URL = "https://your-api.com"
  VITE_ML_API_URL = "https://your-ml-api.com"
```

---

### Option 3: GitHub Pages

#### Setup
```bash
# Install gh-pages
npm install -D gh-pages

# Add to package.json scripts
"predeploy": "npm run build",
"deploy": "gh-pages -d dist"

# Deploy
npm run deploy
```

#### Configuration
Update `vite.config.ts`:
```typescript
export default defineConfig({
  base: '/your-repo-name/',
  // ... rest of config
});
```

---

## 🗄️ Backend Deployment

### Option 1: Railway (Recommended)

#### Setup
1. Create Railway account
2. New Project → Deploy from GitHub
3. Add MongoDB plugin
4. Set environment variables
5. Deploy

#### Environment Variables
```
PORT=5000
MONGODB_URI=<railway-mongodb-url>
JWT_SECRET=<your-secret>
ML_API_URL=<ml-api-url>
NODE_ENV=production
```

---

### Option 2: Heroku

#### Setup
```bash
# Login to Heroku
heroku login

# Create app
heroku create stockhub-api

# Add MongoDB
heroku addons:create mongolab

# Deploy
git push heroku main

# Set environment variables
heroku config:set JWT_SECRET=your-secret
heroku config:set ML_API_URL=your-ml-api
```

#### Procfile
```
web: node src/server.js
```

---

### Option 3: DigitalOcean

#### Setup
1. Create Droplet (Ubuntu 22.04)
2. SSH into server
3. Install Node.js and MongoDB
4. Clone repository
5. Install dependencies
6. Setup PM2
7. Configure Nginx

#### PM2 Setup
```bash
npm install -g pm2
pm2 start src/server.js --name stockhub-api
pm2 startup
pm2 save
```

#### Nginx Configuration
```nginx
server {
    listen 80;
    server_name api.stockhub.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 🤖 ML Backend Deployment

### Option 1: Google Cloud Run

#### Setup
```bash
# Build Docker image
docker build -t stockhub-ml .

# Tag image
docker tag stockhub-ml gcr.io/YOUR_PROJECT/stockhub-ml

# Push to GCR
docker push gcr.io/YOUR_PROJECT/stockhub-ml

# Deploy to Cloud Run
gcloud run deploy stockhub-ml \
  --image gcr.io/YOUR_PROJECT/stockhub-ml \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

#### Dockerfile
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### Option 2: AWS EC2

#### Setup
1. Launch EC2 instance (t2.medium)
2. Install Python 3.9+
3. Clone repository
4. Install dependencies
5. Setup systemd service

#### systemd Service
```ini
[Unit]
Description=StockHub ML API
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/stockhub-ml
ExecStart=/usr/bin/python3 -m uvicorn app.main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 💾 Database Deployment

### Option 1: MongoDB Atlas (Recommended)

#### Setup
1. Create MongoDB Atlas account
2. Create cluster (M0 Free tier)
3. Create database user
4. Whitelist IP addresses (or allow from anywhere for development)
5. Get connection string
6. Update backend environment variables

#### Connection String
```
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/stockhub?retryWrites=true&w=majority
```

---

### Option 2: Self-Hosted MongoDB

#### Setup on Ubuntu
```bash
# Import MongoDB GPG key
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

# Add MongoDB repository
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

# Update and install
sudo apt-get update
sudo apt-get install -y mongodb-org

# Start MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod

# Secure MongoDB
mongo
> use admin
> db.createUser({user: "admin", pwd: "password", roles: ["root"]})
```

---

## 🔒 SSL/TLS Certificate

### Using Let's Encrypt (Free)

```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d stockhub.com -d www.stockhub.com

# Auto-renewal
sudo certbot renew --dry-run
```

---

## 📊 Monitoring Setup

### Frontend Monitoring

#### Google Analytics
Add to `index.html`:
```html
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

#### Sentry (Error Tracking)
```bash
npm install @sentry/react
```

```typescript
import * as Sentry from "@sentry/react";

Sentry.init({
  dsn: "YOUR_SENTRY_DSN",
  integrations: [new Sentry.BrowserTracing()],
  tracesSampleRate: 1.0,
});
```

---

### Backend Monitoring

#### PM2 Monitoring
```bash
pm2 install pm2-logrotate
pm2 logs
pm2 monit
```

#### Node.js Performance
```javascript
const express = require('express');
const promBundle = require('express-prom-bundle');

const metricsMiddleware = promBundle({
  includeMethod: true,
  includePath: true
});

app.use(metricsMiddleware);
```

---

## 🧪 Post-Deployment Testing

### Checklist
- [ ] Homepage loads correctly
- [ ] Registration works
- [ ] Login works
- [ ] Dashboard displays data
- [ ] Portfolio management works
- [ ] AI recommendations loading
- [ ] Chatbot responding
- [ ] Charts rendering
- [ ] Mobile responsive
- [ ] All API calls working
- [ ] HTTPS working
- [ ] Error pages displaying
- [ ] Loading states showing

### Performance Testing
```bash
# Using Lighthouse
npm install -g lighthouse
lighthouse https://stockhub.com --view

# Load testing
npm install -g loadtest
loadtest -c 10 -n 100 https://api.stockhub.com/health
```

---

## 🔄 CI/CD Setup

### GitHub Actions

Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.API_URL }}
          VITE_ML_API_URL: ${{ secrets.ML_API_URL }}
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
```

---

## 📧 Email Configuration (Optional)

### SendGrid Setup
```javascript
const sgMail = require('@sendgrid/mail');
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

const msg = {
  to: 'user@example.com',
  from: 'noreply@stockhub.com',
  subject: 'Welcome to StockHub Pro',
  html: '<strong>Welcome!</strong>',
};

await sgMail.send(msg);
```

---

## 🔔 Push Notifications (Optional)

### Firebase Cloud Messaging
```bash
npm install firebase
```

```typescript
import { initializeApp } from 'firebase/app';
import { getMessaging, getToken } from 'firebase/messaging';

const firebaseConfig = {
  // Your config
};

const app = initializeApp(firebaseConfig);
const messaging = getMessaging(app);

getToken(messaging, { vapidKey: 'YOUR_VAPID_KEY' });
```

---

## 📱 PWA Setup (Optional)

### Service Worker
Create `public/sw.js`:
```javascript
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('stockhub-v1').then((cache) => {
      return cache.addAll([
        '/',
        '/index.html',
        '/assets/index.css',
        '/assets/index.js'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

---

## 🎯 Final Checklist

### Pre-Launch
- [ ] All features working
- [ ] Performance optimized
- [ ] Security hardened
- [ ] Monitoring setup
- [ ] Backups configured
- [ ] Documentation updated
- [ ] Legal pages added (Privacy, Terms)
- [ ] Contact page created
- [ ] About page added

### Launch Day
- [ ] Deploy to production
- [ ] Test all features
- [ ] Monitor error logs
- [ ] Check analytics
- [ ] Social media announcement
- [ ] Update documentation

### Post-Launch
- [ ] Monitor performance
- [ ] Check error rates
- [ ] Review user feedback
- [ ] Plan updates
- [ ] Security updates

---

## 🆘 Troubleshooting

### Common Issues

**Issue: Build fails**
```bash
# Clear cache
rm -rf node_modules
rm package-lock.json
npm install
npm run build
```

**Issue: Environment variables not working**
```bash
# Check .env file
cat .env

# Restart server
pm2 restart stockhub-api
```

**Issue: MongoDB connection failed**
```bash
# Check connection string
# Verify IP whitelist
# Test connection
mongo "mongodb+srv://cluster.mongodb.net/test" --username user
```

**Issue: CORS errors**
```javascript
// Update CORS config
app.use(cors({
  origin: ['https://stockhub.com', 'https://www.stockhub.com'],
  credentials: true
}));
```

---

## 📞 Support Resources

- **Vercel**: https://vercel.com/docs
- **Railway**: https://docs.railway.app
- **MongoDB Atlas**: https://docs.atlas.mongodb.com
- **Google Cloud**: https://cloud.google.com/docs
- **Let's Encrypt**: https://letsencrypt.org/docs

---

## 🎉 Congratulations!

You've successfully deployed StockHub Pro to production! 🚀

### Next Steps:
1. Monitor your application
2. Gather user feedback
3. Plan feature updates
4. Scale as needed

**Happy deploying! 📈**

