<template>
	<div class="root" :class="[dropState, uploadState]">
		<form :target="uploads.length && uploads[0].ifr" :action="url" method="post" enctype="multipart/form-data">
			<input :id="'inp'+_uid" name="file" type="file" :accept="validFileTypes" :capture="capture" @change="onchange" :multiple="multiple">
			<input v-if="!hasFileAPI && data !== undefined" type="hidden" name="data" :value="data">
		</form>
		<iframe v-for="item in uploads" v-if="item.ifr" :key="item.ifr" :name="item.ifr" src="about:blank" @load="onload($event.target, item.ifr)"></iframe>
		<div v-if="$slots.default" class="slot"><slot></slot></div>
		<label :class="(loading) ? 'loading' : ''" :for="!uploads.length || multiple ? 'inp'+_uid : ''" @dragenter.prevent.stop="enter" @dragleave.prevent.stop="leave" @dragover.prevent.stop="over" @drop.prevent.stop="drop" :title="uploadInfo">{{buttonLabel}}
      <span class="loading-icon" v-if="loading">
        <svg width="20px"  height="20px"  xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" preserveAspectRatio="xMidYMid" class="lds-flickr" style="background: none;">
            <circle cy="50" cx="56.5712" fill="#63DBC1" r="20">
              <animate attributeName="cx" calcMode="linear" values="30;70;30" keyTimes="0;0.5;1" dur="1" begin="-0.5s" repeatCount="indefinite"></animate>
            </circle>
            <circle cy="50" cx="43.4288" fill="#ffffff" r="20">
              <animate attributeName="cx" calcMode="linear" values="30;70;30" keyTimes="0;0.5;1" dur="1" begin="0s" repeatCount="indefinite"></animate>
            </circle>
            <circle cy="50" cx="56.5712" fill="#63DBC1" r="20">
              <animate attributeName="cx" calcMode="linear" values="30;70;30" keyTimes="0;0.5;1" dur="1" begin="-0.5s" repeatCount="indefinite"></animate>
              <animate attributeName="fill-opacity" values="0;0;1;1" calcMode="discrete" keyTimes="0;0.499;0.5;1" repeatCount="indefinite" dur="1s"></animate>
            </circle>
          </svg>
    </span></label>
    <span class="current-value">{{currentValue}}</span>
  </div>
</template>
<style scoped>


	.root {
		position: relative;
		display: block;
	}

	form,
	iframe {
		display: none;
	}

	.notice,
	.slot {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}



	label {
    display:inline-block;
		background-color: #319bf5;
		cursor: pointer;
    color: #fff;
    border-radius: 2px;
    text-align: center;
    padding: 4px 12px 4px 12px;
    letter-spacing: 1px;
    text-transform: uppercase;
    font-weight: 700;
    font-size: 10px;
    position: relative;
    transition: all 0.2s linear;
	}

  label.loading{
    padding: 4px 24px 4px 24px;
  }

  .loading-icon{
    position: absolute;
    top: 2px;
    right: 8px;
  }

  label:hover{
    background-color: #3ad1b1;
  }

  .current-value{
    font-weight: 100;
    padding: 0 0 0 8px;
  }

	.notice {
		font-weight: bold;
		font-size: 250%;
	}



</style>
<script>
"use strict";

var hasFileAPI = window.FileReader && window.FormData;

// hasFileAPI = false

function isIFrameWindow(iframeElt) {

	try {
		return iframeElt.contentWindow;
	} catch(ex) {}
	return null;
}

function hasDataTransferFileSupport(dataTransfer) {

	if ( !('types' in dataTransfer) )
		return false;
	for ( var i = 0; i < dataTransfer.types.length; ++i )
		if ( dataTransfer.types[i] === 'Files' )
			return true;
	return false;
}

module.exports = {
	props: {
		url: {
			type: String,
			required: true,
		},
    buttonLabel:{
      type: String,
      default: 'Select file'
    },
    currentValue:{
      type:String
    },
		multiple: {
			type: Boolean,
			default: false,
		},
		image: {
			type: Boolean,
			default: false,
		},
    serverResult:{
      type:String,
      default: null
    },
		capture: {
			type: Boolean,
			default: false,
		},
    validFileTypes:{
      type:String,
      default: ".gif|.png|.jpg|.jpeg"
    },
		accept: {
			type: Function,
			default: function(filename) {
        return true
        }
    },
		done: {
			type: Function,
			default: function(status, responseText, feedback) { status !== undefined && feedback(status >= 200 && status < 400) },
		},
		data: {
			type: String,
		},
	},
	data: function() {
		return {
			hasFileAPI: hasFileAPI,
			dropState: '',
			uploadState: '',
			uploads: [],
			total: 0,
			loaded: 0,
			tick: 0,
      loading: false
		}
	},

	computed: {
		uploadInfo: function() {

			var filenameList = [];
			for ( var i = 0; i < this.uploads.length; ++i )
				Array.prototype.push.apply(filenameList, this.uploads[i].filenameList);
			return filenameList.map(function(item) { return '\u2022 ' + item }).join('\n');
		},
		progressStyle: function() {

			if ( this.uploads.length === 0 )
				return undefined;

			var loadedRatio = this.loaded / this.total;
			if ( isNaN(loadedRatio) )
				return { width: '50%', marginLeft: (50 - Math.abs(this.tickNext() % 50*2 - 50))+'%' };
			else
				return { width: loadedRatio*100+'%' };
		},
	},

	watch: {
		'uploads.length': function(length) {

			if ( length !== 0 )
				return;
			this.loaded = 0;
			this.total = 0;
		}
	},

	methods: {
		tickNext: function() {

			window.setTimeout(function() {

				this.tick++;
			}.bind(this), 250);
			return this.tick;
		},

		uploaded: function(status, responseText) {
      this.loading = false;
			var feedback = function(success) {

				this.uploadState = success ? 'uploadSuccess' : 'uploadFailure';
				window.setTimeout(function() {

					this.uploadState = '';
				}.bind(this), 1000);
			}.bind(this);

      this.serverResult = JSON.parse(responseText).filename;
      this.currentValue = this.serverResult;
			this.done(status, responseText, feedback);
		},
		uploadFiles: function(files) {

			var xhr = new XMLHttpRequest();

			var info = {
				free: function() {

					xhr.onreadystatechange = null;
					xhr.upload.onprogress = null;
					if ( xhr.readyState !== 4 )
						xhr.abort();
					this.uploads.splice(this.uploads.indexOf(info), 1);
				}.bind(this),
				filenameList: [],
			}
			this.uploads.push(info);

			var prevLoadedBytes = 0;
			xhr.upload.onprogress = function(ev) {

				if ( !ev.lengthComputable )
					return;
				this.loaded += ev.loaded - prevLoadedBytes;
				prevLoadedBytes = ev.loaded;
			}.bind(this);

			xhr.onreadystatechange = function() {

				if ( xhr.readyState !== 4 )
					return;
				info.free();
				this.uploaded(xhr.status, xhr.responseText);
			}.bind(this);

			var fd = new FormData;
			for ( var i = 0; i < files.length; ++i ) {

				info.filenameList.push(files[i].name);
				this.total += files[i].size;
				fd.append('file', files[i]);
			}

			if ( 'data' in this )
				fd.append('data', this.data);
      this.loading = true;
			xhr.open('POST', this.url, true);

			let csrfToken = document.querySelector('meta[name="csrf-token"]').content;
		  if(csrfToken!=null){
			  xhr.setRequestHeader('X-CSRF-TOKEN', csrfToken);
		  }

			xhr.send(fd);
		},
		enter: function(ev) {

			if ( this.uploads.length && !this.multiple ) {

				this.dropState = 'dropDenied';
				return;
			}

			if ( hasFileAPI && hasDataTransferFileSupport(ev.dataTransfer) ) {

				if ( ('items' in ev.dataTransfer) && ev.dataTransfer.items.length > 1 && !this.multiple )
					this.dropState = 'dropDenied';
				else
					this.dropState = 'dropAllowed';
			} else {

				this.dropState = 'dropUndefined';
			}
		},
		leave: function(ev) {

			this.dropState = '';
		},
		over: function(ev) {

			if ( hasFileAPI && hasDataTransferFileSupport(ev.dataTransfer) )
				ev.dataTransfer.dropEffect = (this.dropState === 'dropDenied' ? 'none' : '');
		},

		drop: function(ev) {

			window.setTimeout(function() {

				this.dropState = '';
			}.bind(this), 500);

			if ( this.uploads.length && !this.multiple ) {

				this.dropState = 'dropDenied';
				return;
			}

			if ( hasFileAPI && hasDataTransferFileSupport(ev.dataTransfer) ) {

				var files = Array.prototype.slice.call(ev.dataTransfer.files);
				var acceptFiles = !files.some(function(file) {

					return !this.accept(file.name);
				}.bind(this));

				if ( !acceptFiles || ev.dataTransfer.files.length === 0 || ev.dataTransfer.files.length > 1 && !this.multiple ) {

					this.dropState = 'dropDenied';
				} else {

					this.uploadFiles(ev.dataTransfer.files);
				}
			} else {

				this.$nextTick(function() {

					if ( ev.target.htmlFor )
						window.document.getElementById(ev.target.htmlFor).click();
				});
			}
		},
		onchange: function(ev) {

			var formElt = ev.target.form;
			if ( hasFileAPI && 'files' in ev.target ) {

				this.uploadFiles(ev.target.files);
				formElt.reset();
			} else {

				this.total += NaN;
				var name = this._uid+'ifr'+Date.now();

				var filename = ev.target.value.match(/[^\/\\]*$/)[0];
				if ( !this.accept(filename) ) {

					formElt.reset();
					this.dropState = 'dropDenied';

					window.setTimeout(function() {

						this.dropState = '';
					}.bind(this), 500);

					return;
				}

				this.uploads.unshift({
					ifr: name,
					free: function() {

						window.document.getElementsByName(name)[0].src = "about:blank";

						for ( var i = 0; i < this.uploads.length; ++i )
							if ( this.uploads[i].ifr === name )
								this.uploads.splice(i, 1);
					}.bind(this),
					filenameList: [filename],
				});

				this.$nextTick(function() {

					formElt.submit();
					this.$nextTick(function() {

						formElt.reset();
					});
				});
			}
		},

		onload: function(iframeElt, name) {

			var iframeWin = isIFrameWindow(iframeElt);
			if ( iframeWin !== null && iframeWin.document.location.href === 'about:blank' )
				return;

			for ( var i = 0; i < this.uploads.length; ++i )
				if ( this.uploads[i].ifr === name )
					this.uploads[i].free();

			this.uploaded(undefined, iframeWin !== null ? iframeWin.document.documentElement.innerText : '');
		},

		free: function() {

			for ( var i = this.uploads.length-1; i >= 0 ; --i )
				this.uploads[i].free();
		}
	}
}

</script>
