<template>
  <view>
    <view class="container">
      <u--textarea v-model="link" placeholder="请输入内容"></u--textarea>
      <view class="button-container">
        <view class="button-item">
          <u-button type="primary" color="rgb(90,199,37)" :plain="true"
                    text="获取图片" @click="getImgList"></u-button>
        </view>
        <view class="button-item">
          <u-button type="primary" :plain="true" text="清空" @click="clear"></u-button>
        </view>
      </view>
    </view>

    <scroll-view v-if="imgList" scroll-x="true" class="scroll_view_H" scroll-with-animation="true">
      <image :src="item" mode="widthFix" class="scroll_view" block v-for="(item,index) in  imgList"
             :key="index" @longtap="showActionSheet(item,imgList)"></image>
    </scroll-view>
  </view>

</template>
<script>
export default {
  data() {
    return {
      link: "",
      imgList: [],
    };
  },
  methods: {
    async getImgList() {
      uni.showLoading({
        title: "加载中...",
      });
      if (this.link) {
        if (this.link.startsWith("http")) {
          let url = "http://124.221.242.69:9876/xhs?share_url=" + this.data.link;
          await this.sendRequest(url)
        } else {
          try {
            // 匹配链接
            const regex = /http(\S*)，复制/;
            const match_link = this.link.match(regex)[0].slice(0, -3);
            if (match_link) {
              let url = "http://124.221.242.69:9876/xhs?share_url=" + match_link;
              await this.sendRequest(url)
            }
          } catch (e) {
            uni.showToast({
              title: "输入内容有误",
              duration: 2000,
              icon: "error",
            });
          }
        }
      } else {
        uni.showToast({
          title: "请输入内容",
          duration: 2000,
          icon: "error",
        });
      }
    },
    clear() {
      this.link = "";
      this.imgList = []
    },
    // 发送请求
    sendRequest(url) {
      uni.$u.http.get(url, {
        timeout: 10000, //超时时间
      }).then(res => {
        this.imgList = res.data.data;
        uni.showToast({
          title: "获取成功",
          duration: 2000,
        });

      }).catch(err => {
        uni.showToast({
          title: "获取失败",
          duration: 2000,
          icon: "error",
        });
      })
    },
    showActionSheet(img_url,img_list) {
      uni.showActionSheet({
        itemList: ['保存全部', '保存到本地相册'],
        success: function (res) {
          if (res.tapIndex === 1) {
            uni.downloadFile({
              url: img_url,
              success: (res) => {
                console.log(res);
                if (res.statusCode === 200) {
                  uni.saveImageToPhotosAlbum({
                    filePath: res.tempFilePath,
                    success: (res) => {
                      uni.showToast({
                        title: "保存成功",
                        duration: 2000,
                      });
                    },
                    fail: (err) => {
                      uni.showToast({
                        title: "保存失败",
                        duration: 2000,
                        icon: "error",
                      });
                    }
                  });
                }
              }
            });
          } else if (res.tapIndex === 0) {
            for (let i=0;i<img_list.length;i++) {
              uni.downloadFile({
                url: img_list[i],
                success: (res) => {
                  if (res.statusCode === 200) {
                    uni.saveImageToPhotosAlbum({
                      filePath: res.tempFilePath,
                      success: (res) => {
                        uni.showToast({
                          title: '保存成功!',
                          duration: 2000,
                        });
                      },
                      fail: (err) => {
                        uni.showToast({
                          title: '第'+ i +'张图片保存失败!',
                          duration: 2000,
                          icon: "error",
                        });
                      }
                    });
                  }
                }
              });
            }
          }
        },
        fail: function (res) {
          console.log(res);
        }
      });
    },
  }
};
</script>

<style scoped lang="scss">
.container {
  padding: 0;
  margin: 20rpx;
}

.scroll_view_H {
  white-space: nowrap;
  width: 100%;
  margin: 0;
  padding: 0;

  .scroll_view {
    width: 100%;
  }
}

.button-container {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  margin: 15rpx;

  .button-item {
    flex: 1;
    margin-right: 20rpx;
  }
}

</style>