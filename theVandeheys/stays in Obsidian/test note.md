---
project:
  name: My Awesome Project
  status: In Progress
  details:
    due_date: 2025-07-31
    priority: High
tasks:
  - name: Research
    status: Done
  - name: Implement Feature X
    status: Completed
junk3:
  - "130"
  - "98"
junk4: 120/83
---
junk5:: hello

junk6:: `= this.junk1 + "/" + this.junk2`
junk7:: `= split(this.junk4,"/")[0]`
junk8:: `= split(this.junk4,"/")[1]`

```cardlink
url: https://obsidian.md/
title: "Obsidian"
description: "Obsidian: A knowledge base that works on local Markdown files."
host: obsidian.md
favicon: https://obsidian.md/favicon.ico
image: https://obsidian.md/images/banner.png
```


```dataview
table without id
(junk1 + "/" + junk2) as BP, (junk3[0]+"/"+junk3[1]) as bp2,
split(junk4,"/")[0] as Systolic, split(junk4,"/")[1] as Diastolic, junk6
from "test note"
```


```dataview
TABLE project.name AS "Project", project.status AS "Status", project.details.due_date AS "Due Date"
FROM ""
WHERE project
SORT project.details.due_date ASC
```


Meta Bind has an in plugin help page. `BUTTON[myButton-id]` Isn't that cool?

```meta-bind-button
label: myButton
icon: ""
style: destructive
class: ""
cssStyle: ""
backgroundImage: ""
tooltip: ""
id: myButton-id
hidden: false
actions:
  - type: command
    command: workspace:open-in-new-window

```

```button
name ButtKicker
type link
action data
```
^button-m2li

> [!faq]- Hidden
> hidden stuff here

> [!faq]- Are callouts foldable?
> Yes! In a foldable callout, the contents are hidden when the callout is collapsed.

> [!note]+ [[kiosk status]]

```dataviewjs
// For a single note, assuming 'dv.current()' refers to the current note
const projectName = dv.current().project.name;
const projectStatus = dv.current().project.status;
const dueDate = dv.current().project.details.due_date;

console.log(`Project Name: ${projectName}`);
console.log(`Project Status: ${projectStatus}`);
console.log(`Due Date: ${dueDate}`);

// Iterating through tasks
dv.current().tasks.forEach(task => {
    console.log(`Task: ${task.name}, Status: ${task.status}`);
});

// Example of a DataviewJS query to list projects
dv.queryMarkdown(`
TABLE project.name AS "Project", project.status AS "Status", project.details.due_date AS "Due Date"
FROM ""
WHERE project
SORT project.details.due_date ASC
`);
```

```dataviewjs
dv.el("div",`\`\`\`meta-bind-button
label: myButton
icon: ""
style: destructive
class: ""
cssStyle: ""
backgroundImage: ""
tooltip: ""
id: myButton-id
hidden: false
actions:
  - type: command
    command: workspace:open-in-new-window

\`\`\``)
```

