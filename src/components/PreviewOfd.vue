<template>
  <div class="ofd-preview-container">
    <div ref="ofdContainer" class="ofd-container"></div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue'
import { parseOfdDocument, renderOfd } from 'ofd.js'

export default {
  name: 'PreviewOfd',
  props: {
    src: {
      type: String,
      required: true
    }
  },
  emits: ['rendered', 'error'],
  setup(props, { emit }) {
    const ofdContainer = ref(null)

    onMounted(async () => {
      try {
        // 解析OFD文件
        parseOfdDocument({
          ofd: props.src,
          success() {
            try {
              // 渲染OFD文档的所有页面
              const divs = renderOfd(0, 800)
              // 清空容器
              if (ofdContainer.value) {
                ofdContainer.value.innerHTML = ''
                // 将渲染的页面添加到容器中
                divs.forEach(div => {
                  ofdContainer.value.appendChild(div)
                })
                // 通知父组件渲染完成
                emit('rendered')
              }
            } catch (error) {
              console.error('OFD渲染失败：', error)
              emit('error', error)
            }
          },
          fail(error) {
            console.error('OFD解析失败：', error)
            emit('error', error)
          }
        })
      } catch (error) {
        console.error('OFD处理失败：', error)
        emit('error', error)
      }
    })

    return {
      ofdContainer
    }
  }
}
</script>

<style scoped>
.ofd-preview-container {
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: #f5f5f5;
  padding: 20px;
  text-align: center;
}

.ofd-container {
  background-color: white;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  margin: 0 auto;
  min-height: 800px;
  text-align: center;
  margin: auto;
}
</style>