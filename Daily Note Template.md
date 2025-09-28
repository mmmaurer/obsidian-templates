---
tags:
  - daily-note
---
<%*
/* ====== CONFIG ====== */
const DAILY_FOLDER = "Daily Notes/2025/";     // trailing slash ok
const FILENAME_FORMAT = "YYYY-MM-DD";         // your daily note filename format
const HEADINGS = ["TODOs", "Keep in mind", "Braindump"]; // sections to carry over
/* ===================== */

// Resolve yesterday's file
const yName = tp.date.yesterday(FILENAME_FORMAT) + ".md";
const yPath = DAILY_FOLDER.endsWith("/") ? DAILY_FOLDER + yName : DAILY_FOLDER + "/" + yName;
const yTFile = tp.file.find_tfile(yPath);

let yContent = "";
if (yTFile) {
  yContent = await app.vault.read(yTFile);
  // normalize line endings to keep regex simple
  yContent = yContent.replace(/\r\n/g, "\n");
}

// Extract the FIRST occurrence of a heading (any level) until the next heading
function extractFirstSection(md, heading) {
  const esc = heading.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
  const re = new RegExp(
    // start-of-file or newline, then any heading level, the title, optional EOL,
// then capture everything until the next heading or end-of-file
    String.raw`(?:^|\n)#{1,6}\s*${esc}\s*(?:\n|$)([\s\S]*?)(?=\n#{1,6}\s|$)`,
    "i"
  );
  const m = md.match(re);
  return m ? (m[1] || "").trim() : "";
}

// Build today's note
let out = "";
// Always start a fresh Log (not carried over)
out += "# Log\n- \n";
for (const h of HEADINGS) {
  const text = extractFirstSection(yContent, h);
  out += `# ${h}\n`;
  if (text) out += text + "\n";
  else out += "\n";
}

tR += out;
%>