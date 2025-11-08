[Obsidian](https://obsidian.md/) is my third most used application after [Keyboard Maestro](https://www.keyboardmaestro.com/) and [Alfred](https://www.alfredapp.com/). I’ve been using the [dataview](https://github.com/blacksmithgu/obsidian-dataview) plugin since I got started with Obsidian. It’s an incredible plugin that gives you the ability to treat your notes like database records. In this example I’ll show how I use dataview to make my projects queryable, and then how I use [Obsidian-Charts](https://github.com/phibr0/obsidian-charts) to make some bar charts of this data.

## Using Dataview Variables

Here’s an example of a _Project_ file in my Obsidian vault:

[![Example project file in Obsidian](https://agileadam.com/wp-content/uploads/2022/07/Xnip2022-07-28_09-33-20.png)](https://agileadam.com/wp-content/uploads/2022/07/Xnip2022-07-28_09-33-20.png)

|   |   |
|---|---|
||> [!NOTE] Project Details<br><br>> **Created:** 2022-07-27 Wednesday<br><br>><br><br>> [Type:: Project]<br><br>> [Tech:: [[Drupal]], [[PHP]]]<br><br>> [Teamwork:: ]<br><br>> [Backend Percentage:: 40]<br><br>> [Frontend Percentage:: 30]<br><br>> [Content Percentage:: 10]<br><br>> [PM Percentage:: 20]<br><br># Agileadam Foo<br><br>This is my project. It's just a demo for an agileadam.com blog post.|

Here’s the template file I use for my projects:

|   |   |
|---|---|
||> [!NOTE] Project Details<br><br>> **Created:** <% tp.date.now("YYYY-MM-DD dddd") %><br><br>><br><br>> [Parent:: [[MaineHealth Projects MOC]]]<br><br>> [Type:: Project]<br><br>> [Tech:: ]<br><br>> [Teamwork:: ]<br><br>> [Backend Percentage:: ]<br><br>> [Frontend Percentage:: ]<br><br>> [Content Percentage:: ]<br><br>> [PM Percentage:: ]<br><br># <% tp.file.title %>|

In my project pages I’m making use of Obsidian variables that are defined inline. You can also define them in the YAML frontmatter of any file.

## Querying with Dataview

Obsidian dataview lets me query the files in my vault _using those variables_. Here’s a straightforward example making use of the variables shown above:

|   |   |
|---|---|
||---<br><br>cssclass: academia, academia-rounded<br><br>---<br><br>```dataview<br><br>TABLE<br><br>backend-percentage as "% Backend",<br><br>frontend-percentage as "% Frontend",<br><br>content-percentage as "% Content",<br><br>pm-percentage as "% PM",<br><br>join(tech, ", ") as "Tech"<br><br>WHERE type = "Project"<br><br>AND !contains(file.path, "System")<br><br>SORT file.name<br><br>LIMIT 3<br><br>```|

The result of this is a table, shown here with a bit of [CSS](https://forum.obsidian.md/t/custom-css-for-tables-5-new-styles-ready-to-use-in-your-notes/17084/3) applied.

[![Table of query results](https://agileadam.com/wp-content/uploads/2022/07/Xnip2022-07-28_09-36-50.png)](https://agileadam.com/wp-content/uploads/2022/07/Xnip2022-07-28_09-36-50.png)Pretty cool, right? This table is updated in near real-time when any of the included data in my vault changes.

In order to use dataview results as the data for Obsidian Charts I used the [dataviewjs](https://blacksmithgu.github.io/obsidian-dataview/api/intro/) aspect of dataview.

After a lot of trial and error I finally understood how to query for the raw data from dataview and interpret/modify/use that data.

Here’s the final result:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19|```dataviewjs<br><br>const rawData = await dv.query('TABLE file.name, backend-percentage, frontend-percentage, content-percentage, pm-percentage WHERE type = "Project" AND !contains(file.path, "System") SORT file.name LIMIT 3');<br><br>const rows = rawData.value.values;<br><br>const chartData = {<br><br>    type: 'bar',<br><br>    data: {<br><br>        labels: rows.map(x => x[1]),<br><br>        datasets: [<br><br>            {label: 'Backend', data: rows.map(x => x[2]), backgroundColor: ['#2D3142']},<br><br>            {label: 'Frontend', data: rows.map(x => x[3]), backgroundColor: ['#EF3054']},<br><br>            {label: 'Content', data: rows.map(x => x[4]), backgroundColor: ['#43AA8B']},<br><br>            {label: 'PM', data: rows.map(x => x[5]), backgroundColor: ['#C7F2A7']},<br><br>        ],<br><br>    },<br><br>}<br><br>window.renderChart(chartData, this.container);<br><br>```|

The result looks like this:

[![Obsidian charts example](https://agileadam.com/wp-content/uploads/2022/07/Xnip2022-07-28_09-46-26.png)](https://agileadam.com/wp-content/uploads/2022/07/Xnip2022-07-28_09-46-26.png)

As I said, it took some trial and error to get here. I knew I had a working query (from the section above) so it was simply a matter of getting it into the chart. I read through the documentation for dataviewjs and determined I needed to use dv.query() . I used console.log(rawData)  to explore the results (after opening the developer tools in Obsidian via _View > Toggle Developer Tools_). After determining where the data was in the results I was able to work up some javascript to build out the data arrays that I thought Obsidian-Charts was expecting. I was awfully confused by the labels and data that Obsidian-Charts uses. The whole chart has labels and each dataset has a label. Eventually I figured it out — the top-level labels are the “columns” on the X axis; the other labels are repeated for each value in their associated data array; think of if this way… each dataset (e.g., Backend) is an array containing the value (e.g., 40) for each master label (e.g., Agileadam Foo). So in this example there are 3 elements in each of the four datasets — one for each of the Agileadam * projects.

The code above is the final refactoring of the following, which may be easier for you to digest if you’re not used to array maps:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29<br><br>30<br><br>31<br><br>32<br><br>33<br><br>34<br><br>35<br><br>36<br><br>37<br><br>38<br><br>39<br><br>40<br><br>41<br><br>42<br><br>43<br><br>44<br><br>45<br><br>46<br><br>47|```dataviewjs<br><br>const rawData = await dv.query('TABLE file.name, backend-percentage, frontend-percentage, content-percentage, pm-percentage WHERE type = "Project" AND !contains(file.path, "System") SORT file.name LIMIT 3');<br><br>const rows = rawData.value.values;<br><br>let backend = [];<br><br>let frontend = [];<br><br>let content = [];<br><br>let pm = [];<br><br>let labels = [];<br><br>for (let row of rows) {<br><br>labels.push(row[1]);<br><br>backend.push(row[2]);<br><br>frontend.push(row[3]);<br><br>content.push(row[4]);<br><br>pm.push(row[5]);<br><br>}<br><br>const chartData = {<br><br>type: 'bar',<br><br>data: {<br><br>labels: labels,<br><br>datasets: [{<br><br>label: 'Backend',<br><br>data: backend,<br><br>backgroundColor: ['#2D3142'],<br><br>},<br><br>{<br><br>label: 'Frontend',<br><br>data: frontend,<br><br>backgroundColor: ['#EF3054'],<br><br>},<br><br>{<br><br>label: 'Content',<br><br>data: content,<br><br>backgroundColor: ['#43AA8B'],<br><br>},<br><br>{<br><br>label: 'PM',<br><br>data: pm,<br><br>backgroundColor: ['#C7F2A7'],<br><br>}],<br><br>},<br><br>}<br><br>window.renderChart(chartData, this.container);<br><br>```|

Also, it may be helpful for you to see was “rows” looks like. Here’s an example:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21|/**<br><br>* EXAMPLE data for the "rows" variable:<br><br>*/<br><br>const rows = [<br><br>    [<br><br>        'SomeFileName.md', // Filename<br><br>        'Some File Title', // Title<br><br>        60,                // Backend percentage<br><br>        30,                // Frontend percentage<br><br>        4,                 // Content percentage<br><br>        6,                 // PM percentage<br><br>    ],<br><br>    [<br><br>        'AnotherFileName.md',<br><br>        'Another File Title Here',<br><br>        25,<br><br>        25,<br><br>        10,<br><br>        40,<br><br>    ],<br><br>];|