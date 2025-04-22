<template>
  <div id="app-wrapper">
    <div class="header-area" v-if="showSidebar">
      <div class="current-file-name">{{ currentFileName }}</div>
      <div class="menu-area">
        <div class="menu-wrapper">
          <div class="menu-trigger">文件列表</div>
          <ul class="file-list">
            <li
              v-for="(file, index) in files"
              :key="file.url"
              :class="{ active: currentFileIndex === index }"
              @click="previewFile(index)"
            >
              {{ file.name }}
            </li>
          </ul>
        </div>
        <button class="download-btn" @click="downloadCurrentFile">下载</button>
        <button class="download-btn" @click="closePreview">关闭</button>
      </div>
    </div>
    <div class="main-content">
      <div v-if="isLoading" class="loading-indicator">加载中...</div>
      <div class="nav-buttons" v-if="showSidebar">
        <button
          class="nav-btn prev viewer-prev"
          :disabled="!hasPrevFile"
          @click="previewFile(currentFileIndex - 1)"
        >
          &lt;
        </button>
        <button
          class="nav-btn next viewer-next"
          :disabled="!hasNextFile"
          @click="previewFile(currentFileIndex + 1)"
        >
          &gt;
        </button>
      </div>
      <component
        :is="currentComponent"
        v-if="!isLoading"
        :src="currentFile"
        style="height: calc(100vh - 50px)"
        @rendered="renderedHandler"
        @error="errorHandler"
        :options="currentComponent === 'VueOfficeExcel' ? options : {}"
      />
    </div>
  </div>
</template>

<script>
import { defineAsyncComponent } from "vue";
import PreviewTxt from "./components/PreviewTxt.vue";
import PreviewNotSupported from "./components/PreviewNotSupported.vue";
import "@vue-office/excel/lib/index.css";
import "viewerjs/dist/viewer.css";

// 定义加载失败提示组件
const ErrorComponent = {
  template: "<div>加载失败</div>",
};

// 定义加载中提示组件
const LoadingComponent = {
  template: "<div>加载中...</div>",
};

const VueOfficeDocx = defineAsyncComponent({
  loader: () => import("@vue-office/docx").then((comp) => comp.default || comp),
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent,
});

const VueOfficeExcel = defineAsyncComponent({
  loader: () =>
    import("@vue-office/excel").then((comp) => comp.default || comp),
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent,
});

const VueOfficePdf = defineAsyncComponent({
  loader: () => import("@vue-office/pdf").then((comp) => comp.default || comp),
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent,
});

const VueOfficePptx = defineAsyncComponent({
  loader: () => import("@vue-office/pptx").then((comp) => comp.default || comp),
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent,
});

const PreviewImage = defineAsyncComponent({
  loader: () =>
    import("./components/PreviewImage.vue").then(
      (comp) => comp.default || comp
    ),
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent,
});

// const PreviewOfd = defineAsyncComponent({
//   loader: () => import('./components/PreviewOfd.vue').then(comp => comp.default || comp),
//   errorComponent: ErrorComponent,
//   loadingComponent: LoadingComponent
// });

export default {
  name: "App",
  components: {
    VueOfficeDocx,
    VueOfficeExcel,
    VueOfficePdf,
    VueOfficePptx,
    PreviewTxt,
    PreviewImage,
    // PreviewOfd,
    PreviewNotSupported,
  },
  data() {
    return {
      files: [],
      currentFile: "",
      currentComponent: null,
      showSidebar: false,
      unsupportedFile: null,
      isLoading: false,
      currentFileName: "",
      currentFileIndex: 0,
      options: {
        xls: false,
        minColLength: 0,
        minRowLength: 0,
        widthOffset: 30,
        heightOffset: 30,
        beforeTransformData: (workbookData) => {
          return workbookData;
        },
        transformData: (workbookData) => {
          return workbookData;
        },
      },
    };
  },
  computed: {
    hasPrevFile() {
      return this.currentFileIndex > 0;
    },
    hasNextFile() {
      return this.currentFileIndex < this.files.length - 1;
    },
  },
  created() {
    debugger;
    window.showFiles = this.showFiles;
    window.addEventListener("keydown", this.handleKeydown);
    const search = new URL(location.href).search;
    if (search) {
      const json = sessionStorage.getItem(search);
      if (json) {
        try {
          var data = JSON.parse(json);
          if (data && data.files) {
            this.showFiles(data.files, data.index, true);
          }
        } catch {}
      }
    } else {
      window.showDemo = this.showDemo;
      this.showDemo(); // 数据演示，实际可去掉这一行
    }
  },
  beforeDestroy() {
    window.removeEventListener("keydown", this.handleKeydown);
  },
  methods: {
    componentMap(type) {
      switch (type) {
        case "docx":
          return "VueOfficeDocx";
        case "xlsx":
          // case 'xls':
          return "VueOfficeExcel";
        case "pdf":
          return "VueOfficePdf";
        // case 'ofd':
        //   return 'PreviewOfd';
        case "pptx":
          // case 'ppt':
          return "VueOfficePptx";
        case "xml":
        case "txt":
        case "css":
          return "PreviewTxt";
        // 图像文件
        case "jpg":
        case "jpeg":
        case "bmp":
        case "gif":
        case "png":
        case "tif":
          return "PreviewImage";
        case "avi":
        case "mpg":
        case "mpeg":
        case "mov":
        case "mp4":
          return "PreviewVideo";
        case "wav":
        case "ram":
        case "mp3":
          return "PreviewSound";
        default:
          return "PreviewNotSupported";
      }
    },
    showDemo() {
      const demoFiles = [
        // 图片 jpg,jpeg,bmp,gif,png,tif
        { name: "花.jpg", url: "/public/files/花.jpg" },
        { name: "小草.jpeg", url: "/public/files/小草.jpeg" },
        { name: "风景.bmp", url: "/public/files/风景.bmp" },
        { name: "小猫.gif", url: "/public/files/小猫.gif" },
        { name: "server.png", url: "/public/files/server.png" },
        { name: "特殊图片2.tif", url: "/public/files/特殊图片2.tif" },
        // Word文档 ( doc,docx )
        { name: "租赁合同.docx", url: "/public/files/租赁合同.docx" },
        // Excel文档 ( xls,xlsx )
        { name: "表格.xlsx", url: "/public/files/表格.xlsx" },
        // PowerPoint文档 ( ppt,pptx )
        { name: "演示文稿.pptx", url: "/public/files/演示文稿.pptx" },
        // 文本文件
        {
          name: "子路、曾皙、冉有、公西华侍坐.txt",
          url: "/public/files/子路、曾皙、冉有、公西华侍坐.txt",
        },
        { name: "数据协议.xml", url: "/public/files/数据协议.xml" },
        { name: "index.css", url: "/public/files/index.css" },
        // pdf文档
        { name: "租赁文件.pdf", url: "/public/files/租赁文件.pdf" },
        // 发票
        { name: "发票.ofd", url: "/public/files/发票.ofd" },
        // 不支持的文件格式
        { name: "旧版表格.xls", url: "/public/files/旧版表格.xls" },
        { name: "旧版文档.doc", url: "/public/files/旧版文档.doc" },
      ];
      this.showFiles(demoFiles, 0);
    },
    showFiles(files, index, nocache) {
      if (!nocache) {
        sessionStorage.setItem(location.href, JSON.stringify({ files, index }));
      }
      if (!Array.isArray(files)) {
        files = [files];
      }
      if (!index || isNaN(index)) {
        index = 0;
      }

      // 获取当前URL中的参数部分
      const currentUrlParams = new URL(location.href).search;

      this.files = files.map((file) => {
        const fileUrl = file.url;
        // 如果文件URL已经包含参数，使用&连接，否则使用?
        const connector = fileUrl.includes("?") ? "&" : "?";
        const finalUrl = currentUrlParams
          ? `${fileUrl}${connector}${currentUrlParams.slice(1)}`
          : fileUrl;

        return {
          name: typeof file === "string" ? file : file.name,
          url: finalUrl,
          type: file.type || file.name.split(".").pop().toLowerCase(),
        };
      });

      this.showSidebar = files.length > 1;

      if (this.files.length > 0) {
        this.previewFile(index);
      }
    },
    previewFile(index) {
      const file = this.files[index];
      if (!file) {
        return;
      }
      this.currentFileIndex = index;
      const componentName = this.componentMap(file.type);
      if (componentName) {
        this.currentFile = file.url;
        this.currentComponent = componentName;
        this.unsupportedFile = null;
        this.currentFileName = file.name;
      } else {
        this.unsupportedFile = file.name;
      }
    },
    renderedHandler() {
      this.isLoading = false;
      console.log("渲染完成");
    },
    errorHandler() {
      this.isLoading = false;
      console.log("渲染失败");
    },
    async downloadCurrentFile() {
      try {
        const currentFileObj = this.files[this.currentFileIndex];
        if (!currentFileObj) return;

        // 如果是URL，先获取文件内容再下载
        this.downloadFromUrl(currentFileObj.url, currentFileObj.name);
      } catch (error) {
        console.error("下载文件失败:", error);
      }
    },
    downloadFromUrl(url, filename) {
      const link = document.createElement("a");
      link.href = url;
      link.download = filename;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    closePreview() {
      // this.currentFile = '';
      // this.currentComponent = null;
      // this.currentFileName = '';
      // this.isLoading = false;
      try {
        window.close();
      } catch {}
    },
    handleKeydown(event) {
      if (this.showSidebar) {
        if (event.key === "ArrowLeft") {
          this.previewFile(this.currentFileIndex - 1);
        } else if (event.key === "ArrowRight") {
          this.previewFile(this.currentFileIndex + 1);
        }
      }
    },
  },
};
</script>

<style>
body {
  height: 100vh;
  margin: 0;
  padding: 0;
}

#app-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  margin: 0;
  padding: 0;
}

.header-area {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  height: 50px;
  background-color: #fff;
  border-bottom: 1px solid #e6e6e6;
  display: flex;
  align-items: center;
  padding: 0 20px;
  justify-content: space-between;
}

.current-file-name {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
}

.menu-area {
  display: flex;
  align-items: center;
  gap: 12px;
}

.download-btn {
  padding: 8px 16px;
  background-color: #409eff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.download-btn:hover {
  background-color: #66b1ff;
}

.menu-wrapper {
  position: relative;
}

.menu-trigger {
  padding: 8px 16px;
  cursor: pointer;
  color: #303133;
  background-color: #f5f7fa;
  border-radius: 4px;
}

.menu-wrapper:hover .file-list {
  display: block;
}

.file-list {
  display: none;
  position: absolute;
  top: 100%;
  right: 0;
  list-style: none;
  padding: 8px 0;
  margin: 0px 0 0;
  background-color: #fff;
  border: 1px solid #e6e6e6;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  min-width: 150px;
  z-index: 1000;
}

.file-list li {
  padding: 8px 16px;
  cursor: pointer;
  transition: all 0.3s;
  color: #303133;
  white-space: nowrap;
}

.file-list li:hover {
  background-color: #f5f7fa;
}

.file-list li.active {
  color: #409eff;
  background-color: #ecf5ff;
}

.main-content {
  flex: 1;
  position: relative;
  height: calc(100vh - 50px);
  margin-top: 50px;
  /* 添加这行，为固定定位的header留出空间 */
}

.nav-buttons {
  position: fixed;
  top: 50%;
  left: 0;
  right: 0;
  transform: translateY(-50%);
  pointer-events: none;
  z-index: 100;
  display: flex;
  justify-content: space-between;
  padding: 0 20px;
  /* 添加左右内边距 */
}

.nav-btn {
  pointer-events: auto;
  background: rgba(0, 0, 0, 0.3);
  border: none;
  color: white;
  width: 40px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s;
  border-radius: 4px;
  position: relative;
  top: 0%;
  transform: translateY(-50%);
}

.nav-btn:hover {
  background: rgba(0, 0, 0, 0.5);
}

.nav-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.nav-btn .arrow {
  font-size: 24px;
  line-height: 1;
}

.loading-indicator {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 18px;
  color: #409eff;
}

.unsupported-tip {
  padding: 40px;
  color: #d32f2f;
  font-size: 18px;
  text-align: center;
}

.nav-btn {
  position: relative;
}

.nav-btn::before {
  content: "";
  position: absolute;
  width: 20px;
  height: 20px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
}

/* 
.vue-office-docx-main{
  height: 100vh;
}
.docx-wrapper{
  height: 100vh;
} */
</style>