<template>
	<view class="uni-container">
		<uni-forms ref="form" :model="formData" validateTrigger="bind">
			<uni-forms-item name="url" label="图片" required>
				<uni-file-picker v-model="imageData.url" :image-styles="{'width' : '200rpx'}" return-type="object"
					file-mediatype="image" limit="1" mode="grid" @success="(res) => iconUrlSuccess(res,'url')"
					@delete="(res) => iconUrlDelete(res,'url')">
				</uni-file-picker>
			</uni-forms-item>
			<uni-forms-item name="title" label="标题">
				<uni-easyinput placeholder="标题" v-model="formData.title"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="desc" label="描述">
				<uni-easyinput placeholder="描述" v-model="formData.desc"></uni-easyinput>
			</uni-forms-item>

			<uni-forms-item name="categoryIds" label="分类" required>
				<uni-data-checkbox :multiple="true" :localdata="categoryCheckList"
					v-model="formData.categoryIds"></uni-data-checkbox>
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
				},
				categoryList: [],
				imageData: {
					url: []
				},
			}
		},
		computed: {
			categoryCheckList() {
				return this.categoryList.map(v => ({
					value: v._id,
					text: v.name
				}))
			}
		},
		onReady() {
			this.$refs.form.setRules(this.rules)
			db.collection('category').get().then(res => {
				console.log('log', res.result.data);
				this.categoryList = res.result.data
			})
		},
		methods: {
			iconUrlSuccess(res, key) {
				this.formData[key] = res.tempFilePaths[0]
			},
			iconUrlDelete(res, key) {
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