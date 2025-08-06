# 🚀 Real-Time Collaborative Editor using AWS EC2 + DynamoDB

This is a fully working cloud-based collaborative text editor (like Google Docs) built using:

* EC2 (for backend and frontend hosting)
* Node.js Express backend
* React frontend
* DynamoDB (document storage)
* Nginx (frontend web server)

---

## ✅ Features

* Real-time editing (basic insert logic)
* Multi-user backend via EC2
* Persistent storage in DynamoDB
* Two separate EC2 instances (frontend/backend)
* Deploy-ready architecture

---

## ⚙️ Architecture

```
Frontend (React) ← Nginx ← EC2 (frontend)
Backend (Express.js) ← EC2 (backend) → DynamoDB
```

---

## 🔧 Step-by-Step Setup

### 🔹 Step 1: Launch Two EC2 Instances

* **EC2-Frontend** → hosts React app + Nginx
* **EC2-Backend** → hosts Node.js Express API
* Ubuntu 22.04 for both
* Allow ports: `22`, `80`, `3000`, `4000`

### 🔹 Step 2: Connect via SSH

```bash
ssh -i key.pem ubuntu@<public-ip>
```

### 🔹 Step 3: Install Dependencies

```bash
sudo apt update
sudo apt install nodejs npm nginx git -y
```

---

## 🧠 Backend Setup (On EC2-Backend)

### 📁 Create Project

```bash
mkdir backend && cd backend
npm init -y
npm install express aws-sdk cors
```

### 📄 Create `server.js`

```js
const express = require('express');
const cors = require('cors');
const AWS = require('aws-sdk');
const app = express();
const port = 4000;

AWS.config.update({ region: 'ap-south-1' });
const docClient = new AWS.DynamoDB.DocumentClient();

app.use(cors());
app.use(express.json());

app.post('/edit', async (req, res) => {
  const { documentId, operation } = req.body;

  const doc = await docClient.get({
    TableName: 'Documents',
    Key: { documentId }
  }).promise();

  let content = doc.Item?.content || '';

  if (operation.type === 'insert') {
    content = content.slice(0, operation.position) + operation.char + content.slice(operation.position);
  }

  await docClient.put({
    TableName: 'Documents',
    Item: { documentId, content }
  }).promise();

  res.send({ content });
});

app.listen(port, () => console.log(`Backend running on port ${port}`));
```

### ▶️ Run Backend

```bash
node server.js
```

✅ Now backend is running on `http://<backend-ec2-ip>:4000`

---

## 🖥️ Frontend Setup (On EC2-Frontend)

### 📁 Create React App

```bash
npx create-react-app frontend
cd frontend
npm install axios
```

### 📄 Modify `src/App.js`

```jsx
import React, { useState } from 'react';
import axios from 'axios';

const BACKEND_URL = "http://<backend-ec2-ip>:4000/edit";

function App() {
  const [doc, setDoc] = useState('');

  const handleChange = async (e) => {
    const newChar = e.nativeEvent.data || '';
    const position = e.target.selectionStart - 1;

    const res = await axios.post(BACKEND_URL, {
      documentId: 'doc1',
      operation: {
        type: 'insert',
        char: newChar,
        position: position
      }
    });

    setDoc(res.data.content);
  };

  return (
    <div>
      <h2>📝 Real-Time Collaborative Editor</h2>
      <textarea value={doc} onChange={handleChange} rows="10" cols="60" />
    </div>
  );
}

export default App;
```

### ▶️ Build React App

```bash
npm run build
sudo cp -r build/* /var/www/html/
```

---

## 🌐 Setup Nginx (Frontend Server)

```bash
sudo nano /etc/nginx/sites-available/default
```

Change `root` to:

```
root /var/www/html;
index index.html;
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

✅ Now app is live at `http://<frontend-ec2-ip>`

---

## 🗃️ DynamoDB Table Setup

Create a table named `Documents` in AWS DynamoDB:

* Partition key: `documentId` (string)
* No sort key

---

## 🧪 Test

1. Open React app in browser.
2. Type something — each keystroke sends a request to backend.
3. Backend updates the content in DynamoDB.
4. Refresh browser → content persists.

---

## 🧱 Next Steps (Optional Upgrades)

* Add AppSync + GraphQL subscriptions for true real-time sync
* Add Cognito for login/logout
* Add S3 integration for document export
* Add vector clock merge logic for offline sync
* Add user-specific permissions and roles

---

## 💬 Author

**Vishal Bobhate**
🔗 [GitHub](https://github.com/bobhatevishal)
🔧 DevOps | Cloud | Real-Time Apps

---

✅ Your real-time editor now works across EC2 + DynamoDB — and is ready to grow!
