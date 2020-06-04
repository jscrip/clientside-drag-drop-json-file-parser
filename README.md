# Drag & Drop JSON File Parser for the Browser
 Parse JSON Files in the Browser 100% Client Side. No frills JSON file parsing with only 20-30 lines of code  

## Testing & Deployment
 ### Quick Test
    Copy and paste the code (below) directly into the console.
 ### Deploy
    Wrap it in HTML script tags

## Usage:
    No special permissions or dependencies required.
    Once loaded, simply drag & drop local JSON files anywhere on the active page. 
    On success, the JSON data should be logged to the developer console. Feel free to customize it there.

## Warning
### not thoroughly tested - use at your own risk.
I use modern Firefox. I'm not concerned about cross-browser compatibility since it suits my needs, but let me know if you run into issues. I might be able to help.

```javascript
(() => {
const fileToString = (inputFile) => {
	const fr = new FileReader();
	return new Promise((resolve, reject) => {
		fr.onerror = () => { fr.abort(); reject(new DOMException("Problem parsing input file."));} //handle file read error
		fr.onload = () => resolve(fr.result); //resolve promise once the file is loaded
		fr.readAsText(inputFile); //read the file as plain text. On success, the promise resolves, passign the result back to the dropHandler function
})}
const dropHandler = async (e) => {
	e.preventDefault(); //disable default drop behavior
  for await (let file of e.dataTransfer.files) { //loop through dropped files
	try {
		let name = file.name.replace(".csv","");
		const str = await fileToString(file); //convert dropped file to string
		const data = await JSON.parse(str); //convert file string to json
		console.log({name,data});  /*do stuff with the JSON data*/
	} catch (err) {console.log({err})}
   }}
window.ondragover = (e) => e.preventDefault(); //Prevent default browser behavior
window.ondrop = (e) => dropHandler(e); //Handle dropped files
})();
```
