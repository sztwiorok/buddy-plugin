# Sandbox Agent Examples

## Table of Contents
- [Example 1: Simple Script Task](#example-1-simple-script-task)
- [Example 2: Test Generation](#example-2-test-generation)
- [Example 3: Multi-Agent Code Review](#example-3-multi-agent-code-review)

---

## Example 1: Simple Script Task

**Goal:** Delegate a simple task - list files and save to file. Quick test to verify delegation works.

### Steps

```bash
# 1. Create sandbox
bdy sandbox create -i test-delegate --resources 2x4 --wait-for-configured

# 2. Delegate simple task
bdy sandbox exec command test-delegate "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"List all files in /etc directory and save the output to /tmp/etc-files.txt\""

# 3. Get command ID
bdy sandbox exec list test-delegate
# Note the command ID (e.g., cmd-abc123)

# 4. Monitor progress
bdy sandbox exec status test-delegate <cmd-id>
bdy sandbox exec logs test-delegate <cmd-id>

# 5. Wait for completion
bdy sandbox exec logs test-delegate <cmd-id> --wait

# 6. Verify result file was created
bdy sandbox exec command test-delegate "cat /tmp/etc-files.txt" --wait

# 7. Cleanup
bdy sandbox destroy test-delegate
```

### Expected Result

File `/tmp/etc-files.txt` contains list of files from /etc directory.

---

## Example 2: Test Generation

**Goal:** Delegate test generation for a Node.js application. Demonstrates setup, code copy, and result verification.

### Steps

```bash
# 1. Create sandbox with Node.js
bdy sandbox create -i test-gen --resources 4x8 \
  --install-command "curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs" \
  --wait-for-configured

# 2. Create simple test app locally
mkdir -p /tmp/test-app
cat > /tmp/test-app/math.js << 'EOF'
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }
function multiply(a, b) { return a * b; }
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}
module.exports = { add, subtract, multiply, divide };
EOF

cat > /tmp/test-app/package.json << 'EOF'
{"name": "test-app", "version": "1.0.0"}
EOF

# 3. Copy to sandbox
bdy sandbox cp --silent /tmp/test-app test-gen:/app

# 4. Delegate test generation
bdy sandbox exec command test-gen "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Analyze /app/math.js and create comprehensive unit tests using Jest. Install Jest as dev dependency, write tests to /app/math.test.js covering all functions including edge cases, run the tests and ensure they pass.\""

# 5. Get command ID and monitor
bdy sandbox exec list test-gen
bdy sandbox exec logs test-gen <cmd-id> --wait

# 6. Verify tests exist
bdy sandbox exec command test-gen "cat /app/math.test.js" --wait

# 7. Verify tests pass
bdy sandbox exec command test-gen "cd /app && npx jest" --wait

# 8. (Optional) Copy tests back to local
bdy sandbox cp test-gen:/app/math.test.js ./math.test.js --silent

# 9. Cleanup
bdy sandbox destroy test-gen
```

### Expected Result

- Tests in `/app/math.test.js` covering add, subtract, multiply, divide
- Edge case test for division by zero
- All tests pass when running `npx jest`

---

## Example 3: Multi-Agent Code Review

**Goal:** Get comprehensive code review from multiple perspectives. Each agent focuses on different aspect, main agent aggregates and prioritizes findings.

### Sample Code with Intentional Issues

```bash
# Create sample code to review (locally)
mkdir -p /tmp/review-app
cat > /tmp/review-app/api.js << 'EOF'
const express = require('express');
const app = express();

// User endpoint - has SQL injection vulnerability
app.get('/user', (req, res) => {
  const userId = req.query.id;
  const query = `SELECT * FROM users WHERE id = ${userId}`;
  db.query(query, (err, result) => {
    res.json(result);
  });
});

// Login endpoint - has hardcoded credentials
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  if (username === 'admin' && password === 'admin123') {
    res.json({ token: 'secret-token' });
  }
});

// Data processing - blocking operation
app.get('/process', (req, res) => {
  const data = [];
  for (let i = 0; i < 10000000; i++) {
    data.push(Math.random());
  }
  res.json({ count: data.length });
});

app.listen(3000);
EOF
```

### Steps

```bash
# 1. Create 3 sandboxes in parallel
bdy sandbox create -i reviewer-security --resources 2x4 --wait-for-configured &
bdy sandbox create -i reviewer-perf --resources 2x4 --wait-for-configured &
bdy sandbox create -i reviewer-quality --resources 2x4 --wait-for-configured &
wait

# 2. Copy code to all sandboxes
bdy sandbox cp --silent /tmp/review-app reviewer-security:/app &
bdy sandbox cp --silent /tmp/review-app reviewer-perf:/app &
bdy sandbox cp --silent /tmp/review-app reviewer-quality:/app &
wait

# 3. Delegate security review
bdy sandbox exec command reviewer-security "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Review /app/api.js for SECURITY issues only. Look for: SQL injection, XSS, authentication problems, hardcoded secrets, input validation, OWASP top 10. Write findings to /app/security-review.md with severity ratings (CRITICAL/HIGH/MEDIUM/LOW) and remediation suggestions.\""

# 4. Delegate performance review
bdy sandbox exec command reviewer-perf "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Review /app/api.js for PERFORMANCE issues only. Look for: blocking operations, memory leaks, inefficient algorithms, missing caching, connection pooling, async patterns. Write findings to /app/perf-review.md with severity ratings and optimization suggestions.\""

# 5. Delegate code quality review
bdy sandbox exec command reviewer-quality "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Review /app/api.js for CODE QUALITY issues only. Look for: error handling, code structure, naming conventions, missing validation, logging, best practices, maintainability. Write findings to /app/quality-review.md with severity ratings and improvement suggestions.\""

# 6. Get command IDs
bdy sandbox exec list reviewer-security
bdy sandbox exec list reviewer-perf
bdy sandbox exec list reviewer-quality

# 7. Wait for all reviews to complete
bdy sandbox exec logs reviewer-security <cmd-id> --wait
bdy sandbox exec logs reviewer-perf <cmd-id> --wait
bdy sandbox exec logs reviewer-quality <cmd-id> --wait

# 8. Collect all reviews
echo "=== SECURITY REVIEW ==="
bdy sandbox exec command reviewer-security "cat /app/security-review.md" --wait

echo "=== PERFORMANCE REVIEW ==="
bdy sandbox exec command reviewer-perf "cat /app/perf-review.md" --wait

echo "=== CODE QUALITY REVIEW ==="
bdy sandbox exec command reviewer-quality "cat /app/quality-review.md" --wait

# 9. Cleanup
bdy sandbox destroy reviewer-security
bdy sandbox destroy reviewer-perf
bdy sandbox destroy reviewer-quality
```

### Expected Results

**Security Review should find:**
- SQL injection vulnerability (CRITICAL) - line with `SELECT * FROM users WHERE id = ${userId}`
- Hardcoded credentials (CRITICAL) - `admin` / `admin123`
- Hardcoded token (HIGH) - `secret-token`
- Missing input validation (MEDIUM)

**Performance Review should find:**
- Blocking synchronous loop (HIGH) - 10M iterations blocking event loop
- Missing async patterns (MEDIUM)
- No connection pooling mentioned (MEDIUM)

**Code Quality Review should find:**
- Missing error handling (HIGH) - no try/catch, no error responses
- Missing input validation (MEDIUM)
- No logging (LOW)
- Missing middleware for body parsing (MEDIUM)

### Main Agent Aggregation

After collecting all reviews, main agent should:
1. Combine all findings into single list
2. De-duplicate overlapping issues
3. Sort by severity (CRITICAL first)
4. Present prioritized report to user with recommended fix order

**Priority order for this example:**
1. SQL Injection (CRITICAL) - immediate fix required
2. Hardcoded credentials (CRITICAL) - immediate fix required
3. Blocking loop (HIGH) - affects all users
4. Missing error handling (HIGH) - can crash app
5. Other MEDIUM/LOW issues
