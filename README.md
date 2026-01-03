<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>UKCR Discharge Logging System</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />

  <style>
    body {
      background: #0e1117;
      color: #e6e6e6;
      font-family: Arial, Helvetica, sans-serif;
      padding: 24px;
    }

    .container {
      max-width: 820px;
      margin: auto;
    }

    .card {
      background: #161b22;
      border-radius: 12px;
      padding: 22px;
      box-shadow: 0 10px 30px rgba(0,0,0,.45);
    }

    h1 { margin-bottom: 16px; }

    label {
      display: block;
      margin-top: 14px;
      margin-bottom: 6px;
      font-weight: bold;
      font-size: 14px;
    }

    input, textarea, select {
      width: 100%;
      padding: 10px 12px;
      border-radius: 8px;
      border: 1px solid #30363d;
      background: #0d1117;
      color: #e6e6e6;
    }

    textarea { min-height: 90px; }

    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
    }

    @media (max-width: 650px) {
      .grid { grid-template-columns: 1fr; }
    }

    .actions {
      display: flex;
      gap: 10px;
      margin-top: 18px;
    }

    button {
      padding: 10px 14px;
      border-radius: 8px;
      border: none;
      font-weight: bold;
      cursor: pointer;
    }

    .primary { background: #5865F2; color: white; }
    .secondary {
      background: #21262d;
      color: #e6e6e6;
      border: 1px solid #30363d;
    }

    .status {
      margin-top: 14px;
      padding: 12px;
      background: #0d1117;
      border-radius: 8px;
      border: 1px solid #30363d;
      font-size: 13px;
    }
  </style>
</head>

<body>
<div class="container">
  <h1>UKCR Discharge Logging System</h1>

  <div class="card">
    <form id="logForm">
      <div class="grid">
        <div>
          <label>Date & Time of Discharge</label>
<input type="datetime-local" name="datetime" required>
        </div>
        <div>
          <label>DFI Declared</label>
          <select name="dfi" required>
            <option disabled selected>Select</option>
            <option>Yes</option>
            <option>No</option>
            <option>Unknown</option>
          </select>
        </div>
      </div>

      <label>Officers on scene</label>
      <textarea name="officers" required></textarea>

      <label>DFI R Grade</label>
      <input name="rgrade">

      <label>Tactics used</label>
      <textarea name="tactics"></textarea>

      <div class="grid">
        <input name="ofc" placeholder="OFC">
        <input name="tfc" placeholder="TFC">
      </div>

      <label>SFC</label>
      <input name="sfc">

      <label>When and why?</label>
      <textarea name="whenwhy" required></textarea>

      <label>Firearm discharged</label>
      <input name="firearm" required>

      <div class="actions">
        <button class="primary">Submit</button>
        <button class="secondary" type="reset">Clear</button>
      </div>

      <div id="status" class="status">Status: Ready</div>
    </form>
  </div>
</div>

<script>
const WEBHOOK = "YOUR_WEBHOOK_URL";
const form = document.getElementById("logForm");
const statusBox = document.getElementById("status");

form.onsubmit = async e => {
  e.preventDefault();
  const f = new FormData(form);

  const dfi = f.get("dfi");
  const color = dfi === "Yes" ? 0xED4245 : dfi === "Unknown" ? 0xFAA61A : 0x57F287;
  const ref = "UKCR-" + Date.now().toString().slice(-6);

  const embeds = [
    {
      title: "🔫 Firearm Discharge Logged",
      color,
      fields: [
        { name: "📅 Date", value: f.get("date"), inline: true },
        { name: "🚨 DFI", value: `**${dfi}**`, inline: true },
        { name: "🧾 Ref", value: ref, inline: true }
      ],
      footer: { text: "UKCR Discharge System" },
      timestamp: new Date().toISOString()
    },
    {
      color,
      fields: [
        { name: "👮 Officers", value: f.get("officers") },
        { name: "🎯 Tactics", value: f.get("tactics") || "—" },
        {
          name: "🧭 Command",
          value:
            `**OFC:** ${f.get("ofc") || "—"}\n` +
            `**TFC:** ${f.get("tfc") || "—"}\n` +
            `**SFC:** ${f.get("sfc") || "—"}`
        },
        { name: "⏱️ When & Why", value: f.get("whenwhy") },
        { name: "🔧 Firearm", value: `**${f.get("firearm")}**` }
      ]
    }
  ];

  statusBox.textContent = "Sending to Discord…";

  try {
    await fetch(WEBHOOK, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ embeds })
    });
    statusBox.textContent = "✅ Sent successfully";
    form.reset();
  } catch {
    statusBox.textContent = "❌ Failed to send";
  }
};
</script>
</body>
</html>
