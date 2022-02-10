<script lang="ts">
	import { faCloudUploadAlt, faDownload } from '@fortawesome/free-solid-svg-icons';
	import { saveAs } from 'file-saver';
	import { FontAwesomeIcon } from 'fontawesome-svelte';
	import html2canvas from 'html2canvas';

	let main: HTMLElement;
	let dropArea: HTMLElement;
	let dragText: HTMLElement;
	let selectFile: HTMLElement;
	let image: HTMLElement;
	let file: File;
	let uselessDiv: HTMLElement;
	let files: FileList;
	let customBackground: string = '';
	let fontFamily = 'sans-serif';
	let fontSize: number = 14;
	let borderRadius: number = 0;
	let fontColor: string = '#000';
	let isDropVisible = true;
	let imgSrc = '';
	let text: string = "";
	let textArea: HTMLElement;
	let textAreaSvg: HTMLElement;
	let uselessVisible = false;
	const onBrowseFile = () => selectFile.click();
	const onFileChanged = () => {
		file = files[0];
		dropArea.classList.add('active');
		showFile();
	}
	const hideElements = (...elements: HTMLElement[]) => {
		elements.forEach((element: HTMLElement) => {
			element.style.visibility = 'hidden';
			element.style.height = '0px';
		})
	}

	const showElements = (...elements: HTMLElement[]) => {
		elements.forEach((element: HTMLElement) => {
			element.style.visibility = 'visible';
			element.style.height = 'unset';
		})
	}

	const downloadImage = () => {
		const { fontSize, fontFamily, color } = getComputedStyle(textArea)

		hideElements(uselessDiv, textArea);
		if (text != '') {
			const span = document.createElement('span');
			span.setAttribute('style', 'white-space: pre; text-align: center;');
			span.style.fontFamily = fontFamily;
			span.style.fontSize = fontSize;
			span.style.color = color;
			span.textContent = text;
			textAreaSvg.appendChild(span);
		}
		html2canvas(document.body).then((canvas: HTMLCanvasElement) => {
			showElements(uselessDiv, textArea);
			textAreaSvg.innerHTML = '';
			canvas.toBlob((blob: Blob) =>{
				var today = new Date();
				var date = today.getFullYear()+'-'+(today.getMonth()+1)+'-'+today.getDate();
				var time = today.getHours() + ":" + today.getMinutes() + ":" + today.getSeconds();
				var dateTime = date+' '+time;
				saveAs(blob, dateTime);
			})
		})
	}
	const dragOverDropArea = (event: Event) => {
		event.preventDefault();
		dropArea.classList.add("active");
		dragText.textContent = "Release to Upload File";
	}
	const dragLeaveDropArea = () => {
		dropArea.classList.remove("active");
  		dragText.textContent = "Drag & Drop to Upload File";
	}
	const dropInDropArea = (event: DragEvent) => {
		event.preventDefault();
		file = event.dataTransfer.files[0];
		showFile();
	}
	function showFile(){
		let fileType = file.type;
		let validExtensions = ["image/jpeg", "image/jpg", "image/png"]; 
		if(validExtensions.includes(fileType)){ 
			let fileReader = new FileReader();
			fileReader.onload = () => {
				imgSrc = fileReader.result.toString();
				isDropVisible = false;
				uselessVisible = true;
				main.classList.add('noMin');
			}
			fileReader.onerror = () => {
				alert("Could not read file contents!");
				dropArea.classList.remove("active");
				dragText.textContent = "Drag & Drop to Upload File";
			}
			fileReader.readAsDataURL(file);
		} else {
			alert("This is not an Image File!");
			dropArea.classList.remove("active");
			dragText.textContent = "Drag & Drop to Upload File";
		}
	}
	$: if (customBackground != '') {
		document.body.style.backgroundImage = customBackground;
	} else {
		document.body.style.backgroundImage = 'linear-gradient(43deg, #4158D0 0%, #C850C0 46%, #FFCC70 100%)';
	}

	$: if (textArea) textArea.style.fontSize = fontSize + "px";
	$: if (textArea) textArea.style.color = fontColor;
	$: if (textArea) textArea.style.fontFamily = fontFamily;
	$: if (fontFamily != "sans-serif") {
		const url = 'https://fonts.googleapis.com/css2?family=' + fontFamily.replace(' ', '+') + '&display=swap';
		const link = document.createElement('link');
		link.href = url;
		link.rel = 'stylesheet';
		document.head.appendChild(link);
	}
	$: if (image) image.style.borderRadius = borderRadius + "px";
</script>

{#if uselessVisible}
	<div class="useless" bind:this={uselessDiv}>
		<h2 class="glass-head">Glassmorphism ScreenShot Generator</h2>
		<div class="flex">
			<label for="custom-background">Custom Background Image</label>
			<input type="text" placeholder="Enter custom css for backround-image here:" bind:value={customBackground} name="custom-background">
		</div>
		<div class="flex">
			<label for="font-size">Font Size</label>
			<input type="number" name="font-size" min="1" bind:value={fontSize}>
		</div>
		<div class="flex">
			<label for="font-color">Text Color</label>
			<input type="color" name="font-color" bind:value={fontColor}>
		</div>
		<div class="flex">
			<label for="font-family">Font Family (From Google Fonts)</label>
			<input type="text" name="font-family" bind:value={fontFamily}>
		</div>
		<div class="flex">
			<label for="image-border-radius">Image Border Radius</label>
			<input type="number" name="image-border-radius" min="0" bind:value={borderRadius}>
		</div>
		<button class="download-img" on:click={downloadImage}><FontAwesomeIcon icon={faDownload}></FontAwesomeIcon></button>
	</div>
{/if}

<main bind:this={main}>
	{#if isDropVisible}
		<h2 class="glass-head">Glassmorphism ScreenShot Generator</h2>
		<div class="drag-area" bind:this={dropArea} on:dragover={dragOverDropArea} on:dragleave={dragLeaveDropArea} on:drop={dropInDropArea}>
			<div class="icon"><FontAwesomeIcon icon={faCloudUploadAlt}></FontAwesomeIcon></div>
			<header bind:this={dragText}>Drag & Drop to Upload File</header>
			<span>OR</span>
			<button on:click={onBrowseFile}>Browse File</button>
			<input type="file" hidden bind:this={selectFile} bind:files={files} on:change={onFileChanged}>
		</div>

	{:else}
		<div class="edit-area">
			<textarea bind:value={text} bind:this={textArea} placeholder="Your Text Here"/>
			<div bind:this={textAreaSvg}></div>
			<img src={imgSrc} alt="" bind:this={image}>
		</div>
	{/if}
</main>

<style>

	input[type="color"] {
		height: calc(4vw + 0.8rem);
		margin-left: calc(1vw + 1rem);
	}

	.flex {
		display: flex;
		align-items: center;
		justify-content: center;
	}
	label::after {
		content: ':';
	}
	.useless input[type="text"], .useless input[type="number"], label {
		font-size: calc(1vw + 0.8rem);
	}

	.useless *:last-child{
		margin-bottom: calc(2vw + 1rem);
	}

	.useless input[type="text"]::placeholder {
		color: #333;
	}
	.edit-area {
		display: flex;
		text-align: center;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	.useless {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	.glass-head {
		font-size: calc(3vw + 0.8rem);
		color: #fff;
		text-align: center;
		margin-bottom: calc(0.5vw + 0.5rem);
	}

	textarea,input[type="text"], input[type="number"] {
		z-index: 999;
		overflow: hidden;
		background: transparent;
		border: 0;
		outline: 0;
		width: 100%;
		padding: 0;
		margin-top: calc(20px + 2vw);
		text-align: center;
		font-size: calc(3vw + 0.5rem);
		border-bottom: 2px solid #808080;
	}

	textarea:focus, input[type="text"]:focus, input[type="number"]:focus, textarea:active, input[type="text"]:active, input[type="number"]:active{
		border-bottom: 2px solid #000000;
	}

	main {
		min-width: 90%;
		display: flex;
		padding: calc(2vw + 0.5rem);
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background: rgba( 255, 255, 255, 0.25 );
		box-shadow: 0 8px 32px 0 rgba( 31, 38, 135, 0.37 );
		backdrop-filter: blur( 16px ) saturate(60%);
		-webkit-backdrop-filter: blur( 16px ) saturate(60%);
		border-radius: calc(2vw + 0.5rem + 20px);
		border: 1px solid rgba( 255, 255, 255, 0.18 );
	}

	.edit-area img {
		max-width: 100%;
	}

	.download-img {
		margin: 0;
		box-shadow: none;
		border: 0;
		outline: 0;
		cursor: pointer;
		background-color: transparent;
		font-size: calc(5vw + 20px);
	}
	.download-img:hover, .download-img:active, .download-img:focus {
		box-shadow: none;
		border: 0;
		outline: 0;
		
	}
</style>