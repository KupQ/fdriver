<template>
  <div class="main" @dragenter.prevent @dragover.prevent @drop.prevent="onDrop">
    <progress
      v-if="uploadProgress !== null"
      :value="uploadProgress"
      max="100"
    ></progress>
    <UploadPopup
      v-model="showUploadPopup"
      @upload="onUploadClicked"
      @createFolder="createFolder"
    ></UploadPopup>
    <button class="upload-button circle" @click="showUploadPopup = true">
      <img
        style="filter: invert(100%)"
        src="https://cdnjs.cloudflare.com/ajax/libs/material-design-icons/4.0.0/png/file/upload_file/materialicons/36dp/2x/baseline_upload_file_black_36dp.png"
        alt="Upload"
        width="36"
        height="36"
        @contextmenu.prevent
      />
    </button>
     
     
   <Dialog v-model="showSuccessDialog"  max-width="78">
     <div class="modal"  id="upload"  @click.stop>
    <div class="modal__header">
      <button class="modal__close-button" type="button" @click="showSuccessDialog=false">
        <svg class="modal__close-icon" viewBox="0 0 16 16" width="16px" height="16px" aria-hidden="true">
          <g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
            <polyline points="1,1 15,15" />
            <polyline points="15,1 1,15" />
          </g>
        </svg>
        <span class="modal__sr">Close</span>
      </button>
    </div>
    <div class="modal__body">
      <div class="modal__col">       
        <!-- check -->
        <svg class="modal__icon modal__icon--green" viewBox="0 0 24 24" width="24px" height="24px" aria-hidden="true">
          <g fill="none" stroke="hsl(138,90%,50%)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle class="modal__icon-sdo69" cx="12" cy="12" r="11" stroke-dasharray="69.12 69.12" />
            <polyline class="modal__icon-sdo14" points="7 12.5 10 15.5 17 8.5" stroke-dasharray="14.2 14.2" />
          </g>
        </svg>
      </div>
      <div class="modal__col">
        <div class="modal__content">
          <h2 class="modal__title">Upload Successful!</h2>
          <p class="modal__message">Your file has been uploaded. You can copy the link to your clipboard.</p>
          <div class="modal__actions modal__actions--center"  @click.stop>
             <button class="modal__button" type="button" @click="copyLink(`/raw/${focusedItem}`)">
                  <template v-if="linkCopied">
                    <svg class="modal__icon modal__icon--green" viewBox="0 0 24 24" width="16px" height="16px" aria-hidden="true">
                      <g fill="none" stroke="hsl(138, 90%, 50%)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <circle cx="12" cy="12" r="11" />
                        <polyline points="7 12.5 10 15.5 17 8.5" />
                      </g>
                    </svg>
                    Copied
                  </template>
                  <template v-else>
                    Copy Link
                  </template>
                </button>
            <button class="modal__button" type="button" @click="showSuccessDialog=false">Done</button>
          </div>
        </div>
      </div>
    </div>
   </div>
  </Dialog>

<Dialog v-model="showErrorDialog"  max-width="78">
  <div class="modal"  id="upload"  data-state="0" data-ready="false" @click.stop>
    <div class="modal__header">
      <button class="modal__close-button" type="button" @click="showErrorDialog=false">
        <svg class="modal__close-icon" viewBox="0 0 16 16" width="16px" height="16px" aria-hidden="true">
          <g fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
            <polyline points="1,1 15,15" />
            <polyline points="15,1 1,15" />
          </g>
        </svg>
        <span class="modal__sr">Close</span>
      </button>
    </div>
    <div class="modal__body">
      <div class="modal__col">
           
       <svg class="modal__icon modal__icon--red" viewBox="0 0 24 24" width="24px" height="24px" aria-hidden="true" display="none">
				<g fill="none" stroke="hsl(3,90%,50%)" stroke-width="2" stroke-linecap="round">
					<circle class="modal__icon-sdo69" cx="12" cy="12" r="11" stroke-dasharray="69.12 69.12" />
					<line class="modal__icon-sdo14" x1="7" y1="7" x2="17" y2="17" stroke-dasharray="14.2 14.2" />
					<line class="modal__icon-sdo14" x1="17" y1="7" x2="7" y2="17" stroke-dasharray="14.2 14.2" />
				</g>
			</svg>
      </div>
      <div class="modal__content">
				<h2 class="modal__title">Oops!</h2>
				<p class="modal__message">Your file could not be uploaded due to an error. Try uploading it again?</p>
				<div class="modal__actions modal__actions--center">
        <button class="modal__button" type="button" data-action="upload" @click="retryUpload">Retry</button>
 					<button class="modal__button" type="button" data-action="cancel"  @click="showErrorDialog=false">Cancel</button>
				</div>
			</div>
    </div>
   </div>
  </Dialog>




    <div class="app-bar">
      <input type="search" v-model="search" aria-label="Search" />
      <div class="menu-button">
        <button class="circle" @click="showMenu = true">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 448 512"
            width="24"
            height="24"
            title="Menu"
            style="display: block; margin: 4px"
          >
            <!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. -->
            <path
              d="M120 256c0 30.9-25.1 56-56 56s-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56zm160 0c0 30.9-25.1 56-56 56s-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56zm104 56c-30.9 0-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56s-25.1 56-56 56z"
            />
          </svg>
        </button>
        <Menu
          v-model="showMenu"
          :items="[{ text: 'Name' }, { text: 'Size' }, { text: 'Paste' }]"
          @click="onMenuClick"
        />
      </div>
    </div>

    <div v-if="loading" class="loader-container">
      <div class="loader"></div>
    </div>
    <div v-else-if="!filteredFiles.length && !filteredFolders.length" class="no-files">
      <span>No documents are available</span>
    </div>
   
   
</template>

<script>
import {
  generateThumbnail,
  blobDigest,
  multipartUpload,
  SIZE_LIMIT,
} from "/assets/main.mjs";
import Dialog from "./Dialog.vue";
import Menu from "./Menu.vue";
import MimeIcon from "./MimeIcon.vue";
import UploadPopup from "./UploadPopup.vue";

export default {
  data: () => ({
    clipboard: null,
    cwd: new URL(window.location).searchParams.get("p") || "",
    files: [],
    folders: [],
    focusedItem: null,
    loading: false,
    linkCopied: false,
    order: null,
    search: "",
    showContextMenu: false,
    showMenu: false,
    showUploadPopup: false,
    showSuccessDialog:false,
    showErrorDialog:false,
    uploadProgress: null,
    failedUploads: [],
    uploadQueue: [],
  }),

  computed: {
    filteredFiles() {
      let files = this.files;
      if (this.search) {
        files = files.filter((file) =>
          file.key.split("/").pop().includes(this.search)
        );
      }
      return files;
    },

    filteredFolders() {
      let folders = this.folders;
      if (this.search) {
        folders = folders.filter((folder) => folder.includes(this.search));
      }
      return folders;
    },
  },

  methods: {
    copyLink(link) {
    
     const url = new URL(link, window.location.origin);
      navigator.clipboard.writeText(url.toString()).then(() => {
        this.linkCopied = true;
        this.showSuccessDialog = false;
        setTimeout(() => {
          this.linkCopied = false;
        }, 3000); 
      });
      
    },
     retryUpload() {
     
      this.showErrorDialog = false;
      this.showUploadPopup=true;
    },
     
   

    async createFolder() {
      try {
        const folderName = window.prompt("Folder name");
        if (!folderName) return;
        this.showUploadPopup = false;
        const uploadUrl = `/api/write/items/${this.cwd}${folderName}/_$folder$`;
        await axios.put(uploadUrl, "");
        this.fetchFiles();
      } catch (error) {
        fetch("/api/write/")
          .then((value) => {
            if (value.redirected) window.location.href = value.url;
          })
          .catch(() => {});
       
      }
    },

    fetchFiles() {
      this.files = [];
      this.folders = [];
      this.loading = true;
      fetch(`/api/children/${this.cwd}`)
        .then((res) => res.json())
        .then((files) => {
       
          this.files = files.value;
          if (this.order) {
            this.files.sort((a, b) => {
              if (this.order === "size") {
                return b.size - a.size;
              }
            });
          }
          this.folders = files.folders;
          this.loading = false;
        });
    },

    formatSize(size) {
      const units = ["B", "KB", "MB", "GB", "TB"];
      let i = 0;
      while (size >= 1024) {
        size /= 1024;
        i++;
      }
      return `${size.toFixed(1)} ${units[i]}`;
    },

    onDrop(ev) {
      let files;
      if (ev.dataTransfer.items) {
        files = [...ev.dataTransfer.items]
          .filter((item) => item.kind === "file")
          .map((item) => item.getAsFile());
      } else files = ev.dataTransfer.files;
      this.uploadFiles(files);
    },

    onMenuClick(text) {
      switch (text) {
        case "Name":
          this.order = null;
          break;
        case "Size":
          this.order = "size";
          break;
        case "Paste":
          return this.pasteFile();
      }
      let compareFn = undefined;
      if (this.order === "size") compareFn = (a, b) => b.size - a.size;
      this.files.sort(compareFn);
    },

    onUploadClicked(fileElement) {
      if (!fileElement.value) return;
      let filePath=fileElement.value.split('\\').pop().split('/').pop();
      this.focusedItem=filePath;
      
      this.uploadFiles(fileElement.files);
    
      this.showUploadPopup = false;
      fileElement.value = null;
    },

   
   
    async removeFile(key) {
      if (!window.confirm(`Delete ${key} permanently?`)) return;
      await axios.delete(`/api/write/items/${key}`);
      this.fetchFiles();
    },

    
    async processUploadQueue() {
 
  if (!this.uploadQueue.length) {
    this.fetchFiles();
    this.uploadProgress = null;
    return;
  }

  const { basedir, file } = this.uploadQueue.pop(0);
  let thumbnailDigest = null;

  try {
    if (file.type.startsWith("image/") || file.type === "video/mp4") {
      const thumbnailBlob = await generateThumbnail(file);
      const digestHex = await blobDigest(thumbnailBlob);

      const thumbnailUploadUrl = `/api/write/items/_$flaredrive$/thumbnails/${digestHex}.png`;
      await axios.put(thumbnailUploadUrl, thumbnailBlob);
      thumbnailDigest = digestHex;
    }

    const uploadUrl = `/api/write/items/${basedir}${file.name}`;
    const headers = {};
    const onUploadProgress = (progressEvent) => {
      var percentCompleted = (progressEvent.loaded * 100) / progressEvent.total;
      this.uploadProgress = percentCompleted;
    };

    if (thumbnailDigest) headers["fd-thumbnail"] = thumbnailDigest;
    if (file.size >= SIZE_LIMIT) {
      await multipartUpload(`${basedir}${file.name}`, file, {
        headers,
        onUploadProgress,
      });
    } else {
      await axios.put(uploadUrl, file, { headers, onUploadProgress });
    }

    
    this.showSuccessDialog = true;
  } catch (error) {
   this.failedUploads.push({ basedir, file });
    console.error(`Upload ${file.name} failed`, error);
    this.showErrorDialog = true;

    fetch("/api/write/")
      .then((value) => {
        if (value.redirected) window.location.href = value.url;
      })
      .catch(() => {});
  } finally {
    setTimeout(() => this.processUploadQueue(), 0);
  }
},
  


    uploadFiles(files) {
  
     if (this.cwd && !this.cwd.endsWith("/")) this.cwd += "/";

      const uploadTasks = Array.from(files).map((file) => ({
        basedir: this.cwd,
        file,
      }));
      this.uploadQueue.push(...uploadTasks);
      setTimeout(() => this.processUploadQueue()); 
    },
  },

  watch: {
    cwd: {
      handler() {
        this.fetchFiles();
        const url = new URL(window.location);
        if ((url.searchParams.get("p") || "") !== this.cwd) {
          this.cwd
            ? url.searchParams.set("p", this.cwd)
            : url.searchParams.delete("p");
          window.history.pushState(null, "", url.toString());
        }
        document.title = `${
          this.cwd.replace(/.*\/(?!$)|\//g, "") || "/"
        } - FlareDrive`;
      },
      immediate: true,
    },
  },

  created() {
    window.addEventListener("popstate", (ev) => {
      const searchParams = new URL(window.location).searchParams;
      if (searchParams.get("p") !== this.cwd)
        this.cwd = searchParams.get("p") || "";
    });
  },

  components: {
    Dialog,
    Menu,
    MimeIcon,
    UploadPopup,
  },
};
</script>

<style>
@import url('https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css');

.main {
  height: 100%;
}

.app-bar {
  position: sticky;
  top: 0;
  padding: 8px;
  background-color: white;
  display: flex;
}

.menu-button {
  display: flex;
  position: relative;
  margin-left: 4px;
}

.menu-button > button {
  transition: background-color 0.2s ease;
}

.menu-button > button:hover {
  background-color: whitesmoke;
}

.menu {
  position: absolute;
  top: 100%;
  right: 0;
}
.modal {
	
	border-radius: 1em;
	box-shadow: 0 0.75em 1em hsla(var(--hue),10%,5%,0.3);
	color: hsl(var(--hue),10%,5%);
  width: calc(100% - 3em);
	max-width: 34.5em;
  overflow: hidden;
	position: relative;
  transition:
		background-color var(--trans-dur),
		color var(--trans-dur);
}
#upload {
  background: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 600px; 
}
.modal:before {
	background-color: hsl(223,90%,60%);
	border-radius: 50%;
	content: "";
	filter: blur(60px);
	opacity: 0.15;
	position: absolute;
	top: -8em;
	right: -15em;
	width: 25em;
	height: 25em;
	z-index: 0;
	transition: background-color var(--trans-dur);
}
.modal__actions {
	animation-delay: 0.2s;
	display: flex;
	align-items: center;
	flex-wrap: wrap;
}
.modal__body,
.modal__header {
	position: relative;
	z-index: 1;
}
.modal__body {
	display: flex;
	flex-direction: column;
	padding: 0 2em 1.875em 1.875em;
}
.modal__button,
.modal__close-button {
	color: currentColor;
	cursor: pointer;
	-webkit-tap-highlight-color: transparent;
}
.modal__button {
	background-color: #808080;
	border-radius: 0.25rem;
	font-size: 0.75em;
	padding: 0.5rem 2rem;
	transition:
		background-color var(--trans-dur),
		border-color var(--trans-dur),
		opacity var(--trans-dur);
	width: 100%;
}
.modal__button + .modal__button {
	margin-top: 0.75em;
}
.modal__button:disabled {
	opacity: 0.5;
}
.modal__button:focus,
.modal__close-button:focus {
	outline: transparent;
}
.modal__button:hover,
.modal__button:focus-visible {
	background-color: hsla(var(--hue),10%,60%,0.2);
}
.modal__button--upload {
	background-color: transparent;
	border: 0.125rem dashed hsla(var(--hue),10%,50%,0.4);
	flex: 1;
	padding: 0.375rem 2rem;
}
.modal__col + .modal__col {
	flex: 1;
	margin-top: 1.875em;
}
.modal__close-button,
.modal__message,
.modal__progress-value {
	color: hsl(var(--hue),10%,30%);
	transition: color var(--trans-dur);
}
.modal__close-button {
	background-color: transparent;
	display: flex;
	width: 1.5em;
	height: 1.5em;
	transition: color var(--trans-dur);
}
.modal__close-button:hover,
.modal__close-button:focus-visible {
	color: hsl(var(--hue),10%,40%);
}
.modal__close-icon {
	display: block;
	margin: auto;
	pointer-events: none;
	width: 50%;
	height: auto;
}
.modal__content > * {
	animation-name: fadeSlideIn;
	animation-duration: 0.5s;
	animation-timing-function: ease-in-out;
	animation-fill-mode: forwards;
	opacity: 0;
}
.modal__file {
	flex: 1;
	font-size: 0.75em;
	font-weight: 700;
	margin-right: 0.25rem;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
.modal__file ~ .modal__button {
	margin-top: 1.5em;
}
.modal__file-icon {
	color: hsl(var(--hue),10%,50%);
	display: block;
	margin-right: 0.75em;
	width: 1.5em;
	height: 1.5em;
	transition: color var(--trans-dur);
}
.modal__header {
	display: flex;
	justify-content: flex-end;
	align-items: center;
	height: 2.5em;
	padding: 0.5em;
}
.modal__icon {
	display: block;
	margin: auto;
	width: 2.25em;
	height: 2.25em;
}
.modal__icon--blue g {
	stroke: hsl(223,90%,50%);
}
.modal__icon--red g {
	stroke: hsl(3,90%,50%);
}
.modal__icon--green g {
	stroke: hsl(138,90%,40%);
}
.modal__icon circle,
.modal__icon line,
.modal__icon polyline {
	animation: sdo 0.25s ease-in-out forwards;
	transition: stroke var(--trans-dur);
}
.modal__icon :nth-child(2) {
	animation-delay: 0.25s;
}
.modal__icon :nth-child(3) {
	animation-delay: 0.5s;
}
.modal__icon-sdo10 {
	stroke-dashoffset: 10;
}
.modal__icon-sdo14 {
	stroke-dashoffset: 14.2;
}
.modal__icon-sdo69 {
	stroke-dashoffset: 69.12;
	transform: rotate(-90deg);
	transform-origin: 12px 12px;
}
.modal__message {
	animation-delay: 0.1s;
	font-size: 1em;
	margin-bottom: 1.5em;
	min-height: 3em;
}
.modal__progress {
	flex: 1;
}
.modal__progress + .modal__button {
	margin-top: 1.75em;
}
.modal__progress-bar {
	background-image: linear-gradient(90deg,hsl(var(--hue),90%,50%),hsl(var(--hue),90%,70%));
	border-radius: 0.2em;
	overflow: hidden;
	width: 100%;
	height: 0.4em;
	transform: translate3d(0,0,0);
}
.modal__progress-fill {
	background-color: hsl(var(--hue),10%,90%);
	width: inherit;
	height: inherit;
	transition: transform 0.1s ease-in-out;
}
.modal__progress-value {
	font-size: 0.75em;
	font-weight: 700;
	line-height: 1.333;
	text-align: right;
}
.modal__sr {
	overflow: hidden;
	position: absolute;
	width: 1px;
	height: 1px;
}
.modal__title {
	font-size: 1.25em;
	font-weight: 500;
	line-height: 1.2;
	margin-bottom: 1.5rem;
	text-align: center;
}
/* state change */
[data-state="2"]:before {
	background-color: hsl(3,90%,60%);
}
[data-state="3"]:before {
	background-color: hsl(138,90%,60%);
}
.modal__icon + .modal__icon,
[data-state="1"] .modal__icon:first-child,
[data-state="2"] .modal__icon:first-child,
[data-state="3"] .modal__icon:first-child,
.modal__content + .modal__content,
[data-state="1"] .modal__content:first-child,
[data-state="2"] .modal__content:first-child,
[data-state="3"] .modal__content:first-child {
	display: none;
}
[data-state="1"] .modal__icon:first-child,
[data-state="2"] .modal__icon:nth-child(2),
[data-state="3"] .modal__icon:nth-child(3),
[data-state="1"] .modal__content:nth-child(2),
[data-state="2"] .modal__content:nth-child(3),
[data-state="3"] .modal__content:nth-child(4) {
	display: block;
}
[data-ready="false"] .modal__content:first-child .modal__actions:nth-of-type(2),
[data-ready="true"] .modal__content:first-child .modal__actions:first-of-type {
	display: none;
}
[data-ready="true"] .modal__content:first-child .modal__actions:nth-of-type(2) {
	display: flex;
}

.loader-container {
  margin-top: 12px;
  text-align: center;
}

.loader {
  border: 4px solid #f3f3f3; 
  border-top: 4px solid #3498db; 
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 2s linear infinite;
}
.no-files {
  margin-top: 12px;
  text-align: center;
  color: red;
  font-size: 1.60rem;
  font-style: italic;
    font-weight: 600;
}

/* Dark theme */
@media (prefers-color-scheme: dark) {
	:root {
		--bg: hsl(var(--hue),10%,35%);
		--fg: hsl(var(--hue),10%,95%);
	}
	.modal {
		background-color: hsl(var(--hue),10%,10%);
		color: hsl(var(--hue),10%,95%);
	}
	.modal__close-button,
	.modal__message,
	.modal__progress-value {
		color: hsl(var(--hue),10%,70%);
	}
	.modal__close-button:hover,
	.modal__close-button:focus-visible {
		color: hsl(var(--hue),10%,80%);
	}
	.modal__file-icon {
		color: hsl(var(--hue),10%,60%);
	}
	.modal__icon--blue g {
		stroke: hsl(223,90%,60%);
	}
	.modal__icon--red g {
		stroke: hsl(3,90%,60%);
	}
	.modal__icon--green g {
		stroke: hsl(138,90%,60%);
	}
	.modal__progress-fill {
		background-color: hsl(var(--hue),10%,20%);
	}
}

/* Animations */
@keyframes fadeSlideIn {
	from { opacity: 0; transform: translateY(33%); }
	to { opacity: 1; transform: translateY(0); }
}
@keyframes sdo {
	to { stroke-dashoffset: 0; }
}

/* Beyond mobile */
@media (min-width: 768px) {
	.modal__actions--center {
		justify-content: center;
		margin-left: -4.125em;
	}
	.modal__body {
		flex-direction: row;
		align-items: center;
	}
	.modal__button {
		width: auto;
	}
	.modal__button + .modal__button {
		margin-top: 0;
		margin-left: 1.5rem;
	}
	.modal__file ~ .modal__button {
		margin-top: 0;
	}
	.modal__file ~ .modal__close-button {
		margin-right: 1.5rem;
	}
	.modal__progress {
		margin-right: 2em;
	}
	.modal__progress + .modal__button {
		margin-top: 0;
	}
	.modal__col + .modal__col {
		margin-top: 0;
		margin-left: 1.875em;
	}
	.modal__title {
		text-align: left;
	}
}
</style>