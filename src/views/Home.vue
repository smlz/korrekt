<template>
  <div>
    Files: <input type="file" name="filelist" id="fileslist" @change="newFile"><br/>
    {{pdf && pdf.numPages}}
    <br>
    <input type="range" min="0" :max="pageDivisors.length -1" v-model.number="pagesIdx">
    {{pages}} Pages / {{candidates}} Candidates
    <br>
    <canvas id="preview"></canvas>
  </div>
</template>

<script>
// @ is an alias to /src
import pdfjs from 'pdfjs-dist/webpack.js'

pdfjs.GlobalWorkerOptions.workerSrc = '//mozilla.github.io/pdf.js/build/pdf.worker.js';

export default {
  name: 'home',
  data() {
    return {
      pdf: null,
      pagesIdx: 0
    }
  },
  methods: {
    async newFile(ev) {
      let reader = new FileReader()
      reader.onload = async (e) => {
        this.pdf = await pdfjs.getDocument(e.target.result).promise
        //console.log(this.pdf)
        this.pagesIdx = 0
        this.pdf.getPage(1).then(function(page) {
          console.log('Page loaded');

          var scale = 1.5;
          var viewport = page.getViewport({scale: scale});

          // Prepare canvas using PDF page dimensions
          var canvas = document.getElementById('preview');
          var context = canvas.getContext('2d');
          canvas.height = viewport.height;
          canvas.width = viewport.width;

          // Render PDF page into canvas context
          var renderContext = {
            canvasContext: context,
            viewport: viewport
          };
          var renderTask = page.render(renderContext);
          renderTask.promise.then(function () {
            console.log('Page rendered');
          });
        });
        // let page = await this.pdf.getPage(1)


        // var scale = 1;
        // var viewport = page.getViewport({ scale: scale, });

        // var canvas = document.getElementById('preview');
        // canvas.height = viewport.height;
        // canvas.width = viewport.width;

        // var context = canvas.getContext('2d');
        // canvas.height = viewport.height;
        // canvas.width = viewport.width;

        // var renderContext = {
        //   canvasContext: context,
        //   viewport: viewport
        // };
        // await page.render(renderContext);
        // console.log('done')
      }
      reader.readAsArrayBuffer(ev.srcElement.files[0])
    }
  },
  computed: {
    pageDivisors() {
      if (!this.pdf) {
        return []
      } else {
        let res = []
        for (let i=1; i<=this.pdf.numPages; i++) {
          if (this.pdf.numPages % i === 0) {
            res.push(i)
          }
        }
        return res
      }
    },
    pages() {
      if (this.pageDivisors.length === 0) {
        return 0
      } else {
        return this.pageDivisors[this.pagesIdx]
      }
    },
    candidates() {
      if (!this.pdf) {
        return 0
      } else {
        return this.pdf.numPages / this.pages
      }
    }
  }
}
</script>
