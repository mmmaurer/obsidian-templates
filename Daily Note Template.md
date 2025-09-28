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

function filterIncompleteTasks(sectionText) {
  if (!sectionText) return "";

  const lines = sectionText.split("\n");
  const result = [];
  let pending = [];
  const doneStatuses = new Set(["x", "done", "complete", "completed", "checked"]);
  const cancelledStatuses = new Set(["-", "cancelled", "canceled", "cancel", "c"]);

  function flushPending() {
    if (!pending.length) return;
    if (!result.length) {
      while (pending.length && pending[0].trim() === "") pending.shift();
    }
    if (result.length && result[result.length - 1].trim() === "") {
      while (pending.length && pending[0].trim() === "") pending.shift();
    }
    if (pending.length) result.push(...pending);
    pending = [];
  }

  for (const line of lines) {
    const match = line.match(/^\s*[-*]\s+\[([^\]]*)\]/);
    if (!match) {
      pending.push(line);
      continue;
    }

    const status = match[1].trim().toLowerCase();
    if (doneStatuses.has(status) || cancelledStatuses.has(status)) {
      continue;
    }

    flushPending();
    result.push(line);
  }

  while (result.length && result[result.length - 1].trim() === "") {
    result.pop();
  }

  return result.join("\n");
}

// Build today's note
let out = "";
// Always start a fresh Log (not carried over)
out += "# Log\n- \n";
for (const h of HEADINGS) {
  const text = extractFirstSection(yContent, h);
  const sectionText = h === "TODOs" ? filterIncompleteTasks(text) : text;
  out += `# ${h}\n`;
  if (sectionText) out += sectionText + "\n";
  else out += "\n";
}

tR += out;
%>
