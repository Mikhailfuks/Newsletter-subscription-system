
const mongoose = require('mongoose');

const subscriptionSchema = new mongoose.Schema({
    email: { type: String, required: true, unique: true },
}, { timestamps: true });

module.exports = mongoose.model('Subscription', subscriptionSchema);

// index.js
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
require('dotenv').config();
const Subscription = require('./models/Subscription');

const app = express();
const PORT = process.env.PORT || 3000;

// Подключение к MongoDB
mongoose.connect(process.env.MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected successfully!'))
.catch(err => console.error('MongoDB connection error:', err));

// Middleware
app.use(bodyParser.json());
app.use(express.static('public'));

// Маршрут для подписки
app.post('/subscribe', async (req, res) => {
    const { email } = req.body;
    try {
        const subscription = new Subscription({ email });
        await subscription.save();
        res.status(201).json({ message: 'Successfully subscribed!' });
    } catch (error) {
        if (error.code === 11000) {
            return res.status(409).json({ message: 'Email already subscribed!' });
        }
        res.status(500).json({ message: 'Error subscribing', error });
    }
});

// Запуск сервера
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Newsletter Subscription</title>
    <link rel="stylesheet" href="styles.css">
    <script defer src="app.js"></script>
</head>
<body>
    <div class="container">
        <h1>Subscribe to our Newsletter</h1>
        <form id="subscribeForm">
            <input type="email" id="email" placeholder="Enter your email" required>
            <button type="submit">Subscribe</button>
        </form>
        <p id="message"></p>
    </div>
</body>
</html>

/* public/styles.css */
body {
    font-family: Arial, sans-serif;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100vh;
    margin: 0;
    background-color: #f8f8f8;
}

.container {
    text-align: center;
}

input {
    padding: 10px;
    margin: 5px;
}

button {


    padding: 10px 15px;
    background-color: #28a745;
    color: white;
    border: none;
    cursor: pointer;
}

button:hover {
    background-color: #218838;
}

#error {
    color: red;
}

// public/app.js
document.getElementById('subscribeForm').addEventListener('submit', async function(event) {
    event.preventDefault();
    
    const email = document.getElementById('email').value;
    const messageElement = document.getElementById('message');

    try {
        const response = await fetch('/subscribe', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email })
        });

        if (response.ok) {
            messageElement.textContent = 'Successfully subscribed!';
            messageElement.style.color = 'green';
            document.getElementById('email').value = ''; // Очистить поле
        } else {
            const errorData = await response.json();
            messageElement.textContent = errorData.message;
            messageElement.style.color = 'red';
        }
    } catch (error) {
        messageElement.textContent = 'An error occurred. Please try again.';
        messageElement.style.color = 'red';
    }
});
