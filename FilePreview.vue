<template>
  <div class="file-preview" :class="{selected: selected}" @click="selectItem"
       v-bind:style="{ backgroundImage: 'url('+previewUrl+')'}">
    <div class="preview-caption">
      <p dir="ltr">{{previewName}}</p>
      <p dir="ltr">{{previewSize}}</p>
    </div>
    <b-button variant="danger" class="remove-image-btn" v-on:click.stop="$emit('remove')"><i class="icon-trash"></i>
    </b-button>
  </div>
</template>

<script>
  export default {
    name: "FilePreview",
    props: {
      // Using value here allows us to be v-model compatible.
      uniqueId: Number|String,
      name: String,
      size: Number,
      url: String,
      file: File,
      isSelected: false,
      noImageIconUrl : 'http://lequytong.com/Content/Images/no-image-02.png',
      fileImageIconUrl : 'https://lh4.ggpht.com/lUcLbewm2mffnhc1_BbZYi5zjEXAzfeYf73cg2SXjFDX3_xNnHGUMkSRdh86TN8HqcSq7Qr3vA=w512'
    },
    data: function () {
      return {
        selected: false
      }
    },
    watch: {
      isSelected() {
        this.selected = this.isSelected
      }
    },
    computed: {
      previewName: function() {
        return this.name
      },
      previewUrl: function () {
        if(this.url){
          return this.url;
        } else if (this.file) {
          return URL.createObjectURL(this.file)
        } else {
          return this.noImageIconUrl;
        }
      },
      previewSize: function () {
        return this.formatBytes(this.size);
      }
    },
    methods: {
      selectItem() {
        this.selected = !this.isSelected;
        this.$emit('select', this.selected);
      },
      formatBytes(a, b) {
        if (0 == a) return "0 Bytes";
        let c = 1024, d = b || 2, e = ["Bytes", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"],
          f = Math.floor(Math.log(a) / Math.log(c));
        return parseFloat((a / Math.pow(c, f)).toFixed(d)) + " " + e[f]
      }
    }
  }
</script>


<style lang="scss">
  .file-preview{
    width: 150px;
    height: 200px;
    background: #eeeeee4a;
    margin: 7px;
    position: relative;
    background-size: contain;
    background-position: center;
    background-repeat: no-repeat;
    border-radius: 2px;
    box-shadow: 0 0 5px 0  #cacaca;

    .remove-image-btn{
      position: absolute;
      border-radius: 100%;
      width: 25px;
      height: 25px;
      text-align: center;
      padding: 0;
      margin: 0;
      top: -5px;
      right: -5px;
    }

    .preview-caption{
      background: #0000006e;
      text-align: center;
      color: #fff;
      position: absolute;
      left: 0;
      right: 0;
      bottom: 0;
      top: 0;
      padding: 5px;
      display: none;
    }
  }

  .file-preview:hover .preview-caption{
    display: block;
  }

  .file-preview.selected{
    box-shadow: 0 0 5px 0 #00b76b;
  }
</style>
