<template>
  <div>
    <div class="page-title">发布基础数据</div>
    <div class="page-container main-container">
      <div class="mar-20 info-block">
        <p class="title">报表配置</p>
        <div class="tab-container">
          <el-form label-width="96px" label-position="left" ref="formBase" :model="formBase" :rules="rules2">
            <div>
              <el-form-item label="数据类型：" prop="categoryId" class="is-required">
                <el-select v-model="formBase.categoryId" size="mini" @change="changeCategory" class="width-300" filterable>
                  <el-option v-for="item in bigCatesArr" :label="item.tableNote" :value="item.type" :key="item.id"></el-option>
                </el-select>
                <el-select v-model="formBase.propertyTableId" size="mini" @change="changePropertyTable" class="width-300" filterable>
                  <el-option v-for="item in propertyTableArr" :label="item.propertyTableName" :value="item.tableName" :key="item.id"></el-option>
                </el-select>
              </el-form-item>
            </div>
            <div class="">
              <el-form-item label="字段：" prop="checkedFileds" class="is-required">
                <div class="field-color">
                  <div v-if="formBase.propertyTableId">
                    <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选</el-checkbox>
                    <el-checkbox-group v-model="formBase.checkedFileds" @change="handleCheckedChange">
                      <div v-for="(field, index) in fieldsArr" :key="field.id" class="mr-30 line-block">
                        <el-checkbox :label="field">{{field.wordExplain}}</el-checkbox>
                      </div>
                    </el-checkbox-group>
                  </div>
                </div>
              </el-form-item>
            </div>
            <div>
              <el-form-item label="筛选字段：">
                <div class="search-table2">
                  <div class="table-cell">
                    <p v-for="(item, index) in searchForm" :key="index" class="serach-con-form">
                      <span  class="input-con">
                        <el-select v-model="item.field" size="mini" @change="changeField(item.field,index)">
                          <el-option v-for="child in formBase.checkedFileds" :key="child.id" :value="child.word" :label="child.wordExplain"></el-option>
                        </el-select>
                      </span>
                      <span  class="input-con width-150">
                        <el-select v-model="item.conditions" size="mini">
                          <el-option v-for="child in conditionArray" :key="child.id" :value="child.value" :label="child.label" :disabled="(typeObj[item.field] == 'DATE' && child.label != '=')"></el-option>
                        </el-select>
                      </span>
                      <span class="input-con width-250">
                      <el-input size="mini" v-model="item.value" maxlength="200" v-if="typeObj[item.field] != 'DATE' && typeObj[item.field] != 'SELECT'"></el-input>
                      <el-select v-model="item.value" size="mini"  v-if="typeObj[item.field] == 'SELECT'">
                        <el-option v-for="option in typeObj[item.field + '-option']" :key="option.id" :value="option.enumValue" :label="option.enumName"></el-option>
                      </el-select>
                      <el-date-picker
                        v-model="item.rangeDate"
                        type="daterange"
                        start-placeholder="开始日期"
                        end-placeholder="结束日期"
                        value-format="yyyy-MM-dd HH:mm:ss" size="mini"
                        v-if="typeObj[item.field] == 'DATE'">
                      </el-date-picker>
                    </span>
                      <span class="btn-con">
                        <el-button  size="mini" @click="addData(index)" class="plusBtn"><i class="plusBg"></i></el-button>
                        <el-button v-if="searchForm.length > 1" size="mini" @click="deleteData(index)" class="minusBtn"><i class="minusBg"></i></el-button>
                      </span>
                    </p>
                    <div class="table-cell2">
                      <el-button size="mini" @click="reset">重置</el-button>
                    </div>
                  </div>
                </div>
              </el-form-item>
            </div>
          </el-form>
        </div>
      </div>
      <div class="rightPage mb-20">
        <el-button type="primary" size="small" @click="createTable()">生成报表</el-button>
      </div>
      <div class="mar-20 info-block" v-if="firstCreate">
        <div class="tab-container">
          <el-table :data="result" border>
            <el-table-column v-for=" (item, index) in selectedKeys" :label="item.wordExplain" :key="index" show-overflow-tooltip>
              <template slot-scope="scope">
                <span>{{scope.row[item.word]}}</span>
              </template>
            </el-table-column>
          </el-table>
          <div class="footerPage">
            <span></span>
            <div class="rightPage">
              <el-pagination background @current-change="handleCurrentChange" :current-page.sync="page.page" :page-size="page.size" layout="total, prev, pager, next, jumper" :total="total">
              </el-pagination>
            </div>
          </div>
        </div>
      </div>
      <div class="mar-20 info-block" v-if="firstCreate">
        <p class="title">基本信息</p>
        <div class="tab-container">
          <el-form ref="form" :model="form" :rules="rules" label-width="96px" label-position="left">
            <el-form-item label="数据名称：" prop="name">
              <el-input size="mini" class="form-input" v-model="form.name" maxlength="50" placeholder="请输入数据名称"></el-input>
              <el-button size="mini" type="primary" class="ml-15" @click="uploadImg">
                <img :src="uploadUrl"/>
              </el-button>
            </el-form-item>
            <el-form-item label="类型：" prop="type" class="is-required">
              <div class="height-40-copy">
                <el-checkbox-group v-model="typeList" @change="changeTypeList">
                  <el-checkbox v-for="(item, index) in typeArray" :label="item.codeExt" :checked="item.checked" :key="'key0' + index">{{item.name}}</el-checkbox>
                </el-checkbox-group>
              </div>
            </el-form-item>
            <el-form-item label="对象类型：" prop="basicType" class="is-required">
              <el-button type="primary" size="mini" @click="chooseBasic">选择对象</el-button>
              <el-input v-model="form.basicType" class="hide"></el-input>
              <span class="basic-span" v-for="(item, index) in chooseTypeArray" :key="index">{{item.tableNote}} <i class="el-icon-close" @click="closeBasic(item, index)"></i></span>
            </el-form-item>
            <el-form-item label="数据简介：" prop="shortDescription">
              <el-input type="textarea" size="mini" class="radius" v-model="form.shortDescription" maxlength="500" placeholder="请输入数据简介"></el-input>
            </el-form-item>
            <el-form-item label="是否收费：" prop="isFree" class="line-block">
              <el-radio-group v-model="form.isFree">
                <el-radio label="1">是</el-radio>
                <el-radio label="0">否</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="收费标准:" prop="payStandard" v-if="form.isFree == '1'">
              <el-input size="mini" v-model="form.payStandard" class="form-input width-300" maxlength="9">
                <template slot="append">水利币/次</template>
              </el-input>
            </el-form-item>
            <!-- <span v-if="form.isFree == '1'">
              <el-input-number v-model="form.payStandard" :precision="2" :min="0" :max="999999.99" size="mini" class="mgn" ></el-input-number><span>水利币/份</span>
            </span> -->
            </el-form-item>
          </el-form>
          <template v-if="delImgVisible">
            <div class="img-preview" v-for="(n, index) in imgList" :key="index" :data-index="index">
              <img @click="preview($event)" :src="n.url"/>
              <i class="el-icon-error img-close" @click="delImg" v-if="delImgVisible"></i>
            </div>
          </template>
        </div>
      </div>
      <div class="rightPage" v-if="firstCreate">
        <el-button type="primary" size="mini" @click="save">保存</el-button>
        <el-button type="primary" size="mini" @click="submit">提交</el-button>
        <el-button size="mini" @click="back">返回</el-button>
      </div>
    </div>
    <el-dialog title="上传图片" :visible="showUploadImg" :append-to-body="true" @close="cancel" width="80%">
      <div>
        <p class="dialog-tab">
          <span :class="defaultImg ? 'active' : ''" @click="changeImgTab(1)">默认图片</span>
          <span :class="!defaultImg ? 'active' : ''" @click="changeImgTab(2)">自定义图片</span>
        </p>
        <div v-show="defaultImg" class="default-con">
          <span v-for="(item, index) in urlObjArray" :key="index" @click="chooseImg(index)" :class="{'active': item.actived}">
            <img :src="urlObj[item.url]">{{item.name}}
          </span>
        </div>
        <span class="img-outer" v-show="!defaultImg && !fileImg" @click="$refs.imgFile.click()">
          <img :src="fileUrl" style="width: 40px;"/><br/>
          <small>点击上传图片</small>
        </span>
        <span class="img-outer2" v-show="!defaultImg && fileImg" @click="$refs.imgFile.click()">
          <img :src="chooseFileUrl"/>
        </span>
        <input id="upload" ref="imgFile" class="hide" type="file" @change="doUploadImg" />
      </div>
      <span class="button-con-right">
        <el-button type="primary" @click="addImg" size="small">确定</el-button>
        <el-button  @click="cancel" size="small">取消</el-button>
      </span>
    </el-dialog>
    <el-dialog title="预览" :visible="previewVisible" :append-to-body="true" fullscreen lock-scroll @close="cancel">
      <div class="vertival-view">
        <img :src="form.picPath" class="img-size"/>
      </div>
    </el-dialog>
    <el-dialog title="对象类型" :visible="showBasic" :append-to-body="true" @close="cancel" width="600px">
      <div class="basic-list">
        <p class="filter-con">
          <label>筛选条件：</label>
          <span>
            <el-input v-model="filterVal" size="small"></el-input>
          </span>
        </p>
        <el-checkbox-group v-model="basicTypeList">
          <el-checkbox :label="item.type" class="checkbox-mar" :checked="item.checked" v-for="(item, index) in basicTypeFilterArray" :key="'key0' + index">{{item.tableNote}}</el-checkbox>
        </el-checkbox-group>
      </div>
      <div class="rightPage">
        <el-button size="mini" type="primary" @click="chooseBasicType">确定</el-button>
        <el-button size="mini" @click="cancel">取消</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { uploadFile, getBasicTypeArray, getApiTypeArray } from '@/api/api/index'
import { getBigCates, getSonCates, getFields, getTable, saveBase, submitBase } from '@/api/data/index'
import { getType } from '@/api/common/common.js'
export default {
  name: 'addBase',
  data() {
    return {
      fieldParams: [],
      columns: [],
      searchForm: [{}],
      conditionArray: [
        { value: '=', label: '=' },
        { value: '!=', label: '!=' },
        { value: '&gt;', label: '>' },
        { value: '&gt;=', label: '>=' },
        { value: '&lt;', label: '<' },
        { value: '&lt;=', label: '<=' },
        { value: 'like', label: '包含' }
      ],
      typeObj: {}, //字段-类型对应关系
      myField: '',
      showBasic: false, // 显示、隐藏基础对象选择框
      basicTypeArray: [],
      chooseTypeArray: [],
      basicTypeList: [],
      previewVisible: false,
      content: '',
      contentKey: '',
      bigCatesArr: [],
      propertyTableArr: [],
      fieldsArr: [],
      isIndeterminate: false,
      checkAll: false,
      formBase: {
        categoryId: '',
        propertyTableId: '',
        checkedFileds: []
      },
      showUploadImg: false,
      defaultImg: false,
      delImgVisible: false,
      fileImg: false,
      typeList: [],
      form: {
        name: '',
        picId: '',
        picPath: '',
        type: '',
        basicType: '',
        shortDescription: '',
        isFree: '',
        payStandard: 0
      },
      rules: {
        name: [
          { required: true, message: '请输入数据名称', trigger: 'blur' }
        ],
        type: [
          { validator: this.checkMupltiBox, key: 'typeList', message: '请选择类型', trigger: 'change' }
        ],
        basicType: [
          { validator: this.checkBasicType, trigger: 'change' }
        ],
        shortDescription: [
          { required: true, message: '请输入数据简介', trigger: 'blur' }
        ],
        isFree: [
          { required: true, message: '请选择是否收费', trigger: 'change' }
        ],
        payStandard: [
          { required: true, message: '请输入收费标准', trigger: 'blur' },
          { validator: this.checkPay, trigger: 'change' }
        ]
      },
      rules2: {
        checkedFileds: [
          { validator: this.checkField, key: 'checkedFileds', message: '请选择字段', trigger: 'change' }
        ]
      },
      switchFlag: false,
      imgList: [{
        url: '',
        width: 1000,
        height: 600
      }],
      uploadUrl: require('@/assets/images/file.png'),
      defaultUrl: require('@/assets/images/default.png'),
      fileUrl: require('@/assets/images/file2.png'),
      chooseFileUrl: '',
      typeArray: [],
      page: {
        page: 1,
        size: 10
      },
      total: 0,
      defineDialogVisible: false,
      result: [],
      selectedKeys: [],
      filterVal: '', //过滤条件
      firstCreate: false
    }

  },
  computed: {
    basicTypeFilterArray() {
      if (this.filterVal) {
        return this.basicTypeArray.filter(v => {
          return v.tableNote.indexOf(this.filterVal) > -1
        })
      } else {
        return this.basicTypeArray
      }
    },
    urlObj() {
      return JSON.parse(JSON.stringify(this.$store.state.app.urlObj))
    },
    urlObjArray() {
      return JSON.parse(JSON.stringify(this.$store.state.app.urlObjArray))
    }
  },
  created() {
    getBasicTypeArray().then(resp => {
      this.basicTypeArray = resp.data
    })
    getBigCates().then(res => {
      this.bigCatesArr = res.data
      this.formBase.categoryId = this.bigCatesArr[0].type
      getSonCates(this.formBase.categoryId).then(res => {
        this.propertyTableArr = res.data
        this.formBase.propertyTableId = this.propertyTableArr[0].tableName
        getFields(this.formBase.propertyTableId).then(res => {
          this.fieldsArr = res.data
          //获取字段对应类型
          this.fieldsArr.forEach(v => {
            this.typeObj[v.word] = (v.wordType == 'DATE' || v.wordType.indexOf('TIMESTAMP') > -1) ? 'DATE' : (v.controlType == 'SELECT' ? 'SELECT' : v.wordType)
            if (v.controlType == 'SELECT') {
              this.typeObj[v.word + '-option'] = v.enumList //下拉框时的下拉数据
            }
          })
        })
      })
    })
    getApiTypeArray({ type: '2', isChild: '0' }).then(resp => {
      this.typeArray = resp.data
    })
  },
  methods: {
    chooseImg(index) { //选中默认图片
      this.urlObjArray.forEach(v => {
        v.actived = false
      })
      this.urlObjArray[index].actived = true
      this.$forceUpdate()
      this.chooseIndex = index
    },
    chooseBasic() { //显示对象选择框
      this.showBasic = true
    },
    chooseBasicType(data) { // 选择对象
      this.form.basicType = this.basicTypeList.join()
      this.showBasic = false
      this.chooseTypeArray = []
      this.basicTypeArray.forEach(v => {
        if (this.basicTypeList.indexOf(v.type) > -1) {
          this.chooseTypeArray.push(v)
        }
      })
    },
    closeBasic(data, index) {
      this.chooseTypeArray.splice(index, 1)
      this.basicTypeList.splice(this.basicTypeList.indexOf(data.type), 1)
      this.form.basicType = this.basicTypeList.join()
    },
    checkBasicType(rule, value, callback) {
      if (this.basicTypeList.length > 0) {
        callback()
      } else {
        callback(new Error('请选择对象类型'))
      }
    },
    checkField(rule, value, callback) {
      if (this.formBase[rule.key].length == 0 && !this.switchFlag) {
        callback(new Error(rule.message))
      } else {
        this.switchFlag = false
        callback()
      }
    },
    checkPay(rule, value, callback) {
      value += ''
      //收费标准
      if (/[^\d^\.]+/g.test(value)) {

        this.form.payStandard = value.replace(/[^\d^\.]+/g, '')
        callback()

      } else if (value.indexOf('.') > -1 && value.charAt(0) !== '.') {
        //有小数点
        this.form.payStandard = value.slice(0, value.indexOf('.') + 3)
        callback()
      } else if (value.indexOf('.') < 0) {
        //没有小数点
        this.form.payStandard = value.slice(0, 6)
        callback()
      } else if (value.charAt(0) == '.') {
        //没有整数，有小数点
        this.form.payStandard = '0' + value
        callback()
      } else {
        callback()
      }
    },
    checkMupltiBox(rule, value, callback) {
      if (this[rule.key].length == 0) {
        callback(new Error(rule.message))
      } else {
        callback()
      }
    },
    changeCategory() {
      this.switchFlag = true
      getSonCates(this.formBase.categoryId).then(res => {
        this.propertyTableArr = res.data

        if (this.propertyTableArr.length > 0) {

          this.formBase.propertyTableId = this.propertyTableArr[0].tableName
          this.changePropertyTable()

        } else {
          this.formBase.propertyTableId = ''
        }
      })
    },
    changePropertyTable() {
      let _this = this
      this.checkAll = false
      this.isIndeterminate = false
      setTimeout(() => {
        _this.switchFlag = true
        getFields(_this.formBase.propertyTableId).then(res => {
          let id = _this.formBase.categoryId
          let pid = _this.formBase.propertyTableId
          _this.formBase = {
            categoryId: id,
            propertyTableId: pid,
            checkedFileds: []
          }

          
          _this.fieldsArr = res.data
          _this.searchForm = [{}]

          //获取字段对应类型
          _this.fieldsArr.forEach(v => {
            _this.typeObj[v.word] = (v.wordType == 'DATE' || v.wordType.indexOf('TIMESTAMP') > -1) ? 'DATE' : (v.controlType == 'SELECT' ? 'SELECT' : v.wordType)
            if (v.controlType == 'SELECT') {
              _this.typeObj[v.word + '-option'] = v.enumList //下拉框时的下拉数据
            }
            _this.typeObj[v.word + '-desc'] = v.wordExplain

          })
        })
      }, 100)

    },
    handleCheckAllChange(val) {
      this.formBase.checkedFileds = val ? this.fieldsArr : [];
      this.isIndeterminate = false;
    },
    handleCheckedChange(value) {
      
      let checkedCount = value.length;
      this.checkAll = checkedCount === this.fieldsArr.length;
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.fieldsArr.length;
    },
    handleCurrentChange(page) { //分页
      this.page.page = page
      this.createTable(page)
    },
    changeTypeList() {
      this.form.type = this.typeList.join()
    },
    uploadImg() {
      this.showUploadImg = true
    },
    dataURLtoFile(dataurl, filename) { //将base64转换为文件
      var arr = dataurl.split(','),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      //return new File([u8arr], filename, {type:mime}) // ie不兼容
      return new Blob([u8arr], {type:mime})
    },
    addImg() {
      if (this.defaultImg) {
        this.file = this.dataURLtoFile(this.urlObj[this.urlObjArray[this.chooseIndex].url], this.urlObjArray[this.chooseIndex].name + '.png')
        this.file.name = this.urlObjArray[this.chooseIndex].name + '.png'
      } else {
        if (this.file == '' || this.file == null || this.file == undefined) {
          this.$message({
            type: 'warning',
            message: '图片不能为空！'
          })
          return false
        }
        if (this.file.size > 3 * 1024 * 1024) {
          this.$message({
            type: 'warning',
            message: '图片大小不能超过3M！'
          })
          return false
        }
        let type = this.file.name.substring(this.file.name.lastIndexOf('.') + 1).toLowerCase()
        let typeArray = ['jpg', 'jpeg', 'tiff', 'raw', 'bmp', 'gif', 'png']
        if (typeArray.indexOf(type) < 0) {
          this.$message({
            type: 'warning',
            message: '图片格式不正确！'
          })
          return false
        }
      }
      let formData = new FormData()
      formData.append('file', this.file, this.file.name)
      uploadFile(formData).then(resp => {
        if (resp.code == 200) {
          if (!this.defaultImg) {
            this.$message({
              type: 'success',
              message: '上传成功'
            })
          }
          this.imgList[0].url = resp.data.attachmentPath
          this.form.picId = resp.data.attachmentId
          this.form.picPath = resp.data.attachmentPath
          this.showUploadImg = false
          this.delImgVisible = true
        } else {
          this.$message({
            type: 'success',
            message: resp.message
          })
        }
      })
    },
    delImg() {
      this.form.picAttachmentId = ''
      this.form.picPath = ''
      this.imgList[0].url = ''
      this.chooseFileUrl = ''
      this.fileImg = false
      this.delImgVisible = false
    },
    changeImgTab(flag) {
      if (flag == 1) {
        this.defaultImg = true
      } else {
        this.defaultImg = false
      }
    },
    doUploadImg(e) {
      if (!e || !window.FileReader) {
        return false
      }
      this.fileImg = true
      let file = e.target.files[0]
      if (file) {
        this.file = file
        let reader = new FileReader()
        let _this = this
        reader.readAsDataURL(file)
        this.chooseFileUrl = reader.result
        reader.onloadend = function() {
          _this.chooseFileUrl = this.result
        }
      }

    },
    preview(e) {
      this.previewVisible = true
    },
    cancel() {
      this.showDemo = false
      this.showUploadImg = false
      this.defineDialogVisible = false
      this.previewVisible = false
      this.showBasic = false
    },
    reset() { // 重置
      this.searchForm = [{}]
    },
    addData(index) { //新增一行筛选条件
      let param = {

      }
      this.searchForm.splice(index + 1, 0, param)
    },
    deleteData(index) { //删除一行筛选条件
      this.searchForm.splice(index, 1)
    },
    changeField(val, index) {
      //判断字段为时间，筛选条件只能选'='
      if (this.typeObj[val] == 'DATE') {
        this.$set(this.searchForm[index], 'conditions', '=')
        this.$set(this.searchForm[index], 'value', '')
        this.$set(this.searchForm[index], 'rangeDate', [])
      } else if(this.typeObj[val] == 'SELECT') {
        this.$set(this.searchForm[index], 'value', '')
      }
    },
    checkParam(data) { //校验查询参数格式
      let idArr = []
      let result = ''
      data.forEach( v => {
        if(this.typeObj[v.field] == 'NUMBER' && isNaN(v.value)  ) {
          result = 'number' + '字段[' + this.typeObj[v.field + '-desc'] +']的值必须为数值类型'
        }
        if (idArr.indexOf(v.field) > -1) {
          result = 'same'
        } else {
          idArr.push(v.field)
        }
        if (!v.field || !v.conditions || !v.value) {
          result = 'empty'
        }
      })
      return result
    },
    createTable(tag) {

      if (this.formBase.checkedFileds.length > 0) { //判断是否选择字段
        
        let json = {}


        let flag = Object.keys(this.searchForm[0]).length > 0 //判断是否有搜索条件

       
       
        if(flag) {
          this.searchForm.forEach( ve => {
            ve.value = (this.typeObj[ve.field] == 'DATE') ? ve.rangeDate.join() : ve.value
          })

          if (this.checkParam(this.searchForm) == 'same') {
            this.$message({
              type: 'warning',
              message: '筛选字段不能重复'
            })
            return false
          }
          if (this.checkParam(this.searchForm) == 'empty') {
            this.$message({
              type: 'warning',
              message: '筛选字段不能为空'
            })
            return false
          }

          if (this.checkParam(this.searchForm).indexOf('number') > -1) {
            this.$message({
              type: 'warning',
              message: this.checkParam(this.searchForm).slice(6)
            })
            return false
          }
        }
          
          
          //置空保存提交参数
          this.fieldParams = []

          this.firstCreate = true
         

          let arr = [].concat(this.formBase.checkedFileds)
          arr.forEach( (v, i) => {
            this.searchForm.forEach( ele => {
              if(ele.field == v.word) {
                arr[i] = ele
              }
            })
          })


          arr.map( ele => {
            if(ele.field) {
              json[ele.field] = [{ conditions: ele.conditions, value: ele.value }]

              //赋值保存提交参数
              this.fieldParams.push({
                conditions: ele.conditions,
                field: ele.field,
                value: ele.value,
                fieldName: this.typeObj[ele.field + '-desc']
              })

            } else {
              json[ele.word] = [{ conditions: '', value: '' }]
              //赋值保存提交参数
                this.fieldParams.push({
                  conditions: '',
                  field: ele.word,
                  value: '',
                  fieldName: ele.wordExplain
                })
            }
          })


       json.tableName = this.formBase.propertyTableId

      
        this.page.page = tag ? tag : 1
        let param = Object.assign({}, json, this.page)
        getTable(param).then(res => {
          if(res.code == 200) {
            this.firstCreate = true //首次生成报表
            this.selectedKeys = this.formBase.checkedFileds //报表表头
            this.result = res.data.result
            this.total = res.data.total
          }
        })
      } else {
        this.$message({
          type: 'warning',
          message: '请选择字段',
          duration: 1300
        })
      }
    },
    save() {
      this.$refs['formBase'].validate((valid) => {
        if (valid) {
          let json = {}
          json.mainType = this.formBase.categoryId
          json.subTableName = this.formBase.propertyTableId

          json.fieldsList = this.fieldParams

          this.$refs['form'].validate((valid) => {
            if (valid) {
              this.form.payStandard = this.form.isFree == '1' ? this.form.payStandard : 0
              let param = Object.assign({}, json, this.form)
              //保存
              saveBase(param).then(res => {
                if (res.code == 200) {
                  this.$message({
                    type: 'success',
                    message: '保存成功',
                    duration: 1300
                  })
                  this.$router.push({
                    name: 'dataCenter',
                    params: {
                      typeNum: 2,
                      type: 'init'
                    }
                  })
                  this.$store.commit('closeTag', 'addBase')
                } else {
                  this.$message({
                    type: 'error',
                    message: res.message
                  })
                }
              })
            } else {
              this.$message({
                type: 'warning',
                message: '保存失败，请完善发布信息',
                duration: 1300
              })
              return false
            }
          })
        } else {
          return false
        }
      })
    },
    submit() {
      this.$refs['formBase'].validate((valid) => {
        if (valid) {
          let json = {}
          json.mainType = this.formBase.categoryId
          json.subTableName = this.formBase.propertyTableId

          json.fieldsList = this.fieldParams


          this.$refs['form'].validate((valid) => {
            if (valid) {
              this.form.payStandard = this.form.isFree == '1' ? this.form.payStandard : 0
              let param = Object.assign({}, json, this.form)
              //提交
              submitBase(param).then(res => {
                if (res.code == 200) {
                  this.$message({
                    type: 'success',
                    message: '提交成功',
                    duration: 1300
                  })
                  this.$router.push({
                    name: 'dataCenter',
                    params: {
                      typeNum: 2,
                      type: 'init'
                    }
                  })
                  this.$store.commit('closeTag', 'addBase')
                } else {
                  this.$message({
                    type: 'error',
                    message: res.message
                  })
                }
              })
            } else {
              this.$message({
                type: 'warning',
                message: '提交失败，请完善发布信息',
                duration: 1300
              })
              return false
            }
          })
        } else {
          return false
        }
      })
    },
    back() {
      this.$router.push({
        name: 'dataCenter',
        params: {
          typeNum: 2,
          type: 'init'
        }
      })
      this.$store.commit('closeTag', 'addBase')
    }
  }
}

</script>
<style lang="less">
@import '../../assets/scss/self.less';

</style>
