# vuejs-file-uploader

Installation
------
Run bellow command:
```
npm i vuejs-file-uploader
```

#Usage
------
Add bellow code to your template part:
```html
<FileUploader
:upload-call-back="upload"
:delete-call-back="delete"
:multiple="true"
:fetch-call-back="get"
:upload-percentage="uploadStatus"
v-model="files"
></FileUploader>
```

Then add bellow code to your js part:
```js
import FileUploader from "vuejs-file-uploader/FileUploader";
export default {
    name: "add-product",
    components: {
      FileUploader
    },
    data: function () {
      return {
        uploadStatus: 0,
        files: []
      };
    },
    mounted: function () {
      this.get();
    },
    methods: {
      delete: function (id) {
        HTTP.delete("delete/address" + id)
          .then(response => {
            // File deleted successfully
          })
          .catch(error => {
            // File not deleted.
          });
      },
      get: function() {
        // Load initial files.
        this.files = [
            {
              id: response.data.data[i].id,
              name: response.data.data[i].file_name,
              size: response.data.data[i].file_size,
              url: response.data.data[i].file_url,
              selected: false
            }
        ]  
      },
      upload: function (files) {
        if (files.length <= 0) {
          throw "No file selected to upload!";
        }

        return new Promise((resolve, reject) => {
          let formData = new FormData();
          /*
            Iteate over any file sent over appending the files
            to the form data.
           */
          for (let i = 0; i < files.length; i++) {
            let file = files[i];

            formData.append("file[" + i + "]", file);
          }

          HTTP.post("upload/address", formData, {
            headers: {
              "Content-Type": "multipart/form-data"
            },
            onUploadProgress: function (progressEvent) {
              this.uploadStatus = parseInt(
                Math.round((progressEvent.loaded * 100) / progressEvent.total)
              );
            }.bind(this)
          })
            .then(response => {
              // File uploaded successfully

              let res = [];
              for (let i = 0; i < response.data.data.length; i++) {
                res.push({
                  id: response.data.data[i].id,
                  name: response.data.data[i].file_name,
                  size: response.data.data[i].file_size,
                  url: response.data.data[i].file_url,
                  selected: false
                });
              }
              this.uploadStatus = 0;
              resolve(res);
            })
            .catch(error => {
              // File not uploaded.
              reject(error);
            });
        });
      }
    }
  };
```