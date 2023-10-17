const express = require("express");
const mongoose = require("mongoose");
const multer = require("multer");
const path = require("path");
const app = express();

// Conexão com o banco de dados
mongoose.connect("mongodb://localhost:27017/mini-blog", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Configuração do multer
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    cb(null, `${file.fieldname}-${Date.now()}${path.extname(file.originalname)}`);
  },
});
const upload = multer({ storage });

// Model de postagem
const Post = mongoose.model("Post", {
  title: {
    type: String,
    required: true,
  },
  content: {
    type: String,
    required: true,
  },
  image: {
    type: String,
  },
});

// Middlewares
app.use(express.json());
app.use(express.urlencoded({ extended: true });

// Configurar CORS
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

// Rotas
app.get("/", (req, res) => {
  res.send("Hello World!");
});

app.post("/api/posts", upload.single("image"), async (req, res) => {
  const { title, content } = req.body;
  const post = new Post({
    title,
    content,
    image: req.file.filename,
  });
  await post.save();
  res.send(post);
});

app.get("/api/posts", async (req, res) => {
  const posts = await Post.find();
  res.send(posts);
});

app.get("/api/posts/:id", async (req, res) => {
  const post = await Post.findById(req.params.id);
  res.send(post);
});

app.put("/api/posts/:id", upload.single("image"), async (req, res) => {
  const { title, content } = req.body;
  try {
    const post = await Post.findById(req.params.id);
    if (!post) {
      return res.status(404).send("Post not found");
    }
    post.title = title;
    post.content = content;
    if (req.file) {
      post.image = req.file.filename;
    }
    await post.save();
    res.send(post);
  } catch (error) {
    res.status(500).send("Error updating the post");
  }
});

app.delete("/api/posts/:id", async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);
    if (!post) {
      return res.status(404).send("Post not found");
    }
    await post.remove();
    res.send("Post deleted successfully");
  } catch (error) {
    res.status(500).send("Error deleting the post");
  }
});

// Configurar o servidor Express para ouvir em uma porta
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
