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

AIRTEL_API_BASE=https://sandbox.airtelmoney.com
AIRTEL_CLIENT_ID=your_airtel_client_id
AIRTEL_CLIENT_SECRET=your_airtel_client_secret
AIRTEL_CALLBACK_URL=https://your-backend-url/api/payment/airtel/webhook
PORT=3000


const express = require('express');
const router = express.Router();
const airtel = require('../controllers/airtelController');

router.post('/airtel/create', airtel.createPayment);
router.post('/airtel/webhook', airtel.webhookListener);

module.exports = router;


const axios = require('axios');

const AIRTEL_BASE = process.env.AIRTEL_API_BASE;
const CLIENT_ID = process.env.AIRTEL_CLIENT_ID;
const CLIENT_SECRET = process.env.AIRTEL_CLIENT_SECRET;
const CALLBACK_URL = process.env.AIRTEL_CALLBACK_URL;

async function getAccessToken() {
  const resp = await axios.post(`${AIRTEL_BASE}/oauth/token`, {
    client_id: CLIENT_ID,
    client_secret: CLIENT_SECRET,
    grant_type: 'client_credentials'
  });
  return resp.data.access_token;
}

exports.createPayment = async (req, res) => {
  const { orderId, amount, customerMsisdn } = req.body;
  try {
    const token = await getAccessToken();
    const payload = {
      amount: amount.toString(),
      currency: 'XAF',
      externalId: orderId,
      payer: { partyIdType: 'MSISDN', partyId: customerMsisdn },
      payerMessage: `Paiement commande ${orderId}`,
      callbackUrl: CALLBACK_URL,
    };
    const response = await axios.post(`${AIRTEL_BASE}/v1/payments`, payload, {
      headers: { Authorization: `Bearer ${token}` },
    });
    res.json({ ok: true, data: response.data });
  } catch (err) {
    console.error('Airtel Payment Error:', err.message);
    res.status(500).json({ ok: false, error: err.message });
  }
};

exports.webhookListener = async (req, res) => {
  console.log('Airtel webhook:', req.body);
  res.status(200).send('OK');
};



{
  "name": "abd-shop-frontend",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:3000",
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.6.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  }
}


<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ABD-SHOP</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>


import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);


import React from 'react';
import ABDShopStarter from './ABDShopStarter';

function App() {
  return <ABDShopStarter />;
}

export default App;


import React, { useState } from 'react';
import axios from 'axios';

export default function ABDShopStarter() {
  const [orderId, setOrderId] = useState('TEST123');
  const [amount, setAmount] = useState(1500);
  const [phone, setPhone] = useState('22890000000');
  const [loading, setLoading] = useState(false);

  const handlePay = async () => {
    setLoading(true);
    try {
      const resp = await axios.post('/api/payment/airtel/create', {
        orderId,
        amount,
        customerMsisdn: phone
      });
      if (resp.data.data.paymentUrl) {
        window.location.href = resp.data.data.paymentUrl;
      } else {
        alert('Paiement initié, vérifiez votre téléphone Airtel Money.');
      }
    } catch (err) {
      console.error(err);
      alert('Erreur lors de la création du paiement.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div style={{ padding: '20px', maxWidth: '400px', margin: 'auto' }}>
      <h1>ABD-SHOP</h1>
      <div>
        <label>Montant (FCFA)</label>
        <input value={amount} onChange={(e) => setAmount(e.target.value)} type="number" />
      </div>
      <div>
        <label>Téléphone Airtel Money</label>
        <input value={phone} onChange={(e) => setPhone(e.target.value)} type="text" />
      </div>
      <button onClick={handlePay} disabled={loading} style={{ marginTop: '10px' }}>
        {loading ? 'En cours...' : 'Payer via Airtel Money'}
      </button>
    </div>
  );
}