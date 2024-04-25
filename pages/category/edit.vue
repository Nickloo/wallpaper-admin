<template>
  <view class="uni-container">
    <uni-forms ref="form" :model="formData" validateTrigger="bind">
      <uni-forms-item name="icon" label="">
        <!-- <uni-easyinput placeholder="分类图标" v-model="formData.icon"></uni-easyinput> -->
				<uni-file-picker v-model="formData.icon" :image-styles="{'width' : '200rpx'}"
					return-type="object" file-mediatype="image" limit="1" mode="grid">
				</uni-file-picker>
      </uni-forms-item>
      <uni-forms-item name="name" label="">
        <uni-easyinput placeholder="分类" v-model="formData.name"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="desc" label="">
        <uni-easyinput placeholder="描述" v-model="formData.desc"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="status" label="">
        <uni-easyinput placeholder="状态 0-停用 1-可用" type="number" v-model="formData.status"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="sort" label="">
        <uni-easyinput placeholder="排序" type="number" v-model="formData.sort"></uni-easyinput>
      </uni-forms-item>
      <view class="uni-button-group">
        <button type="primary" class="uni-button" style="width: 100px;" @click="submit">提交</button>
        <navigator open-type="navigateBack" style="margin-left: 15px;">
          <button class="uni-button" style="width: 100px;">返回</button>
        </navigator>
      </view>
    </uni-forms>
  </view>
</template>

<script>
  import { validator } from '../../js_sdk/validator/category.js';

  const db = uniCloud.database();
  const dbCmd = db.command;
  const dbCollectionName = 'category';

  function getValidator(fields) {
    let result = {}
    for (let key in validator) {
      if (fields.includes(key)) {
        result[key] = validator[key]
      }
    }
    return result
  }

  

  export default {
    data() {
      let formData = {
        "icon": "",
        "name": "",
        "desc": "",
        "status": 1,
        "sort": 1
      }
      return {
        formData,
        formOptions: {},
        rules: {
          ...getValidator(Object.keys(formData))
        }
      }
    },
    onLoad(e) {
      if (e.id) {
        const id = e.id
        this.formDataId = id
        this.getDetail(id)
      }
    },
    onReady() {
      this.$refs.form.setRules(this.rules)
    },
    methods: {
      
      /**
       * 验证表单并提交
       */
      submit() {
        uni.showLoading({
          mask: true
        })
        this.$refs.form.validate().then((res) => {
          return this.submitForm(res)
        }).catch(() => {
        }).finally(() => {
          uni.hideLoading()
        })
      },

      /**
       * 提交表单
       */
      submitForm(value) {
        // 使用 clientDB 提交数据
        return db.collection(dbCollectionName).doc(this.formDataId).update(value).then((res) => {
          uni.showToast({
            title: '修改成功'
          })
          this.getOpenerEventChannel().emit('refreshData')
          setTimeout(() => uni.navigateBack(), 500)
        }).catch((err) => {
          uni.showModal({
            content: err.message || '请求服务失败',
            showCancel: false
          })
        })
      },

      /**
       * 获取表单数据
       * @param {Object} id
       */
      getDetail(id) {
        uni.showLoading({
          mask: true
        })
        db.collection(dbCollectionName).doc(id).field("icon,name,desc,status,sort").get().then((res) => {
          const data = res.result.data[0]
          if (data) {
            this.formData = data
            
          }
        }).catch((err) => {
          uni.showModal({
            content: err.message || '请求服务失败',
            showCancel: false
          })
        }).finally(() => {
          uni.hideLoading()
        })
      }
    }
  }
</script>
