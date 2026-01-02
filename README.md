<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>UKCR Discharge Logging System</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    body {
      margin: 0;
      padding: 24px;
      background: #0e1117;
      color: #e6e6e6;
      font-family: Arial, Helvetica, sans-serif;
    }

    .container {
      max-width: 800px;
      margin: auto;
    }

    h1 {
      margin-bottom: 16px;
    }

    .card {
      background: #161b22;
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 10px 30px rgba(0,0,0,.4);
    }

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
      font-size: 14px;
      box-sizing: border-box;
    }

    textarea {
      min-height: 90px;
      resize: vertical;
    }

    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
    }

    @media (max-width: 650px) {
      .grid {
        grid-template-columns: 1fr;
      }
    }

    .actions {
      display: flex;
      gap: 10px;
      margin-top: 20px;
    }

    button {
      padding: 10px 14px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
      font-weight: bold;
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

    .note {
      margin-top: 10px;
      font-size: 12px;
      color: #9da5b4;
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
        <input name="rgrade" placeholder="R1 / R2 / etc">

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
          <button type="submit" class="primary">Submit to Discord</button>
          <button type="button" class="secondary" id="previewBtn">Preview</button>
          <button type="reset" class="secondary">Clear</button>
        </div>

        <div id="status" class="status">Status: Ready</div>

        <div class="note">
          ⚠️ Webhook is stored client-side. Regenerate it if this file is shared.
        </div>

      </form>
    </div>
  </div>

  <script>
    /*********************************************************
     * DISCORD WEBHOOK (CHANGE HERE IF NEEDED)
     *********************************************************/
    const DISCORD_WEBHOOK_URL =
      "https://discord.com/api/webhooks/1456747189765410840/hcaWfGVQ0hdnx40LYyzc2C-EYGELw5PL8dygWyCjau3pJvCcAzrInIqnZQCswgSQRi9r";

    const form = document.getElementById("logForm");
    const statusBox = document.getElementById("status");
    const previewBtn = document.getElementById("previewBtn");

    function collectData() {
      const f = new FormData(form);
      return {
        date: f.get("date"),
        officers: f.get("officers"),
        dfi: f.get("dfi"),
        rgrade: f.get("rgrade"),
        tactics: f.get("tactics"),
        ofc: f.get("ofc"),
        tfc: f.get("tfc"),
        sfc: f.get("sfc"),
        whenwhy: f.get("whenwhy"),
        firearm: f.get("firearm")
      };
    }

    function formatMessage(d) {
      return [
        "**UKCR Discharge Logging System**",
        "",
        `**Date:** ${d.date}`,
        `**Officers on scene:**\n${d.officers}`,
        `**DFI (Declared firearms incident):** ${d.dfi}`,
        `**DFI as R Grade:** ${d.rgrade || "-"}`,
        `**Tactics used:**\n${d.tactics || "-"}`,
        `**On scene OFC:** ${d.ofc || "-"}`,
        `**On Scene TFC:** ${d.tfc || "-"}`,
        `**On Scene SFC:** ${d.sfc || "-"}`,
        `**When and why?:**\n${d.whenwhy}`,
        `**What firearm was discharged?:** ${d.firearm}`
      ].join("\n");
    }

    previewBtn.onclick = () => {
      const msg = formatMessage(collectData());
      statusBox.textContent = "PREVIEW:\n\n" + msg;
    };

    form.onsubmit = async (e) => {
      e.preventDefault();

      const message = formatMessage(collectData());

      if (message.length > 1900) {
        statusBox.textContent = "Error: Message too long for Discord.";
        return;
      }

      statusBox.textContent = "Sending to Discord...";

      try {
        const res = await fetch(DISCORD_WEBHOOK_URL, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ content: message })
        });

        if (!res.ok) throw new Error("Discord error");

        statusBox.textContent = "✅ Successfully sent to Discord";
        form.reset();
      } catch (err) {
        statusBox.textContent = "❌ Failed to send message";
      }
    };
  </script>
</body>
</html>
