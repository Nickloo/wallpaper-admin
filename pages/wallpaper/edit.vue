<template>
	<view class="uni-container">
		<uni-forms ref="form" :model="formData" validateTrigger="bind">
			<uni-forms-item name="url" label="图片" required>
				<uni-easyinput placeholder="图片地址" v-model="formData.url"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="title" label="标题" required>
				<uni-easyinput placeholder="标题" v-model="formData.title"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="desc" label="描述">
				<uni-easyinput placeholder="描述" v-model="formData.desc"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="categoryIds" label="分类">
				<uni-data-checkbox :multiple="true" v-model="formData.categoryIds"></uni-data-checkbox>
			</uni-forms-item>
			<uni-forms-item name="status" label="状态">
				<uni-easyinput placeholder="状态 0-停用 1-可用" type="number" v-model="formData.status"></uni-easyinput>
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
	import {
		validator
	} from '../../js_sdk/validator/wallpaper.js';

	const db = uniCloud.database();
	const dbCmd = db.command;
	const dbCollectionName = 'wallpaper';

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
				"title": "",
				"desc": "",
				"url": "",
				"categoryIds": [],
				"status": 1
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
				}).catch(() => {}).finally(() => {
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
				db.collection(dbCollectionName).doc(id).field("title,desc,url,categoryIds,status").get().then((res) => {
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