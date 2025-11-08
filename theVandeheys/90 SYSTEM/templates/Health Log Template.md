<% "---" %>
<%*
let log = console.log
let title = tp.file.title
let now = tp.date.now("YYYY-MM-DDTHH:mm:ss")
let s = await tp.system.prompt("enter date time",now,false)
let newDateTime = moment(s)

	title =  newDateTime.format("YYYY-MM-DD-HHmm")
	now = newDateTime.format("YYYY-MM-DDTHH:mm:ss")
	log(title,now)
	await tp.file.move(`/healthLog/healthLogEntries/${title}`)

%>

tags:
  - healthLog
datetime: <% now %>
weight:
bs:
bp:
pulse:
systolic:
diastolic:
note:

<% "---" %>

![[healthLog]]


