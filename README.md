
# Gemini Prompt Wrapper - Mini Project Guide

## 📚 Project Objective

Create a Node.js + Express backend API using the MVC pattern to securely call the **Gemini 2.0 Flash API** for content generation. Students will:

- Use **ES Modules** instead of CommonJS.
- Implement the **MVC** pattern with a **class-based controller**.
- Securely use API keys using a .env file.
- Implement error handling using a global error handler middleware.
- Document API with **Swagger (OpenAPI)**.
- Create a basic **frontend** (HTML + JS) to call the backend API.

---

## ✅ Step-by-Step Instructions

### Step 1: Initialize Project

```bash
mkdir gemini-api-wrapper
cd gemini-api-wrapper
npm init -y
```

### Step 2: Install Dependencies

```bash
npm install express axios dotenv cors
npm install --save-dev nodemon
```

### Step 3: File Structure

```
gemini-api-wrapper/
├── controllers/
│   └── GeminiController.js
├── dao/
│   └── GeminiDao.js
├── routes/
│   └── geminiRoutes.js
├── swagger/
│   └── openapi.yaml
├── public/
│   └── index.html
├── .env
├── app.js
└── package.json
```

### Step 4: Configure Express App (ESM)

**`app.js`**
```js
import express from 'express';
import dotenv from 'dotenv';
import cors from 'cors';
import geminiRoutes from './routes/geminiRoutes.js';

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static('public'));

app.use('/api/generate', geminiRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

Update `package.json` to use ESM:
```json
"type": "module"
```

### Step 5: Swagger (OpenAPI) Definition

**`swagger/openapi.yaml`**
```yaml
openapi: 3.0.0
info:
  title: Gemini Prompt Wrapper API
  version: 1.0.0
paths:
  /api/generate:
    post:
      summary: Generate content using Gemini 2.0 Flash
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                prompt:
                  type: string
                  example: "Write a poem about spring"
      responses:
        '200':
          description: Gemini Response
        '400':
          description: Invalid input
        '500':
          description: Gemini API Error
```

### Step 6: Routes

**`routes/geminiRoutes.js`**
```js
import express from 'express';
import GeminiController from '../controllers/GeminiController.js';

const router = express.Router();
const controller = new GeminiController();

router.post('/', controller.generateContent);

export default router;
```

### Step 7: Controller (Class-Based)

**`controllers/GeminiController.js`**
```js
import GeminiDao from '../dao/GeminiDao.js';

export default class GeminiController {
  async generateContent(req, res) {
    const { prompt } = req.body;
	// TODO: Validate prompt and handle request/response
	// validatePrompt();  //e.g. Limit the prompt size to 1000 characters, otherwise throw validation error
    const dao = new GeminiDao();
    try {
      const result = await dao.callGeminiAPI(prompt);
      res.status(200).json({ content: result });
    } catch (error) {
      res.status(500).json({ error: 'Failed to fetch content from Gemini' });
	  //TODO: Use a global error handler to handle the request
    }
  }
}
```

### Step 8: DAO - Call Gemini API

**`dao/GeminiDao.js`**
```js
import axios from 'axios';

const GEMINI_API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent';
const API_KEY = process.env.GEMINI_API_KEY;

export default class GeminiDao {
  async callGeminiAPI(prompt) {
    const response = await axios.post(
      `${GEMINI_API_URL}?key=${API_KEY}`,
      { contents: [{ parts: [{ text: prompt }] }] },
      { headers: { 'Content-Type': 'application/json' } }
    );

    const candidates = response.data.candidates;
    return candidates[0]?.content?.parts[0]?.text || 'No response from Gemini.';
  }
}

//TODO: Optional: You can try converting this code to use standard `fetch` function instead of axios
```

### Step 9: .env File

**`.env`**
```env
GEMINI_API_KEY=your_api_key_here
```

### Step 10: Frontend

**`public/index.html`**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Gemini Prompt Wrapper</title>
</head>
<body>
  <h1>Ask Gemini</h1>
  <textarea id="prompt" rows="5" cols="50" placeholder="Type your prompt..."></textarea><br>
  <button onclick="sendPrompt()">Generate</button>
  <pre id="result"></pre>

  <script>
    async function sendPrompt() {
      const prompt = document.getElementById('prompt').value;
      const res = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ prompt })
      });
      const data = await res.json();
      document.getElementById('result').innerText = data.content || data.error;
    }
  </script>
</body>
</html>
```

---

## 📌 Submission Requirements

Students must submit:
- ✅ GitHub repo link
- ✅ Working Swagger API doc (openapi.yaml)
- ✅ Working demo (local or deployed)
- ✅ Architecture explanation (MVC + DAO)
- ✅ (Optional) Bonus features like style selector, history, tone.

---

## 🎯 Bonus Challenges

- Allow tone/style selector (funny, poetic, formal)
- Add rate limiting using middleware
- Save responses in-memory for user session (mock DB)

---

All the best! 🎉
