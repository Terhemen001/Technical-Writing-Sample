
---


# Beginner’s Guide to Performance Monitoring in Node.js:

## Learn how to identify slow requests, track performance bottlenecks, and monitor a Node.js application using practical, code-heavy examples.

Performance issues rarely announce themselves politely. A Node.js application may work perfectly in development, only to become slow and unpredictable under real user traffic. Requests begin to lag, APIs time out, and users quietly leave.

This guide walks through **solving a real performance problem in a Node.js app** by introducing **performance monitoring**. We’ll focus on **what to monitor**, **why it matters**, and **how to instrument your code** in a way that reflects how AppSignal-style observability works — simple, practical, and code-first.

---

## The Real Problem: “My Node.js App Is Slow, But I Don’t Know Why”

As a beginner, performance debugging often looks like this:

- Requests randomly take 2–5 seconds
- No errors are thrown
- CPU and memory look “fine”
- Logs don’t show anything obvious

The issue isn’t the absence of logs — it’s the absence of **timing data**.

Performance monitoring answers questions like:
- Which route is slow?
- Which function is the bottleneck?
- Is the database or external API the problem?

Let’s solve this step by step.

---

## Step 1: A Simple Node.js App (Baseline)

We’ll start with a small Express app that simulates a slow endpoint.

```bash
npm init -y
npm install express
```

```js
// app.js
const express = require("express");
const app = express();

function slowOperation() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("done"), 2000);
  });
}

app.get("/users", async (req, res) => {
  await slowOperation();
  res.json({ users: [] });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

When you hit `/users`, the response takes ~2 seconds.
But in a real app, delays are rarely this obvious.

---

## Step 2: Why Logs Are Not Enough

A common beginner approach:

```js
console.log("Fetching users...");
```

Logs tell you *what happened*, not *how long it took*.
Performance monitoring requires **measuring duration**.

This is where instrumentation comes in.

---

## Step 3: Measuring Performance with Manual Instrumentation

Let’s manually instrument the slow part using timestamps.

```js
app.get("/users", async (req, res) => {
  const start = Date.now();

  await slowOperation();

  const duration = Date.now() - start;
  console.log(`GET /users took ${duration}ms`);

  res.json({ users: [] });
});
```

This is your first performance metric:

* Route name: `/users`
* Duration: `~2000ms`

This pattern is the foundation of all performance monitoring systems, including AppSignal.

---

## Step 4: AppSignal-Style Tracing (Conceptual)

In AppSignal-style monitoring, instead of manually logging durations, you wrap code in **traces**.

Conceptually, it looks like this:

```js
function trace(name, fn) {
  const start = Date.now();

  return Promise.resolve(fn()).finally(() => {
    const duration = Date.now() - start;
    console.log(`[TRACE] ${name} took ${duration}ms`);
  });
}
```

Now rewrite your route:

```js
app.get("/users", async (req, res) => {
  await trace("GET /users", async () => {
    await slowOperation();
  });

  res.json({ users: [] });
});
```

This pattern:

* Names the operation
* Measures duration
* Keeps performance logic separate from business logic

This is exactly how production-grade monitoring tools work internally.

---

## Step 5: Finding the Real Bottleneck

Let’s introduce multiple operations to simulate a real app.

```js
function fetchFromDatabase() {
  return new Promise((resolve) => {
    setTimeout(() => resolve([]), 1200);
  });
}

function fetchFromAPI() {
  return new Promise((resolve) => {
    setTimeout(() => resolve({}), 800);
  });
}
```

Now instrument each step:

```js
app.get("/users", async (req, res) => {
  await trace("fetch_users", async () => {
    await trace("database_query", fetchFromDatabase);
    await trace("external_api_call", fetchFromAPI);
  });

  res.json({ users: [] });
});
```

Console output:

```
[TRACE] database_query took 1203ms
[TRACE] external_api_call took 804ms
[TRACE] fetch_users took 2010ms
```

Now you **know exactly where time is being spent**.

---

## Step 6: Solving the Performance Issue

Now that we have visibility, optimization becomes straightforward.

### Problem:

Database and API calls are executed sequentially.

### Solution:

Run them in parallel.

```js
app.get("/users", async (req, res) => {
  await trace("fetch_users", async () => {
    await Promise.all([
      trace("database_query", fetchFromDatabase),
      trace("external_api_call", fetchFromAPI)
    ]);
  });

  res.json({ users: [] });
});
```

New output:

```
[TRACE] database_query took 1201ms
[TRACE] external_api_call took 802ms
[TRACE] fetch_users took 1210ms
```



---

## Step 7: Monitoring in Production (What AppSignal Automates)

In real-world apps, AppSignal handles:

* Automatic route tracing
* Database query timing
* External request monitoring
* Error correlation with slow traces

A production setup typically looks like:

```js
const Appsignal = require("@appsignal/nodejs");

const appsignal = new Appsignal({
  active: true,
  pushApiKey: process.env.APPSIGNAL_KEY
});
```

Once installed, Express routes are automatically monitored, and you add **custom traces only where needed**.

---

## Step 8: What Beginners Should Always Monitor

At minimum, track:

1. **Request duration**
2. **Database query time**
3. **External API latency**
4. **Slow background jobs**
5. **Error frequency + slow traces**

Performance monitoring is not about guessing — it’s about **evidence**.

---

* Performance problems exist even when apps “work”
* Logs alone don’t explain slowness
* Instrumentation gives visibility into execution time
* Tracing reveals real bottlenecks
* Monitoring turns optimization into a data-driven process

If you don’t measure performance, you’re debugging blind.

---

Performance monitoring is not an advanced skill — it’s a **basic survival tool** for Node.js developers. The earlier you add visibility, the easier scaling and debugging become.

---

