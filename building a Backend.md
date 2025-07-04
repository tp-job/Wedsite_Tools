Build a Backend
```cmd
npm init -y
npm install express sqlite3 cors body-parser jsonwebtoken bcryptjs
npm install --save-dev nodemon
```

Install package.json
```json
 "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
```

database.js
```js
const sqlite3 = require("sqlite3").verbose();

const db = new sqlite3.Database("./db.sqlite", (err) => {
  if (err) console.error(err.message);
  console.log("Connected to SQLite database.");
});

db.run(`
  CREATE TABLE IF NOT EXISTS files (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    filename TEXT,
    filepath TEXT,
    type TEXT
  )
`);

module.exports = db;

```

server.js
```js
const express = require("express");
const cors = require("cors");
const multer = require("multer");
const path = require("path");
const db = require("./database"); // นำเข้า database.js

const app = express();
app.use(cors());
app.use(express.json());
app.use("/uploads", express.static(path.join(__dirname, "uploads")));

const storage = multer.diskStorage({
  destination: "./uploads/",
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  },
});

const upload = multer({ storage });

// อัปโหลดไฟล์และบันทึกลงฐานข้อมูล
app.post("/upload", upload.single("file"), (req, res) => {
  const { filename, path: filepath, mimetype } = req.file;
  db.run(`INSERT INTO files (filename, filepath, type) VALUES (?, ?, ?)`,
    [filename, filepath, mimetype],
    function (err) {
      if (err) return res.status(500).json({ error: err.message });
      res.json({ id: this.lastID, filename, filepath, type: mimetype });
    }
  );
});

// ดึงข้อมูลไฟล์ทั้งหมดจากฐานข้อมูล
app.get("/files", (req, res) => {
  db.all("SELECT * FROM files", [], (err, rows) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(rows);
  });
});

app.listen(5000, () => console.log("Server running on port 5000"));

```
