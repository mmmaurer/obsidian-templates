---
tags:
  - meeting-note
  - phd
date:

---
<%*
const folder = "Meeting Notes/PhD Meetings/";
const files = app.vault.getFiles().filter(f => f.path.startsWith(folder));
const dateRegex = /^(\d{2})\.(\d{2})\.(\d{4})$/;

let latestFile = null;
let latestDate = null;

for (let f of files) {
  const m = f.basename.match(dateRegex);
  if (!m) continue;
  const [_, dd, mm, yyyy] = m;
  const d = new Date(`${yyyy}-${mm}-${dd}`);
  if (!latestDate || d > latestDate) {
    latestDate = d;
    latestFile = f;
  }
}

if (!latestFile) {
  tR += "No dated notes found";
} else {
  const content = await app.vault.read(latestFile);

  // Helper to extract an H1 section by title until the next H1 or EOF
  const getSection = (text, title) => {
    const re = new RegExp(`^#\\s+${title.replace(/[.*+?^${}()|[\\]\\\\]/g, '\\$&')}[^\n]*\\n([\\s\\S]*?)(?=^#\\s|\\Z)`, "m");
    const m = text.match(re);
    return m ? m[1].trim() : null;
  };

  const agendaBody = getSection(content, "Meeting Notes/Agenda");
  const nextBody   = getSection(content, "Next Meeting");

  if (!agendaBody && !nextBody) {
    tR += `Sections "# Meeting Notes/Agenda" and "# Next Meeting" not found in ${latestFile.basename}`;
  } else {
    const heading = `# Last Meeting ([[${latestFile.basename}]])`;

    const parts = [];
    if (agendaBody) parts.push(agendaBody);
    // Append Next Meeting under a subheading at the end of the combined section
    if (nextBody) parts.push(`\n# Meeting Notes/Agenda\n${nextBody}`);

    const combined = parts.join("\n").trim();
    tR += `${heading}\n${combined}`.trimEnd();
  }
}
%>
# Next Meeting
- 
# Keep in Mind
- 