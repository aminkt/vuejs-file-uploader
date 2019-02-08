<template>
  <div class="file-drag-drop">

    <file-preview
      v-for="(file, key) in previewFiles"
      v-bind:ref="'preview'+parseInt( key )"
      :name="file.name"
      :size="file.size"
      :url="file.url"
      :isSelected="file.selected"
      @select="selectItem($event, key)"
      @remove="removeFile(key)"
    ></file-preview>


    <form ref="fileform" @click="$refs.fileInput.click()" v-if="canAddMoreFile">
      <b-progress :max="100" height="20px" v-if="startUpload" :value="getUploadPercentage" show-progress></b-progress>
      <span class="drop-files" v-if="startUpload">درحال آپلود</span>
      <span class="drop-files" v-else>افزودن فایل</span>
      <input :id="name" type="file" ref="fileInput" :multiple="multiple" style="display:none"
             v-on:change="openFilesByFileInput($event)"/>
    </form>
  </div>
</template>

<style lang="scss">
  .file-drag-drop {
    display: flex;
    flex-wrap: wrap;

    form {
      width: 150px;
      height: 200px;
      border: 2px dashed #ccc;
      border-radius: 3px;
      cursor: pointer;
      text-align: center;
      color: #aaa;
      background: #eeeeee4a;
      margin: 7px;

      span {
        vertical-align: middle;
        margin-top: 50%;
        display: block;
      }

      .file-input {
        display: none;
      }

      .progress {
        height: 20px;
        width: 80%;
        margin: 55px auto 0;
      }
    }

    form:hover {
      background: #e4e4e4;
    }
  }
</style>

<script>
  import FilePreview from './FilePreview'

  export default {
    name: "FileUploader",
    components: {
      FilePreview
    },
    props: {
      name: 'file',
      uploadCallBack: Function,
      deleteCallBack: Function,
      fetchCallBack: Function,
      uploadPercentage: Number,
      value: Array,
      maxFilesNum: Number,
      multiple: false,
      multiSelect: false,
    },
    data: function () {
      return {
        dragAndDropCapable: false,
        /* Hold ready to upload files and after upload will empty. */
        files: [],
        /* Hold uploaded files to preview */
        previewFiles: [],
        localUploadPercentage: 0,
        startUpload: false,
        canAddMoreFile: true
      }
    },
    watch: {
      value() {
        this.previewFiles = this.value
      },
      previewFiles() {
        this.canAddMoreFile = this.previewFiles.length < this.getMaxFilesNum;
      }
    },
    mounted() {
      /* Initial preview files */
      if (this.fetchCallback !== undefined) {
        this.fetchFiles();
      }

      /*
        Determine if drag and drop functionality is capable in the browser
      */
      this.dragAndDropCapable = this.determineDragAndDropCapable();

      /*
        If drag and drop capable, then we continue to bind events to our elements.
      */
      if (this.dragAndDropCapable) {
        /*
          Listen to all of the drag events and bind an event listener to each
          for the fileform.
        */
        ['drag', 'dragstart', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach(function (evt) {
          /*
            For each event add an event listener that prevents the default action
            (opening the file in the browser) and stop the propagation of the event (so
            no other elements open the file in the browser)
          */
          this.$refs.fileform.addEventListener(evt, function (e) {
            e.preventDefault();
            e.stopPropagation();
          }.bind(this), false);
        }.bind(this));

        /*
          Add an event listener for drop to the form
        */
        this.$refs.fileform.addEventListener('drop', function (e) {
          /*
            Capture the files from the drop event and add them to our local files
            array.
          */
          if (this.multiple) {
            for (let i = 0; i < e.dataTransfer.files.length; i++) {
              this.files.push(e.dataTransfer.files[i]);
            }
          } else if (e.dataTransfer.files[0]) {
            this.files.push(e.dataTransfer.files[0]);
          }


          /*
            Instantly upload files
          */
          this.submitFiles();
        }.bind(this));
      }
    },
    computed: {
      /**
       * Return file url from File object
       *
       * @param  file File object
       */
      getFileUrl: function (file) {
        if (file instanceof File) {
          return URL.createObjectURL(file)
        }
        throw "Parameter is not a JS File object!"
      },
      getUploadPercentage: function () {
        if (this.uploadPercentage !== undefined) {
          this.localUploadPercentage = this.uploadPercentage;
        }
        return this.localUploadPercentage;
      },

      /**
       * Return max file to upload
       */
      getMaxFilesNum: function () {
        if (this.maxFilesNum && this.maxFilesNum > 0) {
          return parseInt(this.maxFilesNum);
        }

        if (!this.multiple) {
          return 1;
        }

        return 100;
      }
    },
    methods: {
      /**
       * Open File chooser on click event on form
       */
      openFilesByFileInput(e) {
        let files = e.target.files || e.dataTransfer.files;
        for (let i = 0; i < files.length; i++) {
          this.files.push(files[i]);
        }
        this.clearFileInput();
        this.submitFiles();
      },
      /**
       * Clear file input data.
       */
      clearFileInput() {
        const input = this.$refs.fileInput;
        input.type = 'text';
        input.type = 'file';
      },
      /**
       Determines if the drag and drop functionality is in the
       window
       */
      determineDragAndDropCapable() {
        /*
          Create a test element to see if certain events
          are present that let us do drag and drop.
        */
        let div = document.createElement('div');

        /*
          Check to see if the `draggable` event is in the element
          or the `ondragstart` and `ondrop` events are in the element. If
          they are, then we have what we need for dragging and dropping files.

          We also check to see if the window has `FormData` and `FileReader` objects
          present so we can do our AJAX uploading
        */
        return (('draggable' in div)
          || ('ondragstart' in div && 'ondrop' in div))
          && 'FormData' in window
          && 'FileReader' in window;
      },

      /**
       Gets the image preview for the file.
       */
      getImagePreviews() {
        /*
          Iterate over all of the files and generate an image preview for each one.
        */
        for (let i = 0; i < this.files.length; i++) {
          /*
            Ensure the file is an image file
          */
          if (/\.(jpe?g|png|gif)$/i.test(this.files[i].name)) {
            /*
              Create a new FileReader object
            */
            let reader = new FileReader();

            /*
              Add an event listener for when the file has been loaded
              to update the src on the file preview.
            */
            reader.addEventListener("load", function () {
              this.$refs['preview' + parseInt(i)][0].url = reader.result;
            }.bind(this), false);

            /*
              Read the data for the file in through the reader. When it has
              been loaded, we listen to the event propagated and set the image
              src to what was loaded from the reader.
            */
            reader.readAsDataURL(this.files[i]);
          } else {
            /*
              We do the next tick so the reference is bound and we can access it.
            */
            this.$nextTick(function () {
              this.$refs['preview' + parseInt(i)][0].url = '/images/file.png';
            });
          }
        }
      },

      /**
       Submits the files to the server
       */
      submitFiles: async function () {
        if (this.uploadCallBack === undefined) {
          throw "Upload call back not implemented";
        }
        this.startUpload = true;
        this.uploadCallBack(this.files)
          .then((uploadFiles) => {
            this.previewFiles.push.apply(this.previewFiles, uploadFiles);
            this.$emit('input', this.previewFiles);
            this.files = [];
            this.startUpload = false;
          })
          .catch((error) => {
            this.startUpload = false;
            throw "Callback don't return any data!";
          });
      },

      /**
       Removes a select file the user has uploaded
       @param key  Integer  Index of preview array.
       */
      removeFile(key) {
        if (this.deleteCallBack !== undefined) {
          this.deleteCallBack(this.previewFiles[key].id);
        }
        this.previewFiles.splice(key, 1);
        this.$emit('input', this.previewFiles);
      },

      /**
       * Fetch files and create preview.
       */
      fetchFiles() {
        if (this.fetchCallBack === undefined) {
          console.warn("Fetch images not implemented");
        }

        this.previewFiles = [];

        this.fetchCallback()
          .then((files) => {
            this.previewFiles.push.apply(this.previewFiles, files);
          })
          .catch((error) => {
            throw "Callback don't return any data!";
          });

      },
      selectItem($event, key) {
        if (!this.multiSelect) {
          for (let i=0; i < this.previewFiles.length; i++) {
            this.previewFiles[i].selected = false;
          }
        }
        this.previewFiles[key].selected = $event;
        this.$emit('input', this.previewFiles);
      }
    }
  }
</script>


<style lang="scss">

</style>
