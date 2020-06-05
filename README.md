# Drag & Drop JSON File Parser for the Browser
 Parse JSON Files in the Browser (100% Client Side). Lightweight Vanilla JavaScript (~30 lines of code).  

## How To Demo 
Copy and paste the code (below) and paste it directly into the browser console.
If the code was injected successfully, drag & drop one or more JSON files anywhere.
Results are logged to the console

## Deployment Options
 ### Add to an HTML Page:
	a) Wrap it in HTML <script> tags
	b) Save it as a .js file and load it externally 
	c) Adapt it to work as a userscript (for browser extension like tampermonkey & greasemonkey)
    
## Usage:
	No special permissions or dependencies required.
	Once loaded, simply drag & drop one or more JSON files anywhere on the page. 
	On success, parsed JSON data should be logged to the developer console. Feel free to customize it there.

## Warnings, Disclainers, and Gotchas
### Not Battle Ready - lightly tested, use at your own risk.
Tested on a modern version of Firefox on Windows. At the moment, I'm not concerned about cross-browser compatibility since it suits my needs, but let me know if you run into issues. I might be able to help.
### To avoid crashing the browser, file lists are parsed sequentially.
Tip: Need to parse the whole list before moving on? Adapting this script to use promise.all() should be fairly straight forward.
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
		let name = file.name.replace(".json","");
		const str = await fileToString(file); //convert dropped file to string
		const data = await JSON.parse(str); //convert file string to json
		console.log({name,data});  /*do stuff with the JSON data*/
	} catch (err) {console.log({err})}
   }}
window.ondragover = (e) => e.preventDefault(); //Prevent default browser behavior
window.ondrop = (e) => dropHandler(e); //Handle dropped files
})();
```
## Same code + timer on the drop handler
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
		const t0 = performance.now();	
		let name = file.name.replace(".json","");
		const str = await fileToString(file); //convert dropped file to string
		const data = await JSON.parse(str); //convert file string to json
		const t1 = performance.now();
		const parseTime = `${t1 - t0} milliseconds.`;
		console.log({name,data,parseTime});  /*do stuff with the JSON data*/	
	} catch (err) {console.log({err})}
 }}
window.ondragover = (e) => e.preventDefault(); //Prevent default browser behavior
window.ondrop = (e) => dropHandler(e); //Handle dropped files
})();
```
## This demo integrates d3.js to parse CSV files. 
	Supported File Types: json,csv,txt
	Note: txt files must be delimited by one of the following (ordered by priority):
		1) lines:  "\n"
		2) tabs:   "\t"		
		3) commas: ","

```javascript
(() => {
	
	var options = {
		scripts: ["https://d3js.org/d3.v5.min.js"]
	}

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
		const str = await fileToString(file); //convert dropped file to string
		let format = file.name.slice(file.name.lastIndexOf(".")+1); //get assumed format from file name
		var name = file.name.replace("."+format,"");
		if(format == "json"){
			var data = await JSON.parse(str); //convert file string to json
		}else if(format == "txt"){
			var hasLines = str.indexOf("\n") > -1;
			var hasComma = str.indexOf(",") > -1;
			var hasTab = str.indexOf("\t") > -1;
			if(hasLines){
				var data = await str.split("\n"); //split line-delimited list
			} else if(hasTab){
					var data = await str.split("\t"); //split tab-delimited list
			}	else if(hasComma){
					var data = await str.split(","); //split comma-delimited list
			} 
		}else if(format == "csv"){
			var data = await d3.csvParse(str) //convert csv string to json
		}

		console.log({name,format,data});  /*do stuff with the JSON data*/	
	} catch (err) {console.log({err})}
 }}
 
window.ondragover = (e) => e.preventDefault(); //Prevent default browser behavior
window.ondrop = (e) => dropHandler(e); //Handle dropped files

const loadScripts = (scripts) => {
	console.log(`Loading External Scripts`);
	var scriptCountdown = scripts.length;
	var loadScript = (url) => {
		var scriptsLoaded = () => {
			console.log({scriptsLoaded:scriptCountdown});
			scriptCountdown == 0 ? console.log("External scripts loaded. Ready to parse!") : null;
			return true;
		}
		console.log("loadScript",{url})
		var imported = document.createElement('script');
				imported.src = url;
				imported.addEventListener("load", () => {
					scriptCountdown--;
					scriptsLoaded();
				});
				document.head.appendChild(imported);
	}
	scripts.forEach(loadScript)
}
loadScripts(options.scripts);
})();
```
