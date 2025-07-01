# server.js
const express = require('express');
const multer = require('multer');
const fs = require('fs');
const axios = require('axios');
const path = require('path');
const cors = require('cors');

const app = express();
const upload = multer({ dest: 'uploads/' });

app.use(cors());
app.use(express.static(__dirname));

app.get("/", (req, res) => {
  res.sendFile(path.join(__dirname, "index.html"));
});

app.post('/remove-bg', upload.single('image'), async (req, res) => {
  const filePath = req.file.path;
  try {
    const response = await axios({
      method: 'post',
      url: 'https://api.remove.bg/v1.0/removebg',
      data: fs.createReadStream(filePath),
      headers: {
        'X-Api-Key': 'TqnCkRBrgN2q5WtzhgNCZQPF'  // ✅ Your API Key
      },
      responseType: 'arraybuffer'
    });

    fs.unlinkSync(filePath);
    res.set('Content-Type', 'image/png');
    res.send(response.data);
  } catch (err) {
    console.error(err);
    res.status(500).send("Background removal failed");
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`✅ Server running on port ${PORT}`));
