[[home]]

```dataviewjs
let log = console.log;

let entries = dv.pages().where(p => p.file.path.includes("healthLogEntries") && p.weight > 0)
    .sort(p => p.datetime, 'asc')
let first = entries.findIndex( e => e.mounjaro_start) 
entries = entries.slice(first-10) 

let dates = entries.map(e => new Date(e.datetime).toLocaleDateString('en-CA')).array(); // Using 'en-CA' for YYYY-MM-DD format
let weights = entries.map(e => e.weight).array();

let minWeight = weights.length > 0 ? Math.min(...weights) - 15 : 190;
let maxWeight = weights.length > 0 ? Math.max(...weights) + 10 : 265;

const chartData = {
    type: 'line',
    data: {
        labels: dates,
        datasets: [{
            label: 'Weight',
            data: weights,
            borderColor: 'rgb(75, 192, 192)',
            backgroundColor: 'rgba(75, 192, 192, 0.2)',
            fill: true,
            tension: 0.5,
            indexAxis: 'x'
        }]
    },
    options: {
        scales: {
	        x: {id: 'x', type: "category"},
            y: { id: 'y',
                beginAtZero: false,
                suggestedMin: minWeight,
                suggestedMax: maxWeight
            }
        },
        plugins: {
                annotation: {
                    annotations: [
                    {
                        type: 'line',
                        mode: 'horizontal',
                        scaleID: 'y',
                        value: '215',
                        borderColor: 'blue',
                        borderWidth: 1,
                        label: {
	                        display: true,
                            enabled: true,
                            content: ['Goal','215 lbs'],
                            position: '', // start, end
                            borderColor: "red",
                            color: "black",
                            backgroundColor: 'white',
                            xAdjust: 0,
                            yAdjust: 0
                        }
                    },
                    {
                        type: 'line',
                        mode: 'vertical',
                        scaleID: 'x',
                        value: '2025-02-01',
                        borderColor: 'red',
                        color: 'white',
                        borderWidth: 2,
                        label: {
	                        display: true,
                            enabled: true,
                            content: ['1st','Mounjaro','Injection','245 lbs'],
                            position: 'end', // start
                            borderColor: "red",
                            color: "black",
                            backgroundColor: 'white',
                            xAdjust: 0,
                            yAdjust: 0
                        }
                    }
                    ]
                }
            }
    }
}

//log(chartData); // Check browser console (Ctrl+Shift+I) to see the final object

window.renderChart(chartData, this.container);
```

![[healthLog.base]]

## Full Table

```dataview
TABLE datetime,weight,bs,bp,pulse,note
FROM "_apps/healthLog/healthLogEntries"
SORT datetime DESC
```



