## Lab: Improving Performance with Redis Caching

[Redis caching flow in Node.js API.png](./Redis%20caching%20flow%20in%20Node.js%20API.png)

### Lab Objective:

By the end of this lab, you will be able to:

* Integrate Redis as a caching layer in a Node.js/Express API.
* Store and retrieve API responses in Redis to reduce response times.
* Understand cache expiration and invalidation strategies.

---

### Prerequisites:

You should have:

* Node.js and npm installed
* Basic knowledge of Express.js
* Postman or curl for testing APIs
* Redis installed locally or access to a cloud Redis instance

---

### Lab Setup:

1. **Initialize a new Node.js project:**

```bash
mkdir redis-caching-lab
cd redis-caching-lab
npm init -y
npm install express axios redis
```

2. **Set up a basic Express server (`index.js`):**

```javascript
const express = require('express');
const axios = require('axios');

const app = express();
const PORT = 3000;

app.get('/joke', async (req, res) => {
    try {
        const response = await axios.get('https://official-joke-api.appspot.com/random_joke');
        res.json(response.data);
    } catch (error) {
        res.status(500).json({ error: 'Failed to fetch joke' });
    }
});

app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
```

3. Test the endpoint by running:

```bash
node index.js
```

* Open `http://localhost:3000/joke` in your browser or Postman.

---

### Lab Steps:

#### **Step 1: Connect Redis**

1. Import and configure Redis in `index.js`:

```javascript
const redis = require('redis');
const client = redis.createClient();

client.on('error', (err) => console.log('Redis Client Error', err));
await client.connect();
```

---

#### **Step 2: Implement Caching**

* Modify the `/joke` route to use Redis:

```javascript
app.get('/joke', async (req, res) => {
    try {
        // Check Redis first
        const cachedJoke = await client.get('random_joke');
        if (cachedJoke) {
            return res.json(JSON.parse(cachedJoke));
        }

        // Fetch from API if not cached
        const response = await axios.get('https://official-joke-api.appspot.com/random_joke');
        const joke = response.data;

        // Store in Redis with 30-second expiration
        await client.setEx('random_joke', 30, JSON.stringify(joke));

        res.json(joke);
    } catch (error) {
        res.status(500).json({ error: 'Failed to fetch joke' });
    }
});
```

---

#### **Step 3: Test the Cache**

1. Start the server:

```bash
node index.js
```

2. Make repeated requests to `http://localhost:3000/joke`.
3. Observe that the first request fetches from the API, and subsequent requests (within 30 seconds) are served from Redis.

### Lab Deliverables:

1. Updated `index.js` with Redis caching implemented.
2. Evidence that caching reduces repeated request time (e.g., console logs or Postman response times).
3. Optional: Cache invalidation endpoint working.
