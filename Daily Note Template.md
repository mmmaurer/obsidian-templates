---
tags:
  - daily-note
---
<%*
/* ====== CONFIG ====== */
// path to daily notes folder, relative to vault root, with trailing slash
// Assumes a year-based subfolder structure, e.g. "Daily Notes/2025/"
const DAILY_FOLDER = "Daily Notes/" + tp.date.now("YYYY") + "/"; 
const FILENAME_FORMAT = "YYYY-MM-DD";         // your daily note filename format. I use ISO (2024-06-15.md) to keep files in order
// Headings of sections to carry over from yesterday's note
const HEADINGS = ["TODOs", "Keep in mind", "Braindump"]; // sections to carry over
/* ===================== */

// Resolve last daily note
const folderWithSlash = DAILY_FOLDER.endsWith("/") ? DAILY_FOLDER : DAILY_FOLDER + "/";
const momentLib =
  (typeof app !== "undefined" && app.moment) ||
  (typeof window !== "undefined" && window.moment) ||
  null;
let yName = "";
let yPath = "";
let yTFile = null;

if (momentLib) {
  const MAX_LOOKBACK_DAYS = 14;
  let cursor = momentLib().subtract(1, "day");
  for (let i = 0; i < MAX_LOOKBACK_DAYS; i++) {
    yName = cursor.format(FILENAME_FORMAT) + ".md";
    yPath = folderWithSlash + yName;
    yTFile = tp.file.find_tfile(yPath);
    if (yTFile) break;
    cursor = cursor.subtract(1, "day");
  }
}

if (!yTFile) {
  yName = tp.date.yesterday(FILENAME_FORMAT) + ".md";
  yPath = folderWithSlash + yName;
  yTFile = tp.file.find_tfile(yPath);
}

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
