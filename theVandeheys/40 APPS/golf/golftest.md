---
key2: junk
---

timDiff:: 18.2
patDiff:: 30.18


```dataviewjs
let {timDiff, patDiff} = dv.current(); // Accessing frontmatter from the current file
let allRounds = dv.pages('"golf/rounds"');
let timRounds = allRounds.where(r => r.player == "Tim").sort(r => r.date, 'desc');
let patRounds = allRounds.where(r => r.player == "Pat").sort(r => r.date, 'desc');

let last = timRounds[0];
let last20 = JSON.parse(last.last20);
let prev19 = last20.slice(1);

let output = [];
output.push("Tim's Last Round Player: " + last.player);
output.push("Previous 19 (from last20): " + JSON.stringify(prev19));
output.push("Last 20 (full array): " + JSON.stringify(last20));
output.push("Tim's Last Round Rank: " + last.rank);

// --- Accessing Frontmatter of the Current Note (Before Modification) ---
output.push("\n--- Current Note's Frontmatter (Direct Access - Before Modification) ---");
output.push(JSON.stringify(dv.current().file.frontmatter, null, 2));

// --- Using app.fileManager.processFrontMatter to SET/MODIFY Frontmatter ---
output.push("\n--- Modifying Frontmatter using app.fileManager.processFrontMatter ---");
let currentFrontmatterAfterModification = {}; // Object to hold frontmatter after modification

await app.fileManager.processFrontMatter(dv.current().file.path, (frontmatter) => {
    // Inside this callback, 'frontmatter' is the object you can modify.
    // Changes to 'frontmatter' here will be written back to the file.

    // Set a new key with a value
    frontmatter['last_updated_by_dataview'] = dv.date('now').toFormat('yyyy-MM-dd HH:mm:ss');
    frontmatter['some_new_property'] = 'This is a new value set by DataviewJS!';
    
    // You can also modify existing keys:
    // frontmatter['existing_key'] = 'new_value_for_existing_key';

    output.push('Inside processFrontMatter callback: Keys are being set.');
    
    // Optionally, capture the modified frontmatter to display immediately
    Object.assign(currentFrontmatterAfterModification, frontmatter);
});

// --- Display Frontmatter After Modification ---
output.push("\n--- Frontmatter of Current Note (After Modification by DataviewJS) ---");
output.push(JSON.stringify(currentFrontmatterAfterModification, null, 2));

// If you want to confirm the change by re-reading from the file:
// Note: dv.current().file.frontmatter might not update immediately in the same rendering cycle
// for display, but the file itself will be updated.
// output.push("\n--- Frontmatter from dv.current() after file write (may not update instantly) ---");
// output.push(JSON.stringify(dv.current().file.frontmatter, null, 2));


dv.paragraph(output.join('\n'));
```

// in vault at scripts/coolString.js
class CoolString {
    coolify(s) {
        return `😎 ${s} 😎`
    }
}


// dataviewjs block in *.md
```dataviewjs
const {CoolString} = await cJS()
dv.list(dv.pages('"INBOX"').file.name.map(n => CoolString.coolify(n)))
```

// templater template
<%*
const {CoolString} = await cJS();
tR += CoolString.coolify(tp.file.title);
%>

