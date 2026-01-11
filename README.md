Social Media Dashboard

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Social Media Dashboard</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Chart.js CDN -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
:root {
  --bg: #0f172a;
  --card: #1e293b;
  --accent: #38bdf8;
  --text: #e5e7eb;
  --muted: #94a3b8;
}

* {
  box-sizing: border-box;
  font-family: Arial, Helvetica, sans-serif;
}

body {
  margin: 0;
  background: linear-gradient(135deg, #020617, #0f172a);
  color: var(--text);
}

.dashboard {
  max-width: 1166px;
  margin: auto;
  padding: 16px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.status {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
}

.status span {
  width: 10px;
  height: 10px;
  background: #22c55e;
  border-radius: 50%;
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0% { opacity: 0.4; }
  50% { opacity: 1; }
  100% { opacity: 0.4; }
}

.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 12px;
  margin-bottom: 16px;
}

.card {
  background: var(--card);
  border-radius: 16px;
  padding: 16px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.3);
}

.card h3 {
  margin: 0;
  font-size: 14px;
  color: var(--muted);
}

.card .value {
  font-size: 28px;
  margin-top: 8px;
  color: var(--accent);
}

.main {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 16px;
}

.panel {
  background: var(--card);
  border-radius: 16px;
  padding: 16px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 12px;
}

th, td {
  padding: 10px;
  text-align: left;
  font-size: 14px;
}

th {
  cursor: pointer;
  color: var(--accent);
}

tr:nth-child(even) {
  background: rgba(255,255,255,0.03);
}

button {
  background: var(--accent);
  border: none;
  padding: 8px 12px;
  border-radius: 8px;
  color: #020617;
  cursor: pointer;
  font-weight: bold;
}

button:hover {
  opacity: 0.9;
}
</style>
</head>

<body>
<div class="dashboard">
  <header>
    <h2>Social Media Dashboard</h2>
    <div class="status">
      <span></span> Live
    </div>
  </header>

  <section class="cards">
    <div class="card">
      <h3>Total Followers</h3>
      <div class="value" id="followers">0</div>
    </div>
    <div class="card">
      <h3>Engagement Rate</h3>
      <div class="value" id="engagement">0%</div>
    </div>
    <div class="card">
      <h3>Posts This Month</h3>
      <div class="value" id="posts">0</div>
    </div>
    <div class="card">
      <h3>Reach</h3>
      <div class="value" id="reach">0</div>
    </div>
  </section>

  <section class="main">
    <div class="panel">
      <h3>Growth Overview</h3>
      <canvas id="growthChart"></canvas>
    </div>

    <div class="panel">
      <h3>Platform Metrics</h3>
      <button onclick="exportCSV()">Export CSV</button>
      <table id="dataTable">
        <thead>
          <tr>
            <th onclick="sortTable(0)">Platform</th>
            <th onclick="sortTable(1)">Followers</th>
            <th onclick="sortTable(2)">Engagement %</th>
          </tr>
        </thead>
        <tbody>
          <tr><td>Instagram</td><td>12000</td><td>4.5</td></tr>
          <tr><td>Facebook</td><td>9000</td><td>3.2</td></tr>
          <tr><td>Twitter</td><td>7000</td><td>2.8</td></tr>
          <tr><td>LinkedIn</td><td>5000</td><td>3.9</td></tr>
        </tbody>
      </table>
    </div>
  </section>
</div>

<script>
const counters = [
  { id: "followers", value: 33000 },
  { id: "engagement", value: 3.6, suffix: "%" },
  { id: "posts", value: 48 },
  { id: "reach", value: 120000 }
];

counters.forEach(counter => {
  let start = 0;
  const el = document.getElementById(counter.id);
  const interval = setInterval(() => {
    start += counter.value / 40;
    if (start >= counter.value) {
      el.textContent = counter.value + (counter.suffix || "");
      clearInterval(interval);
    } else {
      el.textContent = Math.floor(start) + (counter.suffix || "");
    }
  }, 30);
});

const ctx = document.getElementById('growthChart');
new Chart(ctx, {
  type: 'line',
  data: {
    labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
    datasets: [{
      label: 'Followers Growth',
      data: [22000, 24000, 26000, 28500, 30500, 33000],
      borderWidth: 2,
      tension: 0.4
    }]
  },
  options: {
    responsive: true,
    plugins: { legend: { display: false } }
  }
});

function sortTable(col) {
  const table = document.getElementById("dataTable");
  const rows = Array.from(table.rows).slice(1);
  const asc = table.dataset.sort !== "asc";
  rows.sort((a, b) => {
    const A = a.cells[col].innerText;
    const B = b.cells[col].innerText;
    return asc ? A.localeCompare(B, undefined, {numeric:true}) :
                 B.localeCompare(A, undefined, {numeric:true});
  });
  table.dataset.sort = asc ? "asc" : "desc";
  rows.forEach(r => table.tBodies[0].appendChild(r));
}

function exportCSV() {
  let csv = "Platform,Followers,Engagement\n";
  document.querySelectorAll("#dataTable tbody tr").forEach(row => {
    csv += [...row.children].map(td => td.innerText).join(",") + "\n";
  });
  const blob = new Blob([csv], { type: "text/csv" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "social_dashboard.csv";
  link.click();
}
</script>
</body>
</html>