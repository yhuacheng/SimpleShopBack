<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="searchForm">
				<el-form-item label="商品搜索">
					<el-input size="small" v-model="searchForm.name" placeholder="请输入商品名称 / ASIN"></el-input>
				</el-form-item>
				<el-form-item label="所属国家">
					<el-select v-model="searchForm.country" placeholder="请选择所属国家" size="small">
						<el-option value="0" label="全部国家"></el-option>
						<el-option v-for="item in countryData" :key="item.Id" :label="item.CountryName" :value="item.Id"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="商品类别">
					<el-select v-model="searchForm.type" placeholder="请选择所属类别" size="small">
						<el-option value="0" label="全部类别"></el-option>
						<el-option v-for="item in typeData" :key="item.Id" :label="item.Name" :value="item.Id"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="状态">
					<el-select v-model="searchForm.state" placeholder="状态" size="small">
						<el-option value="0" label="全部状态"></el-option>
						<el-option value="-1" label="未上架"></el-option>
						<el-option value="1" label="已上架"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item>
					<el-button size="small" @click="searchData">查询</el-button>
					<el-button size="small" @click="resetSearch">重置</el-button>
					<el-button size="small" type="primary" @click="handleAdd">新增</el-button>
					<el-button type="warning" size="small" @click="exportExcel">导出</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table border :data="tableData" @selection-change="selsChange" @row-click="rowClick" v-loading="listLoading" style="width: 100%"
		 id="tableData" ref='tableData'>
			<el-table-column type="selection" align="center"></el-table-column>
			<el-table-column type="index" label="#" align="center"></el-table-column>
			<el-table-column prop="picture" label="商品图" align="center">
				<template slot-scope="scope">
					<img style="width: 40px;height: 40px;" v-if="scope.row.ProductUrl" :src="scope.row.ProductUrl" @click.stop="showImage(scope.row.ProductUrl)" />
				</template>
			</el-table-column>
			<el-table-column prop="ProductName" label="商品名称" align="center" :show-overflow-tooltip='true'></el-table-column>
			<el-table-column prop="ASIN" label="ASIN" align="center"></el-table-column>
			<el-table-column prop="CountryName" label="所属国家" align="center"></el-table-column>
			<el-table-column prop="Name" label="商品类别" align="center"></el-table-column>
			<el-table-column prop="Price" label="价格" align="center">
				<template slot-scope="scope">
					<span>{{scope.row.Price}} {{scope.row.Currency}}</span>
				</template>
			</el-table-column>
			<el-table-column prop="AddTime" label="添加时间" align="center"></el-table-column>
			<el-table-column prop="State" label="状态" align="center">
				<template slot-scope="scope">
					<el-tag size="small" type="danger" v-if="scope.row.State===-1">未上架</el-tag>
					<el-tag size="small" type="success" v-if="scope.row.State===1">已上架</el-tag>
				</template>
			</el-table-column>
			<el-table-column label="操作" align="center" width="100">
				<template v-slot="scope">
					<el-button size="small" type="primary" @click.stop="handleEdit(scope.$index, scope.row)">编辑</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button size="small" type="success" @click="handleState(1)" :disabled="selsData.length<1">上架</el-button>
			<el-button size="small" type="warning" @click="handleState(-1)" :disabled="selsData.length<1">下架</el-button>
			<el-button size="small" type="danger" @click="handleDelete()" :disabled="selsData.length<1">删除</el-button>
			<el-pagination style="float: right;" @size-change="handleSizeChange" @current-change="handleCurrentChange"
			 :current-page="pageIndex" :page-sizes="[10, 20, 50, 100,1000]" :page-size="10" layout="total, sizes, prev, pager, next, jumper"
			 :total="total">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog :title="title" :visible.sync="editModal" :close-on-click-modal="false" :before-close="closeModal" width="70%">
			<el-form :model="editForm" label-width="100px" :rules="rules" ref="editForm">
				<el-row>
					<el-col :span="24">
						<!-- 新增页面商品图片 图片文件与表单数据一起提交-->
						<el-form-item label="商品图片" prop="picture">
							<el-upload action="" list-type="picture-card" multiple :limit="6" :file-list="fileListAdd" :on-remove="handleAddRemove"
							 :on-change="handleAddChange" :auto-upload="false" accept="image/jpeg,image/png,image/gif,image/bmp" :class="{'hide':hideUploadAdd}"
							 :disabled="disUpload">
								<i class="el-icon-plus"></i>
								<div class="el-upload__tip warning" slot="tip">注意：最多上传6张图片，第1张为封面图，每张图片不能大于5M，建议上传 600*600 尺寸</div>
							</el-upload>
							<div v-if="doType=='edit'">
								<el-button type="danger" size="mini" @click='editImg'>重新编辑图片</el-button>
								<el-button type="warning" size="mini" @click='resetImg'>重置商品图片</el-button>
							</div>
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="24">
						<el-form-item label="商品名称" prop="name">
							<el-input v-model="editForm.name"></el-input>
						</el-form-item>
					</el-col>
					<el-col :span="6">
						<el-form-item label="ASIN" prop="asin">
							<el-input v-model="editForm.asin">
							</el-input>
						</el-form-item>
					</el-col>
					<el-col :span="6">
						<el-form-item label="所属国家" prop="country">
							<el-select v-model="editForm.country" placeholder="请选择所属国家" class="w100" @change="getCurrency">
								<el-option v-for="item in countryData" :key="item.Id" :label="item.CountryName" :value="item.Id"></el-option>
							</el-select>
						</el-form-item>
					</el-col>
					<el-col :span="6">
						<el-form-item label="商品类别" prop="type">
							<el-select v-model="editForm.type" placeholder="请选择所属类别" class="w100">
								<el-option v-for="item in typeData" :key="item.Id" :label="item.Name" :value="item.Id"></el-option>
							</el-select>
						</el-form-item>
					</el-col>
					<el-col :span="6">
						<el-form-item label="价格" prop="price">
							<el-input v-model="editForm.price">
								<template slot="append">{{editForm.currency}}</template>
							</el-input>
						</el-form-item>
					</el-col>
					<el-col :span="24">
						<el-form-item label="商品描述" prop="description">
							<quill-editor v-model="editForm.description" ref="myQuillEditor" class="editor-box" :options="editorOption"></quill-editor>
						</el-form-item>
					</el-col>
				</el-row>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="closeModal">取消</el-button>
				<el-button type="primary" v-if="doType==='add'" @click="addSubmit" :loading="btnLoading">提交</el-button>
				<el-button type="primary" v-if="doType==='edit'" @click="editSubmit" :loading="btnLoading">提交</el-button>
			</div>
		</el-dialog>

		<!-- 图片大图预览 -->
		<el-dialog title="图片预览" :visible.sync="ViewImageModal" width="60%">
			<div class="txt-c">
				<img :src='ViewImageUrl' />
			</div>
		</el-dialog>

	</section>
</template>

<script>
	import FileSaver from 'file-saver'
	import XLSX from 'xlsx'

	import {
		goodsList,
		goodsAdd,
		goodsEdit,
		goodsState,
		goodsDelete,
		countryList,
		typeList
	} from '@/api/api'

	import {
		quillEditor
	} from 'vue-quill-editor'
	import 'quill/dist/quill.core.css'
	import 'quill/dist/quill.snow.css'
	import 'quill/dist/quill.bubble.css'

	export default {
		name: 'goods',
		components: {
			quillEditor
		},
		data() {
			return {
				title: '',
				pageIndex: 1,
				pageSize: 10,
				total: 0,
				doType: '',
				editModal: false,
				listLoading: false,
				btnLoading: false,
				tableData: [],
				countryData: [],
				typeData: [],
				selsData: [],
				selectId: '',
				searchForm: {
					name: '',
					country: '0',
					type: '0',
					state: '0'
				},
				editForm: {
					name: '',
					asin: '',
					country: '',
					type: '',
					price: '',
					currency: '',
					description: ''
				},
				fileListAdd: [],
				hideUploadAdd: false,
				fileListData: [],
				disUpload: false,
				editorOption: {
					placeholder: '',
					theme: 'snow',
				},
				ViewImageModal: false,
				ViewImageUrl: '',
				rules: {
					name: {
						required: true,
						message: '请输入商品名称',
						trigger: 'blur'
					},
					asin: {
						required: true,
						message: '请输入商品ASIN',
						trigger: 'blur'
					},
					country: {
						required: true,
						message: '请选择所属国家',
						trigger: 'blur'
					},
					type: {
						required: true,
						message: '请选择商品类别',
						trigger: 'blur'
					},
					price: [{
							required: true,
							message: '请输入商品价格',
							trigger: 'blur'
						},
						{
							pattern: /^[0-9]+([.]{1}[0-9]+){0,1}$/,
							message: '金额格式不正确',
							trigger: 'blur'
						}
					]
				}
			}
		},
		created() {
			this.getData()
			this.getCountryData()
			this.getTypeData()
		},
		methods: {
			// 获取列表数据
			getData() {
				let _this = this
				_this.listLoading = true
				let params = {
					Name: _this.searchForm.name,
					countryId: _this.searchForm.country,
					productTypeId: _this.searchForm.type,
					State: _this.searchForm.state,
					pageIndex: _this.pageIndex,
					pageSize: _this.pageSize
				}
				goodsList(params).then(res => {
					_this.listLoading = false
					_this.tableData = res.Entity
					_this.total = Number(res.TotalCount)
				}).catch((e) => {})
			},

			// 获取国家数据
			getCountryData() {
				let _this = this
				let params = {
					Name: '',
					pageIndex: 1,
					pageSize: 100000000
				}
				countryList(params).then(res => {
					_this.countryData = res.Entity
				}).catch((e) => {})
			},

			//选择国家后获取货币
			getCurrency() {
				let _this = this
				let countryData = _this.countryData
				let country = _this.editForm.country
				for (let x in countryData) {
					if (country == countryData[x].Id)
						_this.editForm.currency = countryData[x].Currency
				}
			},

			// 获取类别数据
			getTypeData() {
				let _this = this
				let params = {
					Name: '',
					pageIndex: 1,
					pageSize: 100000000
				}
				typeList(params).then(res => {
					_this.typeData = res.Entity
				}).catch((e) => {})
			},

			//查询
			searchData() {
				let _this = this
				_this.pageIndex = 1
				_this.getData()
			},

			//重置
			resetSearch() {
				let _this = this
				_this.pageIndex = 1
				_this.searchForm = {
						name: '',
						country: '0',
						type: '0',
						state: '0'
					},
					_this.getData()
			},

			// 显示新增框
			handleAdd() {
				let _this = this
				_this.title = '新增商品'
				_this.doType = 'add'
				_this.editModal = true
			},

			// 显示编辑框
			handleEdit(index, row) {
				let _this = this
				_this.title = '编辑商品'
				_this.doType = 'edit'
				_this.selectId = row.Id
				_this.editModal = true
				_this.editForm.name = row.ProductName
				_this.editForm.asin = row.ASIN
				_this.editForm.country = Number(row.CountryId)
				_this.editForm.type = Number(row.ProductTypeId)
				_this.editForm.price = row.Price
				_this.editForm.currency = row.Currency
				_this.editForm.description = row.ProductDescribe

				//图片转换为 url:'xxx' 格式才能回显
				let img = ''
				if (row.ProductUrl) {
					if (row.Url) {
						img = row.ProductUrl + ',' + row.Url
					} else {
						img = row.ProductUrl
					}
					let imgArr = img.split(',')
					let formatImgArr = []
					for (let x in imgArr) {
						let obj = new Object()
						obj.url = imgArr[x]
						formatImgArr.push(obj)
					}
					_this.fileListAdd = formatImgArr //赋值给图片上传控件回显数据
					_this.fileListData = formatImgArr //记录获取到的原始数据
				}
				_this.disUpload = true //修改时默认先禁用图片上传功能
				_this.hideUploadAdd = true //隐藏图片上传控件
			},

			//关闭新增编辑弹窗
			closeModal() {
				let _this = this
				_this.editModal = false
				_this.btnLoading = false
				_this.$refs['editForm'].resetFields()
				_this.editForm = {
					name: '',
					asin: '',
					country: '',
					type: '',
					price: '',
					currency: '',
					description: ''
				}
				_this.fileListAdd = []
				_this.hideUploadAdd = false
				_this.fileListData = []
				_this.disUpload = false
			},

			//编辑页面打开图片编辑功能
			editImg() {
				let _this = this
				_this.disUpload = false
				_this.fileListAdd = []
				_this.hideUploadAdd = false
			},

			//编辑页面重置图片的回显
			resetImg() {
				let _this = this
				_this.disUpload = true
				_this.fileListAdd = _this.fileListData
				_this.hideUploadAdd = true
			},

			//新增提交
			addSubmit() {
				let _this = this
				_this.$refs.editForm.validate((valid) => {
					if (valid) {
						_this.btnLoading = true

						//创建formData 用formData形式传参
						let params = new FormData();
						params.append('ProductName', _this.editForm.name)
						params.append('ASIN', _this.editForm.asin)
						params.append('CountryId', _this.editForm.country)
						params.append('ProductTypeId', _this.editForm.type)
						params.append('Price', _this.editForm.price)
						params.append('ProductDescribe', _this.editForm.description)
						_this.fileListAdd.map(item => {
							params.append("image", item.raw);
						})

						goodsAdd(params).then(res => {
							_this.btnLoading = false
							_this.closeModal()
							_this.getData()
						}).catch((e) => {
							_this.btnLoading = false
						})
					}
				})
			},

			//编辑提交
			editSubmit() {
				let _this = this
				_this.$refs.editForm.validate((valid) => {
					if (valid) {
						_this.btnLoading = true

						//创建formData 用formData形式传参
						let params = new FormData();
						params.append('Id', _this.selectId)
						params.append('ProductName', _this.editForm.name)
						params.append('ASIN', _this.editForm.asin)
						params.append('CountryId', _this.editForm.country)
						params.append('ProductTypeId', _this.editForm.type)
						params.append('Price', _this.editForm.price)
						params.append('ProductDescribe', _this.editForm.description)
						_this.fileListAdd.map(item => {
							params.append("image", item.raw);
						});

						goodsEdit(params).then(res => {
							_this.btnLoading = false
							_this.closeModal()
							_this.getData()
						}).catch((e) => {
							_this.btnLoading = false
						})
					}
				})
			},

			//商品上架下架
			handleState(val) {
				let _this = this
				let txt = ''
				if (val == 1) {
					txt = '上架'
				}
				if (val == -1) {
					txt = '下架'
				}
				let ids = _this.selsData.map(item => item.Id) //选中的数据
				let num = _this.selsData.length //选中的数量
				_this.$confirm('确认【' + txt + '】选中的【' + num + '】个商品吗？', '信息提示', {
					type: 'warning'
				}).then(() => {
					let params = {
						Id: ids,
						State: val
					}
					goodsState(params).then((res) => {
						_this.getData()
					})
				}).catch(() => {})
			},

			//商品删除
			handleDelete() {
				let _this = this
				let ids = _this.selsData.map(item => item.Id) //选中的数据
				let num = _this.selsData.length //选中的数量
				_this.$confirm('确认【 删除 】选中的【' + num + '】个商品吗？', '信息提示', {
					type: 'warning'
				}).then(() => {
					let params = {
						Id: ids
					}
					goodsDelete(params).then((res) => {
						_this.getData()
					})
				}).catch(() => {})
			},

			// 商品图片上传
			handleAddChange(file, fileList) {
				const isLt5M = file.size / 1024 / 1024 < 5;
				if (!isLt5M) {
					this.$message.error('图片不能大于5M');
					fileList.splice(-1, 1);
				} else {
					this.fileListAdd = fileList;
				}
				// 上传文件>=限制个数时隐藏上传组件
				if (fileList.length >= 6) {
					this.hideUploadAdd = true;
				}
			},
			// 移除文件
			handleAddRemove(file, fileList) {
				this.fileListAdd = fileList
				this.hideUploadAdd = false;
			},

			//查看大图
			showImage(url) {
				let _this = this
				_this.ViewImageModal = true
				_this.ViewImageUrl = url
			},

			//选中行
			rowClick(row) {
				let _this = this
				let findResult = _this.selsData.findIndex((value, index) => {
					return value == row
				})
				if (findResult != -1) {
					_this.$refs.tableData.toggleRowSelection(row, false);
				} else {
					_this.$refs.tableData.toggleRowSelection(row, true);
				}
			},

			//选中数据
			selsChange(selsData) {
				this.selsData = selsData
			},

			//翻页
			handleSizeChange(val) {
				let _this = this
				_this.pageSize = val
				_this.getData()
			},

			//跳转
			handleCurrentChange(val) {
				let _this = this
				_this.pageIndex = val
				_this.getData()
			},

			// 导出
			exportExcel() {
				var wb = XLSX.utils.table_to_book(document.querySelector('#tableData'))
				var wbout = XLSX.write(wb, {
					bookType: 'xlsx',
					bookSST: true,
					type: 'array'
				})
				try {
					FileSaver.saveAs(new Blob([wbout], {
						type: 'application/octet-stream'
					}), '商品信息.xlsx')
				} catch (e) {
					if (typeof console !== 'undefined') {
						console.log(e, wbout)
					}
				}
				return wbout
			}

		}
	}
</script>
