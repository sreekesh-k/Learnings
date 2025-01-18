
# **Behind the Scenes of JWT**

### **What is JWT?**
JWT (JSON Web Token) is a compact, URL-safe token format used for securely transmitting information between parties. It ensures both the integrity and authenticity of the data. 

A JWT consists of three parts:
1. **Header** – Contains metadata about the token, such as the signing algorithm.
2. **Payload** – Contains the claims (i.e., the data being transmitted).
3. **Signature** – Ensures that the token's content is valid and has not been tampered with.

### **JWT Structure**
The structure of a JWT token looks like this:

```plaintext
Header.Payload.Signature
```

**Example JWT:**
```plaintext
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyMywicm9sZSI6ImFkbWluIn0.sflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

- **Header**: Base64Url encoded JSON containing metadata.
- **Payload**: Base64Url encoded JSON containing user data (claims).
- **Signature**: The signature generated using the header, payload, and a secret key.

### **JWT Creation Process (Behind the Scenes)**

#### **1. Header**
The **header** of the JWT typically consists of two parts:
- **`alg`**: The signing algorithm (e.g., HMAC SHA256, RSA, etc.).
- **`typ`**: The token type, which is usually "JWT".

Example:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

The header is then Base64Url encoded:
```plaintext
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

#### **2. Payload**
The **payload** contains the claims (data). Claims are statements about an entity (typically the user) and additional data. There are three types of claims:
- **Registered claims** (predefined): `iss` (issuer), `exp` (expiration time), `sub` (subject), `aud` (audience).
- **Public claims**: Custom claims that are agreed upon and shared between parties.
- **Private claims**: Custom claims used only between specific parties (e.g., user ID, roles).

Example payload (claims):
```json
{
  "userId": 123,
  "role": "admin"
}
```

This payload is also Base64Url encoded:
```plaintext
eyJ1c2VySWQiOjEyMywicm9sZSI6ImFkbWluIn0
```

#### **3. Signature**
The **signature** is created by combining the encoded header and payload, then signing it with a secret key using the specified algorithm (e.g., HMAC SHA256).

Example signing process (using HMAC SHA256):
```plaintext
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secretKey
)
```

This produces the signature:
```plaintext
sflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### **Final JWT**
Combining the three parts results in a JWT:
```plaintext
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyMywicm9sZSI6ImFkbWluIn0.sflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### **JWT Verification Process (Behind the Scenes)**

When a server receives a JWT from a client, it needs to verify its authenticity. Here's the breakdown of the verification process:

#### **1. Split the Token**
The JWT is split into three parts using the `.` separator:
- **Header** (part 1)
- **Payload** (part 2)
- **Signature** (part 3)

#### **2. Recreate the Signature**
The server recreates the signature by taking the **header** and **payload**, and then re-signing them with the **secret key** using the same algorithm (e.g., HMAC SHA256).

Recreation of the signature:
```plaintext
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secretKey
)
```

#### **3. Compare Signatures**
The server compares the **recreated signature** with the **signature** sent in the JWT. If the two match, it confirms that the token has not been tampered with and is valid.

#### **4. Decode the Payload**
Once the token is verified, the server decodes the **payload** to retrieve the data (e.g., `userId`, `role`). This data is now trusted and can be used for authorization purposes.

### **Real-World Scenario**

**Scenario**: A user logs into a system.

1. **Registration**:
   - The user submits their credentials (e.g., username, password).
   - The server hashes the password (using bcrypt) and stores the user data in the database.

2. **Login**:
   - The user submits their credentials again.
   - The server compares the submitted password with the stored hash (using bcrypt).
   - If the password is correct, the server generates a JWT containing the user's `userId` and `role`.
   
   Example JWT:
   ```plaintext
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyMywicm9sZSI6ImFkbWluIn0.sflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
   ```

3. **Accessing Protected Routes**:
   - The user includes the JWT in the `Authorization` header of subsequent requests.
   - The server verifies the JWT by splitting it, decoding the payload, and checking the signature.
   - If the token is valid, the server grants access to the requested resource.

### **Full Example (JWT Creation and Verification)**

**Server Code (Express.js):**
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

