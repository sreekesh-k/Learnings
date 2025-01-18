
# **Authentication and Security in Node.js**

This document covers key concepts and tools for authentication and security in Node.js, including **crypto**, **bcrypt**, **HMAC**, and **JWT**. Examples, alternatives, and a complete working example are provided.

---

## **1. Crypto**
The `crypto` module in Node.js provides cryptographic functionalities. It is used for generating secure keys, encrypting data, and hashing.

### **Usage**
1. **Generating a Secure Random Key**:
   ```javascript
   const crypto = require('crypto');
   const secretKey = crypto.randomBytes(64).toString('hex');
   console.log('Secret Key:', secretKey);
   ```

2. **Hashing Data (e.g., for HMAC)**:
   ```javascript
   const data = "HelloWorld";
   const secret = "mySecretKey";

   const hmac = crypto.createHmac('sha256', secret)
                      .update(data)
                      .digest('hex');
   console.log('HMAC:', hmac);
   ```

### **Alternatives**
- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) for token creation and verification.
- [bcryptjs](https://www.npmjs.com/package/bcryptjs) for simpler password hashing.

---

## **2. Bcrypt**
Bcrypt is a library for hashing passwords securely. It’s ideal for storing passwords in databases.

### **Usage**
1. **Hashing a Password**:
   ```javascript
   const bcrypt = require('bcrypt');
   const password = "userPassword123";

   bcrypt.hash(password, 10, (err, hash) => {
     if (err) throw err;
     console.log('Hashed Password:', hash);
   });
   ```

2. **Verifying a Password**:
   ```javascript
   const hashedPassword = "<stored-hash>";
   const passwordToCheck = "userPassword123";

   bcrypt.compare(passwordToCheck, hashedPassword, (err, result) => {
     if (result) console.log('Password is valid!');
     else console.log('Invalid password.');
   });
   ```

### **Alternatives**
- [Argon2](https://www.npmjs.com/package/argon2): A newer hashing algorithm considered more secure than bcrypt.
- PBKDF2 (via the `crypto` module).

---

## **3. HMAC**
HMAC (Hash-based Message Authentication Code) is used to verify the integrity and authenticity of data.

### **Usage**
1. **Creating an HMAC**:
   ```javascript
   const crypto = require('crypto');
   const secret = "superSecretKey";
   const message = "importantData";

   const hmac = crypto.createHmac('sha256', secret)
                      .update(message)
                      .digest('hex');
   console.log('Generated HMAC:', hmac);
   ```

2. **Verifying HMAC**:
   - Compare the received HMAC with the newly generated one to ensure authenticity.

---

## **4. JWT (JSON Web Token)**
JWT is a compact, URL-safe token format used for authentication. It ensures data integrity and authenticity.

### **Structure**
JWT has three parts:
1. **Header**: Metadata (e.g., algorithm, token type).
2. **Payload**: User data (e.g., userId, role).
3. **Signature**: Validates the token.

Example JWT:
```plaintext
Header.Payload.Signature
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyMywicm9sZSI6ImFkbWluIn0.sflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### **Usage**

#### **Creating a Token**
```javascript
const jwt = require('jsonwebtoken');

const user = { userId: 123, role: "admin" };
const secretKey = "mySuperSecretKey";

const token = jwt.sign(user, secretKey, { expiresIn: '1h' });
console.log('Generated Token:', token);
```

#### **Verifying a Token**
```javascript
const token = "<received-token>";
jwt.verify(token, secretKey, (err, decoded) => {
  if (err) console.log('Invalid Token');
  else console.log('Decoded Payload:', decoded);
});
```

---

## **Real-Life Example Scenario**
### **Login Flow**
1. **Registration**:
   - User provides a password. Hash it with bcrypt and store it in the database.

2. **Login**:
   - Verify the provided password with the stored hash.
   - If valid, generate a JWT token containing the user's `userId` and `role`.

3. **Access Protected Routes**:
   - Include the token in the `Authorization` header when making requests.
   - Verify the token on the server side and allow or deny access.

### **Complete Example Code**
```javascript
const express = require('express');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

const app = express();
app.use(express.json());

const SECRET_KEY = crypto.randomBytes(64).toString('hex');
console.log('SECRET_KEY:', SECRET_KEY);

const users = [];

// Register User
app.post('/register', async (req, res) => {
  const { username, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  users.push({ username, password: hashedPassword });
  res.status(201).json({ message: 'User registered successfully' });
});

// Login User
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.find(u => u.username === username);
  if (!user) return res.status(404).json({ message: 'User not found' });

  const isValid = await bcrypt.compare(password, user.password);
  if (!isValid) return res.status(401).json({ message: 'Invalid password' });

  const token = jwt.sign({ username }, SECRET_KEY, { expiresIn: '1h' });
  res.json({ message: 'Login successful', token });
});

// Protected Route
app.get('/protected', (req, res) => {
  const token = req.header('Authorization')?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'Access Denied' });

  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    res.json({ message: `Hello, ${decoded.username}!` });
  } catch {
    res.status(403).json({ message: 'Invalid Token' });
  }
});

app.listen(3000, () => console.log('Server running on http://localhost:3000'));
```
