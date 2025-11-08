[https://forum.obsidian.md/t/i-created-the-date-picker-with-the-templater-plugin/55782/1](https://forum.obsidian.md/t/i-created-the-date-picker-with-the-templater-plugin/55782/1)
​

​

I created the date picker with the Templater plugin
Share & showcase
templater
read
5 min

reaty
Mar 2023
Hello, everyone.
I wanted to have a way to easily insert different dates into my notes, so I created a simple date picker using the Templater plugin and want to share it. The template is in the end of the post. Here how it works.

When you insert the template you get the suggestion window asking you to pick from the few options:

1
1
1187×485 31.1 KB
If you choose “write date” you will get a prompt asking you to write date in “DD MMMM YYYY” format. It’s actually not necessary to write the full date, you can write only part of it. For example, if you write “14” you will get a 14’th of the current month and the current year.

2
2
1024×396 25 KB
If you choose “today” or “tomorrow”, you will get today’s or tomorrow’s date, it’s pretty straightforward.

If you choose “> weekday” you will get another suggestion window asking you to pick the day of the week. As a result you will get the closest date of this weekday in the future. For example, if today is wednesday, and you pick “monday” you will get the next week’s monday.

3
3
1264×580 39.3 KB
If you choose “> calendar” you will be suggested to pick the year, then the month, then the date. You can choose between this year and 9 next. In the date list you can also see weekdays, and the lines added to the sundays to help visually separate weeks.

4
4
1132×912 68.3 KB
The result date will be inserted in the “YYYY-MM-DD” format. I use it, because it works with the dataview metadata. But you can change if, just change the “resultFormat” value at the top of the template.

Some other options to pick can be easily added to the template, but these are ones I personally find useful for now. Keep in mind that I’m not a programmer, just know a little bit of js. Also sorry if my English is bad.

Here is the template:

<%*
let resultDate = ""
let myformat = "DD MMMM YYYY"
let resultFormat = "YYYY-MM-DD"

let dateString = await tp.system.suggester(["(write date)", "today", "tomorrow", "> weekday", "> calendar"], ["(write date)", "today", "tomorrow", "> weekday", "> calendar"])
if (dateString != null) {

if (dateString == "today") {
	resultDate = tp.date.now()

}  else if (dateString == "tomorrow") {
	resultDate = tp.date.tomorrow()

}  else if (dateString == "> weekday") {
	let weekday = await tp.system.suggester(["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"], [1, 2, 3, 4, 5, 6, 0])
	if (weekday != null) {
		let offset = weekday - tp.date.now("d")
		if (offset <= 0) { offset =  offset + 7 }
		resultDate = tp.date.now("YYYY-MM-DD", offset)
	}

}  else if (dateString == "> calendar") {
	let thisYear = Number(tp.date.now("YYYY"))
	let yearList = []
	for (let  i= 0; i < 10; i++) {
	    yearList.push(thisYear + i)
	}
let year = await tp.system.suggester(yearList, yearList)

if (year != null) {

	let month = await tp.system.suggester(["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"], [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12])

	if (month != null) {

		let daysInMonth = new Date(year, month, 0).getDate();
		let dateList = []
		for (let i = 1; i <= daysInMonth; i++) {
			dateList.push(i)
		}
		
		const getDateString = (year, month, date) => {
			const dateStringFix = (num) => {
			    if (num < 10) {
				      return "0" + num
				} else return "" + num
			}
		    return year + "-" + dateStringFix(month) + "-" + dateStringFix(date)
	    }
	    
		let dateListString = dateList.map(d => {
			let rawString =  getDateString(year, month, d)
			let weekdayString = tp.date.now("d", 0, rawString, "YYYY-MM-DD")
			let resultString = tp.date.now("DD MMMM — dd", 0, rawString, "YYYY-MM-DD")
			if (weekdayString == 0) {
				resultString = resultString + "   ________________________________________________________"
			}
			return resultString
		})
		let date = await tp.system.suggester(dateListString, dateList)
		
		if (date != null) {	
			resultDate = getDateString(year, month, date)
		}}} 

} else if (dateString == "(write date)") {
	dateString = await tp.system.prompt("Date format: " + myformat)
	resultDate = tp.date.now("YYYY-MM-DD", 0, dateString, myformat)
	if (resultDate == "Invalid date") {
		resultDate = ""
}}}

if (resultDate != "") {
resultDate = tp.date.now(resultFormat, 0, resultDate)
}

%><%* tR +=  resultDate%>
