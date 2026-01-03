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

    h1 {
      margin-bottom: 16px;
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
            <input type="date" name="date" required />
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
        <input name="rgrade" />

        <label>Tactics used</label>
        <textarea name="tactics"></textarea>

        <div class="grid">
          <div>
            <label>On scene OFC</label>
            <input name="ofc" />
          </div>
          <div>
            <label>On Scene TFC</label>
            <input name="tfc" />
          </div>
        </div>

        <label>On Scene SFC</label>
        <input name="sfc" />

        <label>When and why?</label>
        <textarea name="whenwhy" required></textarea>

        <label>What firearm was discharged?</label>
        <input name="firearm" required />

        <div class="actions">
          <button class="primary" type="submit">Submit Report</button>
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

    // Basic escape so user text can't mess with formatting too badly
    // (Discord doesn't render HTML, but this helps keep it neat.)
    const esc = (s) =>
      String(s ?? "")
        .replaceAll("&", "&amp;")
        .replaceAll("<", "&lt;")
        .replaceAll(">", "&gt;");

    form.onsubmit = async (e) => {
      e.preventDefault();

      const f = new FormData(form);

      const date = esc(f.get("date"));
      const dfi = esc(f.get("dfi"));
      const rgrade = esc(f.get("rgrade") || "—");
      const officers = esc(f.get("officers") || "—");
      const tactics = esc(f.get("tactics") || "—");
      const ofc = esc(f.get("ofc") || "—");
      const tfc = esc(f.get("tfc") || "—");
      const sfc = esc(f.get("sfc") || "—");
      const whenwhy = esc(f.get("whenwhy") || "—");
      const firearm = esc(f.get("firearm") || "—");

      // Colour changes based on DFI for quick visual severity
      const color =
        dfi === "Yes" ? 0xED4245 :     // Red
        dfi === "Unknown" ? 0xFAA61A : // Amber
        0x57F287;                      // Green

      // "HTML-style" layout using Discord markdown + dividers
      const reportBlock =
`**UKCR DISCHARGE REPORT**
━━━━━━━━━━━━━━━━━━
**📅 Date:** ${date}
**🚨 DFI Declared:** **${dfi}**
**📊 R Grade:** ${rgrade}

**👮 Officers on Scene**
${officers}

**🎯 Tactics Used**
${tactics}

**🧭 Command Structure**
• **OFC:** ${ofc}
• **TFC:** ${tfc}
• **SFC:** ${sfc}

**⏱️ When & Why**
${whenwhy}

**🔧 Firearm Discharged**
**${firearm}**
━━━━━━━━━━━━━━━━━━`;

      const embed = {
        author: {
          name: "UKCR Incident Reporting",
          icon_url: "https://cdn.discordapp.com/emojis/1098212251536922624.png"
        },
        title: "🔫 Firearm Discharge Report",
        description: reportBlock,
        color,
        timestamp: new Date().toISOString(),
        footer: {
          text: "Submitted via UKCR Discharge Logging System"
        }
      };

      statusBox.textContent = "Sending formatted report to Discord…";

      try {
        const res = await fetch(DISCORD_WEBHOOK_URL, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ embeds: [embed] })
        });

        if (!res.ok) throw new Error();

        statusBox.textContent = "✅ Report submitted successfully";
        form.reset();
      } catch (err) {
        statusBox.textContent = "❌ Failed to submit report";
      }
    };
  </script>
</body>
</html>
