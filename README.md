# 🔐 Session Hijacking Practical Demo (Beginner-Friendly)

A simple, practical, and beginner-friendly demonstration of how **Session Hijacking** works using a custom vulnerable PHP application running locally.

---

## ⚖️ Legal Disclaimer

This demo is strictly for **educational and ethical learning purposes**.

✔️ Use this only on systems you **own** or have **explicit permission** to test.  
❌ Unauthorized security testing is **illegal**.

---

# 🚀 Session Hijacking – Introduction

Session Hijacking is an attack where:

- A victim logs into a website  
- Server assigns a **session ID** (like a temporary login token)  
- Attacker steals/copies that ID  
- Attacker pastes it in their own browser  
- Attacker instantly becomes logged in *without username/password*

In this demo, we will create a small PHP app to clearly understand the concept.

---

# 🛠️ Setup Vulnerable PHP Website

Open terminal and run these commands:

```bash
mkdir session-demo
cd session-demo
```

---

## 📄 Create `login.php`

```php
<?php
// make cookie non-HttpOnly so demo can manipulate it via JS/DevTools
session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'domain' => 'localhost',
    'secure' => false,
    'httponly' => false,
    'samesite' => 'Lax'
]);
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // simple login action — no password check (lab-only!)
    $_SESSION['user'] = 'admin';
    header('Location: dashboard.php');
    exit();
}
?>
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Login</title></head>
<body>
  <h2>Login (Test)</h2>
  <form method="post">
    <button type="submit">Login as Admin</button>
  </form>
  <p><a href="dashboard.php">Go to Dashboard</a></p>
</body>
</html>
```

---

## 📄 Create `dashboard.php`

```php
<?php
session_start();
if (!isset($_SESSION['user'])) {
    echo '<p>Not logged in. <a href="login.php">Login</a></p>';
    exit();
}
?>
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Dashboard</title></head>
<body>
  <h2>Welcome, <?php echo htmlspecialchars($_SESSION['user']); ?>!</h2>
  <p>Your session id: <strong><?php echo session_id(); ?></strong></p>
  <p>Cookie value from $_COOKIE: <strong><?php echo isset($_COOKIE['PHPSESSID']) ? htmlspecialchars($_COOKIE['PHPSESSID']) : 'None'; ?></strong></p>
  <p><a href="logout.php">Logout</a></p>
</body>
</html>
```

---

## 📄 Create `logout.php`

```php
<?php
session_start();
$_SESSION = [];
if (ini_get("session.use_cookies")) {
    $params = session_get_cookie_params();
    setcookie(session_name(), '', time() - 42000,
        $params["path"], $params["domain"],
        $params["secure"], $params["httponly"]
    );
}
session_destroy();
header('Location: login.php');
exit();
?>
```

---

# ▶️ Run PHP Server

Inside the project folder:

```bash
php -S 0.0.0.0:8081
```

Your site is now running at:

```
http://localhost:8081
```

---

# 🧪 Practical Demonstration

## 🧑‍💻 Step 1 — Victim Login (Browser A)

Open:

```
http://localhost:8081/login.php
```

Click:

**Login as Admin**

You will be redirected to:

```
dashboard.php
```

Here you will see:

- **Welcome, admin!**
- **Session ID:** `<random_id>`

---

## 🔍 Step 2 — Copy Victim’s Session Cookie

Open DevTools → Application/Storage → Cookies → `http://localhost:8081`

Find cookie:

```
Name: PHPSESSID
Value: a1b2c3d4e5f6...
```

Copy this **value**.

---

# 🕵️ Step 3 — Attacker Browser Setup (Browser B)

1. Open a different browser (or incognito)
2. Install **Cookie-Editor** extension
3. Go to:

```
http://localhost:8081/login.php
```

4. Open Cookie Editor → Add Cookie:

```
Name: PHPSESSID
Value: (paste victim session ID)
Domain: localhost
Path: /
```

Click **Save**.

5. Refresh page.

---

# 🎉 Step 4 — Attack Successful

After refresh, attacker browser will show:

```
Welcome, admin!
```

Attacker is now logged into victim account **without credentials**.

This is **Session Hijacking** in the simplest form.

---

# 🔒 How To Prevent It (Best Practices)

- Use **HttpOnly cookies**
- Always use **HTTPS**
- Regenerate session ID after login
- Use **SameSite=strict**
- Set **short session expiry**
- Implement **IP/User-Agent binding**

---

# 👨‍💻 Author

Made with ❤️ by **The Techzeen**
