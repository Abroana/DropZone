<template>
  <div class="drop-container">
    <form enctype="multipart/form-data">
      <div
        class="drop-zone"
        :class="{ 'drop-zone--hover': this.isHover}"
        @dragover="isHover = true"
        @dragleave="isHover = false"
        @drop.prevent="drop"
      >
        <input type="file" multiple name="uploader" @change.prevent="drop" class="drop-zone__input">
        <div class="drop-zone__title">Перетащите файл сюда</div>
        <div class="drop-zone__text">или</div>
        <div class="drop-zone__subtitle">выберите файл на компьютере</div>
      </div>
    </form>

    <div class="drop-files">
      <div class="file" v-for="item in fileList" :key="item.name">
        <span class="file__remove" @click="remove(item.name)"/>
        <div class="file__icon-container">
          <img
            :src="getIcon(item)"
            class="file__icon"
            :class="{ 'file__icon--portrait': item.portrait, 'file__icon--file': !item.isImg }"
          >
        </div>
        <div class="file__title">{{ item.name }}</div>
        <div class="file__size">{{ item.kSize }}Kb</div>
        <div class="file__exif" v-if="item.exif">EXIF</div>
        <div class="file__exif-data" v-if="item.exif">{{ item.exif }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import EXIF from 'exif-js'

export default {
  name: 'DropZone',
  data: () => ({
    isHover: false,
    fileList: [],
  }),
  methods: {
    remove(name) {
      this.fileList = this.fileList.filter(item => item.name !== name)
    },
    drop(e) {
      this.isHover = false
      let list
      if (e.dataTransfer) {
        list = [...e.dataTransfer.files]
      } else {
        list = [...e.target.files]
      }
      const that = this
      list.map(item => {
        const fileReader = new FileReader()
        fileReader.onloadend = e => {
          const arr = new Uint8Array(e.target.result).subarray(0, 4)
          let header = ''
          arr.map(el => (header += el.toString(16)))
          item.ext = that.getExt(header)
          item.kSize = parseFloat(item.size / 1024).toFixed(1)
          const images = ['png', 'jpg', 'gif']
          if (images.includes(item.ext)) {
            item.isImg = true
            if (item.ext === 'jpg') {
              EXIF.getData(item, function() {
                item.exif = EXIF.pretty(this)
              })
            }

            const img = new Image()
            img.onload = function() {
              item.portrait = this.height > this.width
              that.fileList.push(item)
            }
            img.src = URL.createObjectURL(item)
          } else {
            that.fileList.push(item)
          }
        }
        fileReader.readAsArrayBuffer(item)
      })
    },
    getExt(header) {
      switch (header.toUpperCase()) {
        case '47494638':
          return 'gif'
        case 'FFD8FFDB':
        case 'FFD8FFE0':
        case '104A4649':
        case 'FFD8FFEE':
          return 'jpg'
        case '504B0304':
        case '504B0506':
        case '504B0708':
        case '504B34':
          return 'zip'
        case '52617221':
          return 'rar'
        case 'FFFB':
        case '494433':
          return 'mp3'
        case '89504E47':
          return 'png'
        case '25504446':
          return 'pdf'
        case '38425053':
          return 'psd'
        case '52494646':
          return 'avi'
        case 'D0CF11E0':
          return 'doc'
        case '7B5C7274':
          return 'rtf'
      }
      return ''
    },
    getIcon(item) {
      switch (item.ext) {
        case 'gif':
        case 'jpg':
        case 'png':
          return URL.createObjectURL(item)
        case 'zip':
          return require('../assets/img/zip.png')
        case 'rar':
          return require('../assets/img/zip-1.png')
        case 'pdf':
          return require('../assets/img/pdf.png')
        case 'psd':
          return require('../assets/img/photoshop.png')
        case 'avi':
          return require('../assets/img/avi.png')
        case 'doc':
          return require('../assets/img/doc.png')
        case 'rtf':
          return require('../assets/img/rtf.png')
        default:
          return require('../assets/img/file.png')
      }
    },
  },
}
</script>

<style lang="postcss">
.drop-container {
  width: 450px;
  text-align: center;
  margin: auto;
}

.drop-zone {
  padding: 40px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  font-size: 15px;
  border: 3px dashed #b0bec5;
  border-radius: 5px;
  color: #78909c;
  position: relative;
}

.drop-zone__input {
  opacity: 0;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.drop-zone--hover {
  border-color: #9bc5da;
  color: #9bc5da;
}

.drop-zone__title {
  font-size: 18px;
}

.drop-zone__text {
  font-size: 13px;
  margin: 2px 0;
}

.drop-zone__subtitle {
  font-style: 15px;
  text-decoration: underline;
}

.drop-zone__btn {
  background: #01579b;
  border-radius: 3px;
  font-size: 14px;
  cursor: pointer;
  color: #fff;
  padding: 5px 10px;
  border: 0;
}

.drop-files {
  margin-top: 30px;
  text-align: left;
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  justify-content: flex-start;
}

.file {
  margin: 15px 10px;
  width: 130px;
  position: relative;
}

.file:hover > .file__remove {
  display: block;
}

.file__icon-container {
  width: 130px;
  height: 130px;
  position: relative;
  overflow: hidden;
}

.file__icon {
  position: absolute;
  left: 50%;
  top: 50%;
  height: 100%;
  width: auto;
  transform: translate(-50%, -50%);
}

.file__icon--file {
  left: 0;
  transform: translate(0, -50%);
}

.file__icon--portrait {
  width: 100%;
  height: auto;
}

.file__title {
  font-size: 11px;
  text-align: left;
  margin-top: 3px;
  color: #607d8b;
  overflow: hidden;
  text-overflow: ellipsis;
}

.file__size {
  margin-top: 3px;
  font-size: 10px;
  color: #607d8b;
  padding: 1px 3px;
  background: rgba(255, 255, 255, 0.7);
  font-weight: 700;
  border-radius: 4px;
  display: inline-block;
}

.file__exif {
  position: absolute;
  top: 0;
  left: 0;
  font-weight: 700;
  font-size: 12px;
  background: rgba(255, 255, 255, 0.7);
  color: #607d8b;
  padding: 3px 4px;
  border-radius: 0 0 3px 0;
  cursor: pointer;
}

.file__exif:hover + .file__exif-data {
  display: block;
}

.file__exif-data {
  display: none;
  position: absolute;
  top: 0;
  left: 0;
  transform: translate(-102%);
  white-space: pre;
  background: #fff;
  border-radius: 3px;
  padding: 5px 10px;
  box-sizing: border-box;
  font-size: 13px;
  z-index: 101;
  user-select: none;
}

.file__remove {
  position: absolute;
  top: 3px;
  right: 3px;
  background-color: #eceff1;
  width: 16px;
  height: 16px;
  background-image: url('../assets/img/remove.png');
  background-size: 10px 10px;
  background-position: center;
  border-radius: 50%;
  background-repeat: no-repeat;
  z-index: 100;
  display: none;
  cursor: pointer;
}
</style>

