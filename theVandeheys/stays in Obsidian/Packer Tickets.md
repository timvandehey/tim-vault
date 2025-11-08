- [x] Nancy Willis 2 or 4 for Sept 11 Commanders ✅ 2025-07-01
- [x] Bob Willis 2 for Nov 2 Carolina ✅ 2025-07-13

```dataview
table without ID
   link(file.link, "📝") as "📝"
  ,person as "Name"
  ,("$" + amount) as "Paid"
  ,date as "Date"
from #packerTickets 
sort date
```







