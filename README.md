<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Library Management System</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f8f9fa;
      margin: 0;
      padding: 20px;
    }
    .container {
      max-width: 900px;
      margin: auto;
      background: #fff;
      padding: 25px;
      border-radius: 8px;
      box-shadow: 0 0 15px rgba(0,0,0,0.1);
    }
    h1, h2 {
      text-align: center;
    }
    input, button {
      padding: 10px;
      margin: 8px 0;
      width: 100%;
      box-sizing: border-box;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 15px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 10px;
    }
    th {
      background-color: #f0f0f0;
    }
    .hidden {
      display: none;
    }
    .admin-only {
      display: none;
    }
    .btn {
      background-color: #007bff;
      color: white;
      border: none;
      cursor: pointer;
    }
    .btn-danger {
      background-color: #dc3545;
    }
    .btn-secondary {
      background-color: #6c757d;
    }
  </style>
</head>
<body>
<div class="container">
  <h1>📚 Library Management System</h1>  <div id="auth">
    <h2>Register</h2>
    <input type="text" id="reg-username" placeholder="Username">
    <input type="password" id="reg-password" placeholder="Password">
    <button onclick="register()" class="btn">Register</button><h2>Login</h2>
<input type="text" id="login-username" placeholder="Username">
<input type="password" id="login-password" placeholder="Password">
<button onclick="login()" class="btn">Login</button>

  </div>  <div id="dashboard" class="hidden">
    <h2>Welcome, <span id="user-display"></span></h2>
    <button onclick="logout()" class="btn btn-danger">Logout</button><div class="admin-only">
  <h3>Add Book</h3>
  <input type="text" id="book-title" placeholder="Book Title">
  <input type="text" id="book-author" placeholder="Book Author">
  <input type="text" id="book-genre" placeholder="Genre (for AI)">
  <button onclick="addBook()" class="btn">Add Book</button>
</div>

<h3>All Books</h3>
<input type="text" id="search" onkeyup="renderBooks()" placeholder="Search books by title, author or genre">
<table>
  <thead>
    <tr><th>Title</th><th>Author</th><th>Status</th><th>Action</th></tr>
  </thead>
  <tbody id="book-list"></tbody>
</table>

<h3>📘 My Issued Books</h3>
<table>
  <thead>
    <tr><th>Title</th><th>Author</th><th>Issued On</th><th>Returned On</th></tr>
  </thead>
  <tbody id="issued-list"></tbody>
</table>

<h3>🔮 AI Recommended for You</h3>
<ul id="recommendations"></ul>

  </div>
</div><script>
let users = JSON.parse(localStorage.getItem("users")) || [{ username: "admin", password: "admin", role: "admin" }];
let books = JSON.parse(localStorage.getItem("books")) || [];
let issued = JSON.parse(localStorage.getItem("issued")) || [];
let currentUser = null;

function saveAll() {
  localStorage.setItem("users", JSON.stringify(users));
  localStorage.setItem("books", JSON.stringify(books));
  localStorage.setItem("issued", JSON.stringify(issued));
}

function register() {
  const username = document.getElementById("reg-username").value;
  const password = document.getElementById("reg-password").value;
  if (!username || !password) return alert("Please fill both fields.");
  if (users.find(u => u.username === username)) return alert("Username already exists.");
  users.push({ username, password, role: "student" });
  saveAll();
  alert("Registration successful. You can now login.");
}

function login() {
  const username = document.getElementById("login-username").value;
  const password = document.getElementById("login-password").value;
  const user = users.find(u => u.username === username && u.password === password);
  if (!user) return alert("Invalid credentials.");
  currentUser = user;
  document.getElementById("auth").classList.add("hidden");
  document.getElementById("dashboard").classList.remove("hidden");
  document.getElementById("user-display").textContent = currentUser.username;
  if (user.role === "admin") document.querySelector(".admin-only").style.display = "block";
  renderBooks();
  renderIssued();
  renderRecommendations();
}

function logout() {
  currentUser = null;
  document.getElementById("auth").classList.remove("hidden");
  document.getElementById("dashboard").classList.add("hidden");
  document.querySelector(".admin-only").style.display = "none";
}

function addBook() {
  const title = document.getElementById("book-title").value;
  const author = document.getElementById("book-author").value;
  const genre = document.getElementById("book-genre").value;
  if (!title || !author || !genre) return alert("Fill all fields.");
  books.push({ id: Date.now(), title, author, genre, status: "available" });
  saveAll();
  renderBooks();
}

function renderBooks() {
  const search = document.getElementById("search").value.toLowerCase();
  const tbody = document.getElementById("book-list");
  tbody.innerHTML = "";
  books.filter(b => b.title.toLowerCase().includes(search) || b.author.toLowerCase().includes(search) || b.genre.toLowerCase().includes(search)).forEach(book => {
    const tr = document.createElement("tr");
    tr.innerHTML = `<td>${book.title}</td>
                    <td>${book.author}</td>
                    <td>${book.status}</td>
                    <td>
                      ${currentUser.role === "student" && book.status === "available" ? `<button class='btn btn-secondary' onclick='issueBook(${book.id})'>Issue</button>` : ""}
                      ${currentUser.role === "student" && book.status === "issued" && issued.find(i => i.bookId === book.id && i.username === currentUser.username && !i.returned) ? `<button class='btn btn-secondary' onclick='returnBook(${book.id})'>Return</button>` : ""}
                    </td>`;
    tbody.appendChild(tr);
  });
}

function issueBook(id) {
  const book = books.find(b => b.id === id);
  if (!book || book.status !== "available") return;
  book.status = "issued";
  issued.push({ username: currentUser.username, bookId: id, issueDate: new Date().toLocaleDateString(), returnDate: "", returned: false });
  saveAll();
  renderBooks();
  renderIssued();
  renderRecommendations();
}

function returnBook(id) {
  const book = books.find(b => b.id === id);
  if (!book || book.status !== "issued") return;
  book.status = "available";
  const record = issued.find(i => i.bookId === id && i.username === currentUser.username && !i.returned);
  if (record) {
    record.returned = true;
    record.returnDate = new Date().toLocaleDateString();
  }
  saveAll();
  renderBooks();
  renderIssued();
  renderRecommendations();
}

function renderIssued() {
  const tbody = document.getElementById("issued-list");
  tbody.innerHTML = "";
  issued.filter(i => i.username === currentUser.username).forEach(i => {
    const book = books.find(b => b.id === i.bookId);
    const tr = document.createElement("tr");
    tr.innerHTML = `<td>${book.title}</td><td>${book.author}</td><td>${i.issueDate}</td><td>${i.returnDate || "Not returned"}</td>`;
    tbody.appendChild(tr);
  });
}

function renderRecommendations() {
  const recommendations = document.getElementById("recommendations");
  recommendations.innerHTML = "";
  const myBooks = issued.filter(i => i.username === currentUser.username && !i.returned);
  let keywords = new Set();
  myBooks.forEach(i => {
    const book = books.find(b => b.id === i.bookId);
    if (book) {
      book.title.split(" ").forEach(word => keywords.add(word.toLowerCase()));
      book.author.split(" ").forEach(word => keywords.add(word.toLowerCase()));
      book.genre.split(" ").forEach(word => keywords.add(word.toLowerCase()));
    }
  });
  if (keywords.size === 0) {
    const top = books.slice(0, 3);
    top.forEach(book => {
      const li = document.createElement("li");
      li.textContent = `${book.title} by ${book.author} (${book.genre})`;
      recommendations.appendChild(li);
    });
    return;
  }
  const recs = books.filter(book =>
    [...keywords].some(k => book.title.toLowerCase().includes(k) || book.author.toLowerCase().includes(k) || book.genre.toLowerCase().includes(k)) &&
    !issued.some(i => i.bookId === book.id && i.username === currentUser.username)
  );
  if (recs.length === 0) {
    recommendations.innerHTML = "<li>No new recommendations yet.</li>";
  } else {
    recs.slice(0, 5).forEach(book => {
      const li = document.createElement("li");
      li.textContent = `${book.title} by ${book.author} (${book.genre})`;
      recommendations.appendChild(li);
    });
  }
}
</script></body>
</html>