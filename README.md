<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Customer Information System</title>
<style>
    * { box-sizing: border-box; }
    body {
        font-family: Arial, sans-serif;
        margin: 0;
        background: #f2f4f7;
        color: #222;
    }
    .container {
        max-width: 1050px;
        margin: 30px auto;
        padding: 20px;
    }
    .card {
        background: white;
        padding: 25px;
        border-radius: 12px;
        box-shadow: 0 3px 12px rgba(0,0,0,.12);
    }
    h1, h2 { text-align: center; }
    input, button {
        padding: 10px;
        margin: 5px 0;
        border-radius: 6px;
        border: 1px solid #bbb;
    }
    input { width: 100%; }
    button {
        border: 0;
        background: #2563eb;
        color: white;
        cursor: pointer;
        padding: 10px 16px;
    }
    button:hover { opacity: .9; }
    .danger { background: #dc2626; }
    .secondary { background: #6b7280; }
    .success { background: #16a34a; }
    .hidden { display: none; }
    .login-box {
        max-width: 400px;
        margin: 80px auto;
    }
    .form-grid {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 10px;
    }
    .search { margin: 20px 0; }
    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 15px;
    }
    th, td {
        border: 1px solid #ddd;
        padding: 10px;
        text-align: left;
    }
    th { background: #2563eb; color: white; }
    tr:nth-child(even) { background: #f8fafc; }
    .actions button { margin-right: 4px; }
    #message { min-height: 20px; margin: 8px 0; font-weight: bold; }
    @media (max-width: 700px) {
        .form-grid { grid-template-columns: 1fr; }
        table { font-size: 13px; }
        .container { padding: 10px; }
    }
</style>
</head>
<body>

<div id="loginPage" class="login-box card">
    <h1>Customer Information System</h1>
    <h2>LOG-IN</h2>
    <input type="text" id="username" placeholder="Username">
    <input type="password" id="password" placeholder="Password">
    <button style="width:100%" onclick="login()">Login</button>
    <p id="loginMessage"></p>
    <small>Demo account: admin / 1234</small>
</div>

<div id="systemPage" class="container hidden">
    <div class="card">
        <h1>Customer Information System</h1>

        <div class="form-grid">
            <div>
                <label>Customer ID</label>
                <input type="text" id="customerId" placeholder="e.g. C001">
            </div>
            <div>
                <label>Full Name</label>
                <input type="text" id="customerName" placeholder="Customer name">
            </div>
            <div>
                <label>Email</label>
                <input type="email" id="customerEmail" placeholder="email@example.com">
            </div>
            <div>
                <label>Phone</label>
                <input type="text" id="customerPhone" placeholder="09XXXXXXXXX">
            </div>
        </div>

        <p id="message"></p>

        <button class="success" onclick="addCustomer()">ADD CUSTOMER</button>
        <button onclick="editCustomer()">EDIT CUSTOMER</button>
        <button class="danger" onclick="deleteCustomer()">DELETE</button>
        <button class="secondary" onclick="clearForm()">CLEAR</button>
        <button class="secondary" onclick="logout()">LOG-OUT</button>

        <input class="search" type="text" id="searchBox"
               placeholder="SEARCH customer by ID, name, email, or phone..."
               onkeyup="viewCustomers()">

        <h2>VIEW CUSTOMERS</h2>
        <table>
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Full Name</th>
                    <th>Email</th>
                    <th>Phone</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody id="customerTable"></tbody>
        </table>
    </div>
</div>

<script>
let customers = JSON.parse(localStorage.getItem("customers")) || [
    {id:"C001", name:"Juan Dela Cruz", email:"juan@email.com", phone:"09123456789"},
    {id:"C002", name:"Maria Santos", email:"maria@email.com", phone:"09987654321"}
];

function saveData() {
    localStorage.setItem("customers", JSON.stringify(customers));
}

function login() {
    const user = document.getElementById("username").value.trim();
    const pass = document.getElementById("password").value;

    if (user === "admin" && pass === "1234") {
        document.getElementById("loginPage").classList.add("hidden");
        document.getElementById("systemPage").classList.remove("hidden");
        viewCustomers();
    } else {
        document.getElementById("loginMessage").textContent =
            "Invalid username or password.";
        document.getElementById("loginMessage").style.color = "red";
    }
}

function logout() {
    document.getElementById("systemPage").classList.add("hidden");
    document.getElementById("loginPage").classList.remove("hidden");
    document.getElementById("username").value = "";
    document.getElementById("password").value = "";
}

function getFormData() {
    return {
        id: document.getElementById("customerId").value.trim(),
        name: document.getElementById("customerName").value.trim(),
        email: document.getElementById("customerEmail").value.trim(),
        phone: document.getElementById("customerPhone").value.trim()
    };
}

function addCustomer() {
    const c = getFormData();

    if (!c.id || !c.name || !c.email || !c.phone) {
        showMessage("Please complete all fields.", "red");
        return;
    }

    if (customers.some(x => x.id.toLowerCase() === c.id.toLowerCase())) {
        showMessage("Customer ID already exists.", "red");
        return;
    }

    customers.push(c);
    saveData();
    viewCustomers();
    clearForm();
    showMessage("Customer added successfully.", "green");
}

function editCustomer() {
    const c = getFormData();
    const index = customers.findIndex(x => x.id.toLowerCase() === c.id.toLowerCase());

    if (index === -1) {
        showMessage("Customer ID not found.", "red");
        return;
    }

    if (!c.name || !c.email || !c.phone) {
        showMessage("Please complete all fields.", "red");
        return;
    }

    customers[index] = c;
    saveData();
    viewCustomers();
    showMessage("Customer updated successfully.", "green");
}

function deleteCustomer() {
    const id = document.getElementById("customerId").value.trim();

    if (!id) {
        showMessage("Enter a Customer ID first.", "red");
        return;
    }

    const index = customers.findIndex(x => x.id.toLowerCase() === id.toLowerCase());

    if (index === -1) {
        showMessage("Customer ID not found.", "red");
        return;
    }

    if (confirm("Delete this customer?")) {
        customers.splice(index, 1);
        saveData();
        viewCustomers();
        clearForm();
        showMessage("Customer deleted successfully.", "green");
    }
}

function searchCustomer(c) {
    document.getElementById("customerId").value = c.id;
    document.getElementById("customerName").value = c.name;
    document.getElementById("customerEmail").value = c.email;
    document.getElementById("customerPhone").value = c.phone;
}

function viewCustomers() {
    const search = document.getElementById("searchBox").value.toLowerCase();
    const table = document.getElementById("customerTable");

    const filtered = customers.filter(c =>
        c.id.toLowerCase().includes(search) ||
        c.name.toLowerCase().includes(search) ||
        c.email.toLowerCase().includes(search) ||
        c.phone.toLowerCase().includes(search)
    );

    table.innerHTML = filtered.map(c => `
        <tr>
            <td>${escapeHTML(c.id)}</td>
            <td>${escapeHTML(c.name)}</td>
            <td>${escapeHTML(c.email)}</td>
            <td>${escapeHTML(c.phone)}</td>
            <td class="actions">
                <button onclick='searchCustomer(${JSON.stringify(c)})'>VIEW/EDIT</button>
                <button class="danger" onclick='quickDelete("${escapeHTML(c.id)}")'>DELETE</button>
            </td>
        </tr>
    `).join("");

    if (filtered.length === 0) {
        table.innerHTML = '<tr><td colspan="5" style="text-align:center">No customers found.</td></tr>';
    }
}

function quickDelete(id) {
    if (confirm("Delete customer " + id + "?")) {
        customers = customers.filter(c => c.id !== id);
        saveData();
        viewCustomers();
        clearForm();
        showMessage("Customer deleted successfully.", "green");
    }
}

function clearForm() {
    document.getElementById("customerId").value = "";
    document.getElementById("customerName").value = "";
    document.getElementById("customerEmail").value = "";
    document.getElementById("customerPhone").value = "";
}

function showMessage(text, color) {
    const msg = document.getElementById("message");
    msg.textContent = text;
    msg.style.color = color;
    setTimeout(() => msg.textContent = "", 2500);
}

function escapeHTML(value) {
    return String(value).replace(/[&<>"']/g, ch => ({
        "&":"&amp;", "<":"&lt;", ">":"&gt;", '"':"&quot;", "'":"&#039;"
    }[ch]));
}
</script>
</body>
</html>
