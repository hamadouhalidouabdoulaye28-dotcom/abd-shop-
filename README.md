# abd-shop-abd-shop/
│
├─ backend/
│   ├─ server.js
│   ├─ package.json
│   ├─ .env.example
│   ├─ routes/
│   │   └─ payment.js
│   └─ controllers/
│       └─ airtelController.js
│
└─ frontend/
    ├─ package.json
    ├─ public/
    │   └─ index.html
    └─ src/
        ├─ index.js
        ├─ index.css
        ├─ App.js
        └─ ABDShopStarter.jsx

{
  "name": "abd-shop-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "axios": "^1.4.0",
    "body-parser": "^1.20.2",
    "express": "^4.18.2",
    "dotenv": "^16.0.3"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}


const express = require('express');
const bodyParser = require('body-parser');
const paymentRoutes = require('./routes/payment');
const app = express();

app.use(bodyParser.json());
app.use('/api/payment', paymentRoutes);
app.get('/', (req, res) => res.send('ABD-SHOP Airtel Money API'));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`API running on ${PORT}`));