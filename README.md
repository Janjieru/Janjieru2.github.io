// Backend - server.js (Node.js + Express)
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const path = require('path');
const User = require('./models/User');

const app = express();
const PORT = 5000;
const SECRET_KEY = 'your_secret_key';

app.use(express.json());
app.use(cors());
app.use(express.static(path.join(__dirname, 'public')));

mongoose.connect('mongodb://localhost:27017/authDB', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Homepage Route
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// Signup Route
app.post('/signup', async (req, res) => {
  const { username, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);

  try {
    const newUser = new User({ username, password: hashedPassword });
    await newUser.save();
    res.status(201).json({ message: 'User registered successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Error registering user' });
  }
});

// Login Route
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });

  if (!user) return res.status(400).json({ error: 'User not found' });
  const isMatch = await bcrypt.compare(password, user.password);

  if (!isMatch) return res.status(400).json({ error: 'Invalid credentials' });
  const token = jwt.sign({ id: user._id }, SECRET_KEY, { expiresIn: '1h' });

  res.json({ token });
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

// User Model - models/User.js
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

module.exports = mongoose.model('User', UserSchema);

// Public folder - index.html
/*
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Home Page</title>
</head>
<body>
    <h1>Welcome to the Authentication System</h1>
    <p>Please <a href="/signup">Sign Up</a> or <a href="/login">Log In</a></p>
</body>
</html>
*/
