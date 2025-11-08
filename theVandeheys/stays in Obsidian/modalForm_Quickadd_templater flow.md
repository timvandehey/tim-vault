https://medium.com/@horaciochacn/streamlining-meeting-notes-in-obsidian-a-guide-to-modal-forms-templater-and-quickadd-83b318692ce4
#inbox 

New Meeting Form
This form captures:

Project name
Meeting name
Duration
Participants
Step 2: Create a Templater Template
Templater is a powerful plugin that allows for dynamic template creation. Here’s the template I use for meeting notes:

<%*
  const modalForm = app.plugins.plugins.modalforms.api;
  const result = await modalForm.openForm("Meeting");
-%>
---
tags:
  -  <%* result.get("Project")%>
date: <% tp.date.now("YYYY-MM-DD") %>
project: <% result.get("Project") %> 
type: Meeting
meetingType: <% result.get("Meeting Name") %>
people: <% result.get("Participants") %>
duration: <% result.get("Duration") %>
---
# <% result.get("Project") %>:   <% result.get("Meeting Name") %> - <% tp.date.now("MMMM Do YYYY") %>

## Agenda



## Meeting Notes

- 

## Key Decisions

- 

## Follow-up


## Next Meeting

**Date:** <% tp.date.now("YYYY-MM-DD", 7) %>
**Time:** <% tp.date.now("HH:mm") %>

## Related



<% await tp.file.rename(tp.date.now("YYYY-MM-DD")+" - "+result.get("Meeting Name")+" ")%>
This template:

Opens the Modal Form
Uses the input to populate metadata fields
Creates a structured note with sections for agenda, notes, decisions, and follow-ups
Automatically sets a date for the next meeting
Renames the file with the current date and meeting name
Step 3: Set Up QuickAdd
QuickAdd allows you to create custom commands in Obsidian. I’ve set up a command that:

Creates a new note
Applies the Templater template
The Result
Now, when I need to create a new meeting note, I simply:

Trigger the QuickAdd command
Fill out the Modal Form
Start taking notes in a perfectly formatted document
Here’s what the final result looks like:


New Meeting Note
