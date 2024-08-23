
```dataviewjs
//const table = dv.markdownTable(["File", "Genre", "Time Read", "Rating"], //dv.pages('"DnD/DMing"') .sort(b => b.rating) .map(b => [b.file.link, b.genre, b["time-read"], b.metadata])) 


/**
 * Returns the folder name, NaN if folder doesn't exist
 */
function getFolderName(currPage) {
	const folder = currPage.file.folder
	
	if (folder != "") {	
		const folderSplit = folder.split("/")
		const lastFolderI = folderSplit.length - 1
		//console.log("folderName " + folderSplit[lastFolderI])
		return folderSplit[lastFolderI]
	}
	return undefined
}

/**
 * Checks if the current file is a MoC
 */
function isMOC(currPage) {
	const folderName = getFolderName(currPage)
	
	if (folderName == undefined) {
		return false
	}

	//console.log(currPage.file.name)
	//console.log(folderName)
	if (currPage.file.name == folderName) {
		return true
	} else {
		return false
	}
}

/**
 * Adds a link to the MOC
 * If it's also a MOC it links to previous MOC
 **/
function addMOCLink(currPage) {

	const folder = currPage.file.folder
	const folderSplit = folder.split("/")

	if (isMOC(currPage)) {

		if (folderSplit.length <= 1) {
			return
		}
		//console.log(folderSplit.slice(0, -1))
		const parentFolder = folderSplit.slice(0, -1).join("/")
		const parentFolderMoC = parentFolder + "/" + folderSplit[folderSplit.length - 2]
		//console.log(folderIndexFilePath)
		dv.span("## Parent MoC: " + dv.fileLink(parentFolderMoC) + "\n")
	} else {

		const folderName = getFolderName(currPage)

		if (folderName == undefined) {
			return
		}
		const MoCFile = folder + "/" + folderName
		dv.span("## Parent MoC: " + dv.fileLink(MoCFile) + "\n")
	}
}


function getHeaderLinks(filepath, fileCache, text) {
	const headers = fileCache.headings
	if (headers == undefined) {
		return text
	}
	
	let init = 100
	const minHeaderLevel = headers.reduce(
		(minVal, currHeader) => Math.min(minVal,currHeader.level),
		init
	)
	//console.log(minHeaderLevel)

	let prevHeaderLevel = -1
	for (let header of headers) {
		
		if (header.heading == "Table of Content") {
			continue
		}
		const currentHeaderLevel = header.level - minHeaderLevel 

		if (currentHeaderLevel > prevHeaderLevel + 1) {

			for (let i = prevHeaderLevel + 1; i < currentHeaderLevel ; i++) {
				let tabs = "\t".repeat(i);
				text += tabs + "* " + "\n"
			}
		}
		let tabs = "\t".repeat(currentHeaderLevel);
		const headerFix = header.heading.replace("'", "\'")
		//console.log(headerFix)
		text += tabs + "* " + dv.sectionLink(filepath, headerFix, false, headerFix) + "\n"
		//console.log(text)
	}
	return text
}

function createMOC(currPage, cache) {
	let text = "## Map Of Content: \n"
	//console.log(currPage.file.folder)
	let pages = dv.pages('"' + currPage.file.folder + '"')
	dv.span(text)
	text = ""
	
	for (let page of pages) {
		
		if (page.file.path == currPage.file.path) {
			continue
		}

		if (page.file.folder != currPage.file.folder && isMOC(page) == false) {
			continue
		}
		//console.log(page.file)
		const filepath = page.file.path
		//links.push(dv.fileLink(filepath))

		if (isMOC(page)) {
			text += "### " + dv.fileLink(filepath) + " (MoC)\n"
		} else {
			text += "### " + dv.fileLink(filepath) + "\n"
		}
		
		//console.log(text)
		
		//console.log(filepath)
		const fileCache = cache.getCache(filepath)
		//console.log("See below for file cache")
		//console.log(fileCache.headings)
		text = getHeaderLinks(filepath, fileCache, text)
		dv.span(text)
		text = ""
	}
}

function createTOC(filepath, cache) {
	let text = "## Table of Content:\n"
	//console.log(filepath)
	const fileCache = cache.getCache(filepath)
	text = getHeaderLinks(filepath, fileCache, text)
	dv.span(text)
}

const cache = this.app.metadataCache;

let currentPage = dv.current()

addMOCLink(currentPage)

if (isMOC(currentPage)) {
	createMOC(currentPage, cache)
} else {
	createTOC(currentPage.file.path, cache)
}


```
---