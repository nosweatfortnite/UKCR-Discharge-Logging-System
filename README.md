<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>UKCR Discharge Logging System</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    body {
      background: #0e1117;
      color: #e6e6e6;
      font-family: Arial, Helvetica, sans-serif;
      padding: 24px;
    }

    .container {
      max-width: 800px;
      margin: auto;
    }

    .card {
      background: #161b22;
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 10px 30px rgba(0,0,0,.4);
    }

    h1 { margin-bottom: 16px; }

    label {
      display: block;
      margin-top: 14px;
      margin-bottom: 6px;
      font-weight: bold;
    }

    input, textarea, select {
      width: 100%;
      padding: 10px;
      border-radius: 8px;
      border: 1px solid #30363d;
      background: #0d1117;
      color: #e6e6e6;
      box-sizing: border-box;
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

    .primary {
      background: #5865F2;
      color: white;
    }

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
      white-space: pre-wrap;
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
            <label>Date</label>
            <input type="date" name="date" required>
          </div>
          <div>
            <label>DFI (Declared Firearms Incident)</label>
            <select name="dfi" required>
              <option value="" disabled selected>Select</option>
              <option>Yes</option>
              <option>No</option>
              <option>Unknown</option>
            </select>
          </div>
        </div>

        <label>Officers on scene</label>
        <textarea name="officers" required></textarea>

        <label>DFI as R Grade</label>
        <input name="rgrade">

        <label>Tactics used</label>
        <textarea name="tactics"></textarea>

        <div class="grid">
          <div>
            <label>On scene OFC</label>
            <input name="ofc">
          </div>
          <div>
            <label>On Scene TFC</label>
            <input name="tfc">
          </div>
        </div>

        <label>On Scene SFC</label>
        <input name="sfc">

        <label>When and why?</label>
        <textarea name="whenwhy" required></textarea>

        <label>What firearm was discharged?</label>
        <input name="firearm" required>

        <div class="actions">
          <button class="primary" type="submit">Submit to Discord</button>
          <button class="secondary" type="reset">Clear</button>
        </div>

        <div id="status" class="status">Status: Ready</div>
      </form>
    </div>
  </div>

  <script>
    /*************************************************
     * DISCORD WEBHOOK (CHANGE HERE IF NEEDED)
     *************************************************/
    const DISCORD_WEBHOOK_URL =
      "https://discord.com/api/webhooks/1456747189765410840/hcaWfGVQ0hdnx40LYyzc2C-EYGELw5PL8dygWyCjau3pJvCcAzrInIqnZQCswgSQRi9r";

    const form = document.getElementById("logForm");
    const statusBox = document.getElementById("status");

    form.onsubmit = async (e) => {
      e.preventDefault();

      const f = new FormData(form);

      const embed = {
        title: "UKCR Discharge Logging System",
        color: 0x5865F2,
        timestamp: new Date().toISOString(),
        fields: [
          { name: "Date", value: f.get("date") || "-", inline: true },
          { name: "DFI Declared", value: f.get("dfi") || "-", inline: true },
          { name: "Officers on scene", value: f.get("officers") || "-" },
          { name: "DFI as R Grade", value: f.get("rgrade") || "-" },
          { name: "Tactics used", value: f.get("tactics") || "-" },
          { name: "On scene OFC", value: f.get("ofc") || "-", inline: true },
          { name: "On Scene TFC", value: f.get("tfc") || "-", inline: true },
          { name: "On Scene SFC", value: f.get("sfc") || "-" },
          { name: "When and why?", value: f.get("whenwhy") || "-" },
          { name: "Firearm discharged", value: f.get("firearm") || "-" }
        ]
      };

      statusBox.textContent = "Sending embed to Discord…";

      try {
        const res = await fetch(DISCORD_WEBHOOK_URL, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ embeds: [embed] })
        });

        if (!res.ok) throw new Error();

        statusBox.textContent = "✅ Embed sent successfully";
        form.reset();
      } catch {
        statusBox.textContent = "❌ Failed to send embed";
      }
    };
  </script>
</body>
</html>
