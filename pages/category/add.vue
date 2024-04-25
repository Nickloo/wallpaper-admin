<template>
	<view class="uni-container">
		<uni-forms ref="form" :model="formData" validateTrigger="bind">
			<uni-forms-item name="icon" label="图标">
				<uni-file-picker v-model="imageData.icon" :image-styles="{'width' : '200rpx'}" return-type="object"
					file-mediatype="image" limit="1" mode="grid" @success="(res) => iconUrlSuccess(res,'icon')"
					@delete="(res) => iconUrlDelete(res,'icon')">
				</uni-file-picker>
			</uni-forms-item>
			<uni-forms-item name="name" label="名称">
				<uni-easyinput placeholder="名称" v-model="formData.name"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="desc" label="描述">
				<uni-easyinput placeholder="描述" v-model="formData.desc"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="status" label="状态">
				<uni-easyinput placeholder="状态 0-停用 1-可用" type="number" v-model="formData.status"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="sort" label="排序">
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
	import {
		validator
	} from '../../js_sdk/validator/category.js';

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
				imageData: {
					icon: []
				},
				formData,
				formOptions: {},
				rules: {
					...getValidator(Object.keys(formData))
				}
			}
		},
		onReady() {
			this.$refs.form.setRules(this.rules)
		},
		methods: {
			iconUrlSuccess(res,key) {
				this.formData[key] = res.tempFilePaths[0]
			},
			iconUrlDelete(res,key) {
				this.formData[key] = ''
			},

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
				return db.collection(dbCollectionName).add(value).then((res) => {
					uni.showToast({
						title: '新增成功'
					})
					this.getOpenerEventChannel().emit('refreshData')
					setTimeout(() => uni.navigateBack(), 500)
				}).catch((err) => {
					uni.showModal({
						content: err.message || '请求服务失败',
						showCancel: false
					})
				})
			}
		}
	}
</script>