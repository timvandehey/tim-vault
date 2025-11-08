<%*

/*
const dv = app.plugins.plugins.dataview.api;
const data = await dv.pages('"test note"');
tR += data.test
*/

let choice = await tp.system.suggester(['default','Option 1', 'Option 2', 'Custom'], [{a:7,b:9},'Option 1', 'Option 2', 'Custom'],false);
if (choice == 'Custom') {
    choice = await tp.system.prompt('Please enter your custom option:');
}
	tR += `---\njunk: ${JSON.stringify(choice)}\n---`
-%>
